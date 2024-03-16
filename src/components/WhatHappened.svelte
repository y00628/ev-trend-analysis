<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    let chartContainer;
  
    onMount(async () => {
        // Parse the CSV data
        const res1 = await fetch('texas_vs_average.csv'); 
        const csv = await res1.text();
        const data = d3.csvParse(csv, d3.autoType);
        data.forEach(d => {
        d.Year = new Date(d.Year, 0); // Create a date object with year and January as the month.
        });
        console.log(data)
    
        const margin = { top: 60, right: 30, bottom: 30, left: 50 },
                width = 600 - margin.left - margin.right,
                height = 400 - 20 - margin.bottom;
    
        const svg = d3.select(chartContainer)
            .append('svg')
            .attr('width', width + margin.left + margin.right)
            .attr('height', height + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);
    
        // Create scales
        const xScale = d3.scaleTime()
            .domain(d3.extent(data, d => d.Year))
            .range([0, width]);
    
        const yScale = d3.scaleLinear()
            .domain([0, d3.max(data, d => Math.max(d.TX, d['Total Average']))])
            .range([height, 0]);
    
        // Define the two lines
        const lineTX = d3.line()
            .x(d => xScale(d.Year))
            .y(d => yScale(d.TX));
    
        const lineAvg = d3.line()
            .x(d => xScale(d.Year))
            .y(d => yScale(d['Total Average']));
    
        // Draw the TX line
        svg.append("path")
            .data([data])
            .attr("class", "aline")
            .style("stroke", "orange")
            .style('stroke-width', '4')
            .style("fill", "none") // Explicitly set fill to none
            .attr("d", lineTX);
    
        // Draw the average line
        svg.append("path")
            .data([data])
            .attr("class", "aline")
            .style("stroke", "blue")
            .style('stroke-width', '4')
            .style("fill", "none") // Explicitly set fill to none
            .attr("d", lineAvg);
    
        // Add the X Axis
        svg.append("g")
            .attr("class", "axis")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(xScale))
            .style("font-size", "16px");
    
        // Add the Y Axis
        svg.append("g")
            .attr("class", "axis")
            .call(d3.axisLeft(yScale))
            .style("font-size", "16px");
    
        // Legend
        const legend = svg.append('g')
        .attr('class', 'legend')
        .attr('transform', `translate(${width - 100},${margin.top})`); // Position the legend

        // Add one `g` element for each legend item
        const legendItems = [
        { color: 'orange', text: 'TX', },
        { color: 'blue', text: 'Total Average', }
        ];

        legendItems.forEach((item, index) => {
        // Legend item group
        const legendItem = legend.append('g')
            .attr('transform', `translate(0, ${index * 20})`); // Offset each item

        // Color swatch
        legendItem.append('rect')
            .attr('y', -60)
            .attr('width', 18)
            .attr('height', 18)
            .style('fill', item.color);

        // Text label
        legendItem.append('text')
            .attr('x', 24)
            .attr('y', -50)
            .attr('dy', '.35em')
            .style('text-anchor', 'start')
            .text(item.text);
        });

        // Add labels for the top 5 TX values
        let txData = [{Year: 2005, TX: 763, rank: 1},
        {Year: 2008, TX: 476, rank: 4},
        {Year: 2011, TX: 261, rank: 5},
        {Year: 2020, TX: 616, rank: 2},
        {Year: 2021, TX: 513, rank: 3}];
        txData.forEach(d => {
        d.Year = new Date(d.Year, 0); // Create a date object with year and January as the month.
        });
        console.log(txData);
        txData.forEach(d => {
        svg.append('text')
        .attr('x', xScale(d.Year)) // Use xScale(d.Year) directly for positioning
        .attr('y', yScale(d.TX) - 5) // Position slightly above the line
        .attr('text-anchor', 'middle') // Center the text horizontally
        .style('fill', 'black') // Optional: specify the text color if needed
        .text(d.TX + ': Top ' + d.rank); // Display the TX value

        // Add a title to the graph
        svg.append("text")
        .attr("class", "chart-title") // Optional: for styling via CSS
        .attr("x", width / 2) // Position at the center of the chart width
        .attr("y", 0 - (margin.top / 2)) // Position above the chart area
        .attr("text-anchor", "middle") // Ensure the text is centered at its x position
        .style("font-size", "20px") // Optional: set the font size
        .text("Number of Natural Disasters Each Year"); // Replace with your actual title
});
    
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

</style>
  
<div class="flex-container">
    <div class="text-container">
        <br><br>
        <span style="background-color:transparent;font-family:Arial, Helvetica, sans-serif;font-size:20px;"><span style="font-style:normal;font-variant:normal;font-weight:400;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;">From our trend analysis, Texas had constantly and greatly exceed the average disasters happened in the past two decades. </span>
        <br><br>
        <span style="background-color:transparent;font-family:Arial, Helvetica, sans-serif;font-size:20px;"><span style="font-style:normal;font-variant:normal;font-weight:400;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;">This brings us to our question: What exactly happened to Texas? </span>
        <br><br>
        <span style="background-color:transparent;font-family:Arial, Helvetica, sans-serif;font-size:20px;"><span style="font-style:normal;font-variant:normal;font-weight:400;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;">From the top 5 years with the most natural disasters, the leading causes are: hurricanes, fires, biologicals, and severe ice storms. </span>
    </div>
    <div class="chart-container" bind:this={chartContainer}></div>
</div>