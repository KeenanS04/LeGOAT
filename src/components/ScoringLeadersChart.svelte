<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let chartData = [];


  const margin = { top: 100, right: 30, bottom: 70, left: 140 },
          width = 800 - margin.left - margin.right,
          height = 600 - margin.top - margin.bottom;

  $: {
    // console.log(chartData);
    if (chartData.length > 0) {
      drawChart();
    }
  }

  function drawChart() {
    d3.select("#bar-chart").selectAll("svg").remove();

    const svg = d3.select("#bar-chart")
      .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
      .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

    // X axis
    const x = d3.scaleLinear()
      .domain([0, d3.max(chartData, d => d.score)])
      .range([0, width]);
    svg.append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x));

    // Y axis - Recalculate each time drawChart is called
    const y = d3.scaleBand()
      .domain(chartData.map(d => d.player).sort((a, b) => chartData.find(d => d.player === b).score - chartData.find(d => d.player === a).score))
      .range([0, height])
      .padding(0.1);
    svg.append("g")
      .call(d3.axisLeft(y));

    // Bars - Bind each rect to the player name to track their movement
    const bars = svg.selectAll("rect")
      .data(chartData, d => d.player);

    bars.enter()
      .append("rect")
        .attr("fill", d => d.player.includes('LeBron James') ? 'red' : 'steelblue')
        .attr("x", x(0))
        .attr("height", y.bandwidth())
      // Initial y position based on the player's score
      .attr("y", d => y(d.player))
      .merge(bars) // Merge enter and update selections
        .transition() // Start a transition to animate the bars
        .duration(750)
        .attr("y", d => y(d.player)) // New y position based on updated ranks
        .attr("width", d => x(d.score)); // Update width based on new score

    bars.exit().remove(); // Remove bars for players no longer in the data
  }
  // onMount(() => {
  //   drawChart();
  // });

  // $: if (chartData.length > 0) {
  //   console.log(chartData)
  //   drawChart();
  // }

  // function drawChart() {
  //   // debugger;
  //   let svg = d3.select('#bar-chart').select('svg');
  //   svg.selectAll('*').remove();

  //   let x = d3.scaleBand()
  //     .range([0, width])
  //     .domain(chartData.map(d => d['Leaders Names']))
  //     .padding(0.2);
    
  //   let y = d3.scaleLinear()
  //     .domain([0, d3.max(chartData, d => d['2023'])])
  //     .range([height, 0]);
    
  //   svg.append("g")
  //     .attr("transform", `translate(${margin.left},${margin.top})`)
  //     .call(d3.axisLeft(y));

  //   svg.append("g")
  //     .attr("transform", `translate(${margin.left},${height + margin.top})`)
  //     .call(d3.axisBottom(x))
  //     .selectAll("text")
  //       .attr("transform", "translate(-10,0)rotate(-45)")
  //       .style("text-anchor", "end");

  //   svg.append("g")
  //     .attr("transform", `translate(${margin.left},${margin.top})`)
  //     .selectAll("rect")
  //     .data(chartData)
  //     .enter()
  //     .append("rect")
  //       .attr("x", d => x(d['Leaders Names']))
  //       .attr("y", d => y(d['2023']))
  //       .attr("width", x.bandwidth())
  //       .attr("height", d => height - y(d['2023']))
  //       .attr("fill", "#69b3a2");
  // }
</script>

<div id="bar-chart"></div>

<style>
  #bar-chart {
    width: 500px;
    height: 400px;
  }
</style>