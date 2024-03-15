<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    let chartContainer;
  
    onMount(() => {
      const dataset = [
        { year: 1995, tx: 10, avg: 20 },
        { year: 1998, tx: 250, avg: 25 }, 
      ];
  
      const margin = { top: 20, right: 30, bottom: 30, left: 50 },
            width = 600 - margin.left - margin.right,
            height = 400 - margin.top - margin.bottom;
  
      const svg = d3.select(chartContainer)
        .append('svg')
          .attr('width', width + margin.left + margin.right)
          .attr('height', height + margin.top + margin.bottom)
        .append('g')
          .attr('transform', `translate(${margin.left},${margin.top})`);
  
      // Create scales
      const xScale = d3.scaleTime()
        .domain(d3.extent(dataset, d => d.year))
        .range([0, width]);
  
      const yScale = d3.scaleLinear()
        .domain([0, d3.max(dataset, d => Math.max(d.tx, d.avg))])
        .range([height, 0]);
  
      // Define the two lines
      const lineTX = d3.line()
        .x(d => xScale(d.year))
        .y(d => yScale(d.tx));
  
      const lineAvg = d3.line()
        .x(d => xScale(d.year))
        .y(d => yScale(d.avg));
  
      // Draw the TX line
      svg.append("path")
        .data([dataset])
        .attr("class", "line")
        .style("stroke", "orange")
        .attr("d", lineTX);
  
      // Draw the average line
      svg.append("path")
        .data([dataset])
        .attr("class", "line")
        .style("stroke", "blue")
        .attr("d", lineAvg);
  
      // Add the X Axis
      svg.append("g")
        .attr("transform", `translate(0,${height})`)
        .call(d3.axisBottom(xScale));
  
      // Add the Y Axis
      svg.append("g")
        .call(d3.axisLeft(yScale));
    });
  </script>
  
  <style>
    .flex-container {
      display: flex;
      align-items: flex-start;
    }
  
    .text-container {
      max-width: 50%;
      padding-right: 20px;
    }
  
    .chart-container {
      flex-grow: 1;
    }
  
    .line {
      fill: none;
      stroke-width: 2;
      shape-rendering: crispEdges;
    }
  </style>
  
  <div class="flex-container">
    <div class="text-container">
      <!-- Your text content here -->
      <p>On the weekend of October 17 to 18, 1998, a pair of hurricanes over the Eastern Pacific and a near stationary cold front led to disastrous flash flooding along the Guadalupe River and over the San Antonio metro area.</p>
      <p>I was absolutely shocked when I first saw the line chart and that made me eager to figure out what happened in Texas 1998.</p>
    </div>
    <div class="chart-container" bind:this={chartContainer}></div>
  </div>