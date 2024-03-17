<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    let chartContainer;
  
    onMount(async () => {
        // Parse the CSV data
        const res1 = await fetch('correlation.csv'); 
        const csv = await res1.text();
        const data = d3.csvParse(csv, d3.autoType);
        data.forEach(d => {
        d.Year = new Date(d.Year, 0); // Create a date object with year and January as the month.
        });
        console.log(data)
    
        const margin = { top: 100, right: 30, bottom: 30, left: 50 },
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
            .domain([0, 750])
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
            .style("fill", "#FFC594") // Explicitly set fill to none
            .attr("d", lineTX);
    
        // Draw the average line
        svg.append("path")
            .data([data])
            .attr("class", "aline")
            .style("stroke", "blue")
            .style('stroke-width', '4')
            .style("fill", "#94A6FF") // Explicitly set fill to none
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
            .attr('x', 14)
            .attr('y', -130)
            .attr('width', 18)
            .attr('height', 18)
            .style('fill', item.color);

        // Text label
        legendItem.append('text')
            .attr('x', 38)
            .attr('y', -120)
            .attr('dy', '.35em')
            .style('text-anchor', 'start')
            .text(item.text);
        });

        
        // Add a title to the graph
        svg.append("text")
        .attr("class", "chart-title") // Optional: for styling via CSS
        .attr("x", width / 2) // Position at the center of the chart width
        .attr("y", 0 - (margin.top / 2)) // Position above the chart area
        .attr("text-anchor", "middle") // Ensure the text is centered at its x position
        .style("font-size", "20px") // Optional: set the font size
        .text("Carbon Emission in Million Metric Tons Each Year"); // Replace with your actual title
    
});
</script>
  
<style>
    .chart-container {
        position: relative; 
        margin-left: 185px;
    }

</style>
  
<div>
    <div class="text-container">
        <br>
        <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">From research, there has been no carbon emission law and very little effort to limit carbon pollution in Texas until March 2024. From Chambers and Partners, they straight up confessed that “Texas has not enacted statutory carbon emission targets, effective from 1 September 2023”.&nbsp;</span></p>
        <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">Yes, there are goals and promises. But the pollution trend is still increasing steadily from our analysis, and people on that land deserve to know the truth: how much the industry produces a million tons of carbon pollution each year and how the authorities choose to ignore the fact that the climate is deteriorating because of the pollution. &nbsp;</span></p>
        <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">In the end, we believe that no disaster is a coincidence. We hope to appeal the importance of taking care of our environment and planet to the public from this interactive graphs and project.&nbsp;</span></p>
        <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">Source:&nbsp;</span></p>
        <p><a target="_blank" rel="noopener noreferrer" href="https://practiceguides.chambers.com/practice-guides/environmental-law-2023/usa-texas/"><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">https://practiceguides.chambers.com/practice-guides/environmental-law-2023/usa-texas/</span></a></p>
    </div>
    <div class="chart-container" bind:this={chartContainer}></div>
</div>