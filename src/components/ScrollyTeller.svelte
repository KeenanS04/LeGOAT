<script>
  import { onMount } from 'svelte';
  import Scroller from "@sveltejs/svelte-scroller";
  import ScoringLeadersChart from './ScoringLeadersChart.svelte';
  import * as d3 from 'd3';

  let count, index, offset, progress;
  let width, height;

  let nbaData; 
  let selectedYear = 2023; 
  let chartData = [];

  async function loadData() {
    await d3.csv('nba_data.csv').then(data => {
      nbaData = data;
      updateChartData(selectedYear);
    });
  }

  function updateChartData(year) {
    const scores = nbaData.map(d => ({
      player: d['Leaders Names'],
      score: +d[year]
      //imageURL: d['Image URL'] // Assuming this is where player images are stored
    })).filter(d => d.score > 0); 

    scores.sort((a, b) => b.score - a.score);

    let topScorers = scores.slice(0, 10);

    if (!topScorers.some(d => d.player.includes('#23 LeBron James'))) {
      const leBron = scores.find(d => d.player.includes('#23 LeBron James'));
      if (leBron) topScorers.push(leBron);
    }

    chartData = topScorers;
  }

  onMount(() => {
    loadData();
  });

</script>

<Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
  bind:offset
  bind:progress
>
  <div 
      class="background" 
      slot="background" 
      bind:clientWidth={width} 
      bind:clientHeight={height}
    >
  </div>

  <div class="foreground" slot="foreground">
    <section>
      <h1>NBA Scoring Leaders</h1>
      <ScoringLeadersChart chartData={nbaData}/>
    </section>
    <section>This is the 2004-2005 section.</section>
    <section>This is the 2005-2006 section.</section>
    <section>This is the 2006-2007 section.</section>
    <section>This is the 2007-2008 section.</section>
    <section>This is the 2008-2009 section.</section>
    <section>This is the 2009-20010 section.</section>
    <section>This is the 2010-2011 section.</section>
    <section>This is the 2012-2013 section.</section>
    <section>This is the 2013-2014 section.</section>
    <section>This is the 2014-2015 section.</section>
    <section>This is the 2015-2016 section.</section>
    <section>This is the 2016-2017 section.</section>
    <section>This is the 2017-2018 section.</section>
    <section>This is the 2018-2019 section.</section>
    <section>This is the 2019-2020 section.</section>
    <section>This is the 2020-2021 section.</section>
    <section>This is the 2021-2022 section.</section>
    <section>This is the 2022-2023 section.</section>
    <section>This is the 2023-2024 section.</section>
  </div>
</Scroller>

<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    /* background-image: url(news.jpeg); */
  }

  .foreground {
    width: 100%;
    margin: 0 auto;
    height: auto;
    position: relative;
  }

  section {
    height: 100vh;
    background-color: #f5f5dc;
    border: 3px solid #800000;
    text-align: center;
    max-width: 100%; 
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }
</style>
