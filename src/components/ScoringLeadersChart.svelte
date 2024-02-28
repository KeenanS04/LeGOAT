<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let chartData = [];

  onMount(() => {
    drawChart();
  });

  $: if (chartData.length > 0) {
    drawChart();
  }

  function drawChart() {
    const svg = d3.select('#bar-chart').select('svg');
    svg.selectAll('*').remove();

    const margin = { top: 30, right: 30, bottom: 70, left: 60 },
          width = 500 - margin.left - margin.right,
          height = 400 - margin.top - margin.bottom;

    const x = d3.scaleBand()
      .range([0, width])
      .domain(chartData.map(d => d.player))
      .padding(0.2);
    
    const y = d3.scaleLinear()
      .domain([0, d3.max(chartData, d => d.score)])
      .range([height, 0]);
    
    svg.append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`)
      .call(d3.axisLeft(y));

    svg.append("g")
      .attr("transform", `translate(${margin.left},${height + margin.top})`)
      .call(d3.axisBottom(x))
      .selectAll("text")
        .attr("transform", "translate(-10,0)rotate(-45)")
        .style("text-anchor", "end");

    svg.append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`)
      .selectAll("rect")
      .data(chartData)
      .enter()
      .append("rect")
        .attr("x", d => x(d.player))
        .attr("y", d => y(d.score))
        .attr("width", x.bandwidth())
        .attr("height", d => height - y(d.score))
        .attr("fill", "#69b3a2");
  }
</script>

<svg id="bar-chart" width="500" height="400"></svg>

<style>
  /* Add styles here */
</style>