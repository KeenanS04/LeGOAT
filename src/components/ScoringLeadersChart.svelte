<script>
  import { onMount, onDestroy } from 'svelte';
  import * as d3 from 'd3';

  export let chartData = [];
  export let index;
  index = 0;

  let svg;
  let width, height;
  let margin;

  let x = d3.scaleLinear();
  let y = d3.scaleBand().padding(0.2);

  onMount(() => {
    margin = { 
      top: window.innerHeight/20, 
      right: window.innerWidth/50, 
      bottom: window.innerHeight/20, 
      left: window.innerWidth/10 };
    width = window.innerWidth / 2 - margin.left - margin.right;
    height = window.innerHeight / 2 - margin.top - margin.bottom;

    x.range([0, width]);
    y.range([0, height]); 
    if (typeof window !== 'undefined') {
      updateDimensions();
      window.addEventListener('resize', updateDimensions);
      drawChart();
    }
  });

  onDestroy(() => {
    if (typeof window !== 'undefined') {
      window.removeEventListener('resize', updateDimensions);
    }
  });

  function updateDimensions() {
    if (typeof window === 'undefined') return;
    if (svg) {
      svg.attr("width", width + margin.left + margin.right)
         .attr("height", height + margin.top + margin.bottom);
      updateChart();
    }
  }

  $: if (chartData.length > 0 && index != 0 && typeof window !== 'undefined') {
    updateChart();
  }

  function drawChart() {
    svg = d3.select("#bar-chart")
      .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
      .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

    svg.append("text")
      .attr("x", width / 2)
      .attr("y", -10) 
      .attr("text-anchor", "middle")
      .style("font-size", "18px")
      .style("font-family", "Bebas Neue")
      .style("text-spacing", "2px")
      .text("All Time Scoring Chart");

    svg.append("g")
      .attr("transform", `translate(0,${height})`)
      .attr("class", "x-axis");

    svg.append("g")
      .attr("class", "y-axis");
  }

  function updateChart() {
    if (!svg) return;
    chartData.sort((a, b) => b.score - a.score);

    x.domain([0, d3.max(chartData, d => d.score)]);
    y.domain(chartData.map(d => d.player));

    svg.select(".x-axis").transition().duration(750).call(d3.axisBottom(x));
    svg.select(".y-axis").transition().duration(750).call(d3.axisLeft(y));

    const bars = svg.selectAll(".bar")
      .data(chartData, d => d.player);

    bars.enter()
      .append("rect")
      .attr("class", "bar")
      .merge(bars)
      .transition()
      .duration(750)
      .attr("x", 0)
      .attr("y", d => y(d.player))
      .attr("width", d => x(d.score))
      .attr("height", y.bandwidth())
      .attr("fill", d => d.player.includes('LeBron James') ? '#d4af37' : '#333')
      .attr("rx", 5)
      .attr("ry", 5);

    bars.exit().transition().duration(750).attr("width", 0).remove();
  }
</script>

<div id="bar-chart"></div>

<style>
  #bar-chart {
    width: 50vw;
    height: 50vh;
    position: absolute;
    top: 30px;
    left: 0;
  }
</style>