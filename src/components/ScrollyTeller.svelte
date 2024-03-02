<script>
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import Scroller from "@sveltejs/svelte-scroller";
  import ScoringLeadersChart from './ScoringLeadersChart.svelte';
  import { tweened } from 'svelte/motion';
  import { linear } from 'svelte/easing';
  import MapComp from './map.svelte';
  import * as d3 from 'd3';

  let count, index, offset, progress;
  let width, height;

  let nbaData;
  let selectedYear = 2004;
  let chartData = [];

  let sections = [
    {
      title: 'Prodigy to Phenom: The Foundation in Cleveland (2003 - 2010)',
      content: `LeBron James, the kid from Akron, Ohio, burst onto the professional basketball scene with the promise of greatness. His impact was immediate, earning the NBA Rookie of the Year award and an All-Star Game appearance in his debut season. His rare combination of size, skill, and intelligence reinvigorated the Cleveland Cavaliers. By his fourth season, he led the Cavs to their first NBA Finals appearance in franchise history. Though they fell short, LeBron had already begun etching his name into the annals of basketball lore.

      In Cleveland, LeBron's individual accolades piled up swiftly. He was a six-time All-Star, a two-time All-Star Game MVP, and secured two regular-season MVPs. He led the league in scoring in 2008 and was perennially named to the All-NBA First Team and the NBA All-Defensive First Team, showcasing his two-way prowess. However, the championship glory that Cleveland so desperately craved remained just out of reach.`
    },
    {
      title: 'The Heat is ON: The Decision, Champions and Challenges in Miami (2010 - 2014)',
      content: `The Decision to join the Miami Heat changed the NBA's balance of power and LeBron's career trajectory. In Miami, LeBron refined his game, becoming more efficient and deadly, particularly from beyond the arc. He garnered two more MVP awards and led the Heat to four straight NBA Finals, winning two. His playoff performances during this period were nothing short of legendary, including a 45-point effort against the Boston Celtics to stave off elimination in 2012 and a 37-point closing masterpiece in Game 7 of the 2013 Finals.

      Off the court, LeBron's influence grew. He was not only a fixture at All-Star weekends but also a global ambassador for the sport, representing Team USA. He captured two Olympic medals during this period – gold in Beijing (2008) and London (2012), further cementing his status as basketball royalty.`
    },
    {
      title: 'The Prodigal Son Returns: Triumph and Turmoil back in Cleveland (2014 - 2018)',
      content: `LeBron's return to Cleveland was about redemption and delivering on a long-held promise. He led the Cavaliers to four consecutive NBA Finals, including the miraculous 2016 championship win that not only shattered Cleveland's curse but also solidified his clutch legacy. In these years, he continued to stack up All-NBA First Team selections and All-Star appearances, consistently dazzling fans with his high basketball IQ and versatility.

      LeBron's 2018 playoff run was one for the ages, often single-handedly carrying the Cavs through each round, including a buzzer-beater against the Indiana Pacers that added to his highlight reel of iconic moments.`
    },
    {
      title: 'The L.A story: Cementing a Legacy (2018 - Present)',
      content: `With the Lakers, LeBron's narrative took on a new dimension. His leadership and experience helped capture the 2020 NBA Championship, a poignant triumph in the wake of Kobe Bryant's tragic passing. He continued to break records, becoming the oldest player to average a triple-double over an entire month. LeBron also earned his 17th All-Star selection in 2021, tying with Kobe Bryant and trailing only Kareem Abdul-Jabbar and Karl Malone.
      
      In 2022, at the age of 37, LeBron achieved a historic milestone by becoming the all-time leading scorer in NBA Playoffs history, surpassing Michael Jordan. On February 7, 2023, LeBron James made history by surpassing Kareem Abdul-Jabbar to become the NBA's all-time leading scorer, using his signature fadeaway jumper to seal the record. As of February 29, 2024, he is just 40 points away from reaching the unprecedented milestone of 40,000 career points, poised to set yet another record in his illustrious career.
      
      Off the hardwood, LeBron's impact has been felt in education through his "I PROMISE School," philanthropy, and activism, particularly in his staunch advocacy for racial justice and voter rights. His media company, SpringHill Entertainment, produced the movie "Space Jam: A New Legacy," where he starred as the lead, blending his athletic and artistic pursuits.`
    },
    {
      title: 'GOAT JAMES',
      content: `Debate continues about where LeBron ranks among the greatest of all time, but his achievements and impact on the game are undeniable.`
    },
  ];

  async function loadData() {
    await d3.csv('nba_data.csv').then(data => {
      nbaData = data;
      updateChartData(selectedYear);
    });
  }

  $: if (nbaData) {
    let newYear = selectedYear + 5 * index;
    updateChartData(newYear <= 2023 ? newYear : 2023);
  }
  // $: console.log(chartData);

  function updateChartData(year) {
    const scores = nbaData.map(d => ({
      player: d['Leaders Names'],
      score: +d[year]
      //imageURL: d['Image URL'] // Assuming this is where player images are stored
    })); 

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
  <!-- <div 
      class="background" 
      slot="background" 
      bind:clientWidth={width} 
      bind:clientHeight={height}
    >
    <h1 style="text-align: center; padding: 10px"> {selectedYear+5*index <=  2023 ? selectedYear+5*index : 2023}</h1>
    <ScoringLeadersChart chartData={chartData} index={index}/>
  </div> -->

  <div class="background" slot="background" bind:clientWidth={width} bind:clientHeight={height}>
    <!-- Conditional rendering with fade transition for the chart -->
    {#if chartData.length > 0}
      <div in:fade={{ delay: 300, duration: 600 }} out:fade={{ duration: 600 }}>
        <ScoringLeadersChart chartData={chartData} index={index}/>
      </div>
    {/if}
  </div>

  <!-- <div class="foreground" slot="foreground">
    <section></section>
    <section></section>
    <section></section>
    <section></section>
    <section></section>
    <section></section>
    <section></section>
    <section></section>
  </div> -->

  <!-- <div slot="background">
    <MapComp {index} />
  </div> -->

  <div class="foreground" slot="foreground">
    {#each sections as section, i}
      <section>
        {#if i === index}
          <div in:fade={{ delay: 300, duration: 600 }} out:fade={{ duration: 600 }}>
            <h1>{section.title}</h1>
            <p id='left'>{section.content}</p>
          </div>
        {/if}
      </section>
    {/each}
  </div>
</Scroller>

<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    /* background-image: url(news.jpeg); */
    background-color: #f5f5dc;
    padding: 0;
  }

  .foreground {
    width: 100%;
    margin: 0 auto;
    height: auto;
    position: relative;
  }

  section {
    height: 100vh;
    /* border: 3px solid #800000; */
    text-align: center;
    max-width: 100%; 
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
    border: maroon 3px;
  }

  #left{
    text-align: center;
    padding-left: 60%;
    padding-top: 10%;
  }
</style>
