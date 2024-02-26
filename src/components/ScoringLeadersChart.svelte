<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    export let data = [];
  
    onMount(() => {
      if (data.length) {
        drawChart();
      }
    });
  
    function drawChart() {
      // gotta fill this out
    }
  </script>
  
  <div id="chart"></div>