<script>
    import * as d3 from "d3";
    import * as topojson from "topojson";
    import { onMount } from 'svelte';

    // stores all data here
    //
    let carbon = [];
    

    //topojson
    let states = [];
    let statemesh = [];

    onMount(async () => {
        // read in carbon_cleaned
        const res1 = await fetch('carbon_cleaned.csv'); 
        const csv = await res1.text();
        let carbon = d3.csvParse(csv, d3.autoType);
        console.log(carbon);

        // read in topojson
        const res2 = await fetch('us.json');
        const data = await res2.json();
        states = topojson.feature(
            data,
            data.objects.states
        );
        statemesh = topojson.mesh(
            data,
            data.objects.states,
            (a, b) => a !== b
        ); 

        // creates SVG with specified characteristics
        const width = 1000;
        const marginTop = 46;
        const height = width / 2 + marginTop;

        const svg = d3
            .select(".graph-container")
            .append("svg")
            .attr("width", width)
            .attr("height", height);

        // creates base map
        const projection = d3.geoAlbersUsa();
        const path = d3.geoPath(projection);

        // creates map border
        svg
            .append("path")
            .datum({ type: "Sphere" })
            .attr("fill", "white")
            .attr("stroke", "currentColor")
            .attr("d", path);

        const g = svg.append("g");

        // creates a fixed scale
        const max_carbon_year = 2018;
        const globalValueExtent = d3.extent(carbon, d => d[max_carbon_year]);
        // Define a global color scale
        const globalColorScale = d3.scaleSequential(globalValueExtent, d3.interpolateReds);
        const defaultColor = "#cccccc";


        function populate_color(dataset, year){
            const valuemap = new Map(
                dataset.map((d) => [d.State, +d[year]])
            );

            console.log(valuemap);

            // assigns colors to countries based on FactValueNumeric data
            
            g.selectAll("path")
                .data(states.features)
                .join("path")
                .attr("fill", d => {
                    const stateName = d.properties.name; // Get the country name from the GeoJSON feature
                    const dataValue = valuemap.get(stateName); // Attempt to get the data value for this country

                    // Check if the country was found in the dataset and has a valid data value
                    if (dataValue !== undefined) {
                        return globalColorScale(dataValue); // Use the data value to determine the color
                    } else {
                        return defaultColor; // Use the default color for countries not in the dataset
                    }
                })
                .attr("d", path)
                // tooltip
                .append("title")
                .text(d => {
                    const stateName = d.properties.name;
                    const dataValue = valuemap.get(stateName); // Get the data value for this country
                    // Check if the country was found in the dataset and has a valid data value
                    return dataValue !== undefined ? `${stateName}\n${dataValue} Million Metric Tons` : `${stateName}\nNo Emission Recorded`;
                });

            // adds a white border between countries for *aesthetics*
            svg
                .append("path")
                .datum(statemesh)
                .attr("fill", "none")
                .attr("stroke", "white")
                .attr("d", path);

            const legendWidth = 300, legendHeight = 20, legendMargin = {top: 10, right: 60, bottom: 40, left: 60};

            // Create a sequential color scale (if different, adjust as needed)
            const colorScale = d3.scaleSequential(d3.extent(carbon.map(d => +d[max_carbon_year])), d3.interpolateReds);

            // Define the legend scale
            const legendScale = d3.scaleLinear()
                .domain(d3.extent(carbon.map(d => +d[max_carbon_year])))
                .range([0, legendWidth]);

            // Append a legend group to the SVG
            const legend = svg.append("g")
                .attr("class", "map-legend")
                .attr("transform", `translate(${width - legendWidth - legendMargin.right},${height - legendMargin.bottom})`);

            // Append gradient bar for the legend
            const linearGradient = legend.append("defs")
                .append("linearGradient")
                .attr("id", "gradient")
                .attr("x1", "0%")
                .attr("x2", "100%")
                .attr("y1", "0%")
                .attr("y2", "0%");

            colorScale.ticks().forEach(function(tick, i, ticks) {
                linearGradient.append("stop")
                    .attr("offset", `${100 * i / ticks.length}%`)
                    .attr("stop-color", colorScale(tick));
            });

            // Append color bar
            legend.append("rect")
                .attr("width", legendWidth)
                .attr("height", legendHeight)
                .style("fill", "url(#gradient)");

            // Append legend axis
            legend.append("g")
                .attr("transform", `translate(0, ${legendHeight})`)
                .call(d3.axisBottom(legendScale).ticks(6));
            
            // Append legend title
            legend.append("text")
                .attr("class", "legend-title")
                .attr("x", 300)
                .attr("y", -5)
                .attr("text-anchor", "end")
                .text("Year " + year);
        };

        // use this function to update color
        let selectedYear = 2021;
        
        populate_color(carbon, selectedYear);

    });

</script>

<main>
    <br>
    <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">Rhythm Energy - a company that has expertise in Texas energy and environment - wrote, “undoubtedly, there was a connection between the Texas winter storm of 2021 and climate change. The cold temperatures froze everything, while heavy snowfall hit this sunny state, which wasn’t usual. Scientists think that this may be linked to broader climate change.”</span></p>
    <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">A vast number of articles pointed out that those abrupt climate changes, like global warming, are directly linked to carbon emissions. Moreover, a research “CO2 Can Directly Impact Extreme Weather, Research Suggests” posted in Scientific American examined carbon emission's impact on climate comprehensively.&nbsp;</span></p>
    <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">But is this something happening in Texas? We need more data to tell us the truth.&nbsp;</span></p>
    <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">Source:&nbsp;</span></p>
    <p><a target="_blank" rel="noopener noreferrer" href="https://www.gotrhythm.com/blog/energy-saving-tips/climate-change-texas-winter-storm-growing-challenges"><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">https://www.gotrhythm.com/blog/energy-saving-tips/climate-change-texas-winter-storm-growing-challenges</span></a></p>
    <p><a target="_blank" rel="noopener noreferrer" href="https://www.scientificamerican.com/article/co2-can-directly-impact-extreme-weather-research-suggests/"><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">https://www.scientificamerican.com/article/co2-can-directly-impact-extreme-weather-research-suggests/</span></a></p>
    <p>&nbsp;</p>

    <article id="title">
        <h1 style="font-size: 24px; font-weight: bold; color: #333; margin-bottom: 5px; margin-left: 220px;">
            2021 Carbon Emission Data (in Million Metric Tons) in U.S.</h1>
    </article>
    <div class="graph-container">
        <style>
            .graph-container {
                position: relative; 
                height: 100%; 
            }
        </style>
    </div>
    
    <style>

    </style>
</main>