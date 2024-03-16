<script>
    import * as d3 from "d3";
    import * as topojson from "topojson";
    import { onMount } from 'svelte';

    function rename() {
        return new Map([
            ['AL', 'Alabama'], 
            ['AK', 'Alaska'], 
            ['AS', 'American Samoa'], 
            ['AZ', 'Arizona'], 
            ['AR', 'Arkansas'], 
            ['AA', 'Armed Forces Americas'], 
            ['AE', 'Armed Forces Europe'], 
            ['AP', 'Armed Forces Pacific'], 
            ['CA', 'California'], 
            ['CO', 'Colorado'], 
            ['CT', 'Connecticut'], 
            ['DE', 'Delaware'], 
            ['DC', 'District Of Columbia'], 
            ['FL', 'Florida'], 
            ['GA', 'Georgia'], 
            ['GU', 'Guam'], 
            ['HI', 'Hawaii'], 
            ['ID', 'Idaho'], 
            ['IL', 'Illinois'], 
            ['IN', 'Indiana'], 
            ['IA', 'Iowa'], 
            ['KS', 'Kansas'], 
            ['KY', 'Kentucky'], 
            ['LA', 'Louisiana'], 
            ['ME', 'Maine'], 
            ['MH', 'Marshall Islands'], 
            ['MD', 'Maryland'], 
            ['MA', 'Massachusetts'], 
            ['MI', 'Michigan'], 
            ['MN', 'Minnesota'], 
            ['MS', 'Mississippi'], 
            ['MO', 'Missouri'], 
            ['MT', 'Montana'], 
            ['NE', 'Nebraska'], 
            ['NV', 'Nevada'], 
            ['NH', 'New Hampshire'], 
            ['NJ', 'New Jersey'], 
            ['NM', 'New Mexico'], 
            ['NY', 'New York'], 
            ['NC', 'North Carolina'], 
            ['ND', 'North Dakota'], 
            ['NP', 'Northern Mariana Islands'], 
            ['OH', 'Ohio'], 
            ['OK', 'Oklahoma'], 
            ['OR', 'Oregon'], 
            ['PA', 'Pennsylvania'], 
            ['PR', 'Puerto Rico'], 
            ['RI', 'Rhode Island'], 
            ['SC', 'South Carolina'], 
            ['SD', 'South Dakota'], 
            ['TN', 'Tennessee'], 
            ['TX', 'Texas'], 
            ['VI', 'US Virgin Islands'], 
            ['UT', 'Utah'], 
            ['VT', 'Vermont'], 
            ['VA', 'Virginia'], 
            ['WA', 'Washington'], 
            ['WV', 'West Virginia'], 
            ['WI', 'Wisconsin'], 
            ['WY', 'Wyoming']]);
    }

    // stores all data here
    //
    let disasters = [];

    //topojson
    let states = [];
    let statemesh = [];

    // toggle
    let isAnimating = false;
    let animationInterval;


    onMount(async () => {
        // read in disasters_cleaned
        const res1 = await fetch('disasters_cleaned.csv'); 
        const csv = await res1.text();
        let disasters = d3.csvParse(csv, d3.autoType)
        console.log(disasters);

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

        // Update country names in your dataset
        const renameMap = rename();
        disasters.forEach(d => {

            if (renameMap.has(d.state)) {
                d.state = renameMap.get(d.state); // Replace with the name from the map
            }
        });

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

        function populate_color(dataset, disaster_type, year){

            // creates color code for our map
            const globalValueExtent = d3.extent(dataset, d => d[disaster_type]);
            // Define a global color scale
            const globalColorScale = d3.scaleSequential(globalValueExtent, d3.interpolateReds);
            const defaultColor = "#cccccc";

            const filtered = dataset.filter(
            (entry) => entry.year == year);

            const valuemap = new Map(
                filtered.map((d) => [d.state, +d[disaster_type]])
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
                    return dataValue !== undefined ? `${stateName}\n${dataValue} Times This Year` : `${stateName}\nNo Disasters Recorded`;
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
            const colorScale = d3.scaleSequential(d3.extent(disasters.map(d => +d[disaster_type])), d3.interpolateReds);

            // Define the legend scale
            const legendScale = d3.scaleLinear()
                .domain(d3.extent(disasters.map(d => +d[disaster_type])))
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
                .text("Year " + year + " " + disaster_type);
        };


        // use this function to update color
        let selectedType = 'Severe Ice Storm'; // i.e. the count of all the disasters as default value
        let selectedYear = 2021;
        
        populate_color(disasters, selectedType, selectedYear);

    });

</script>

<main>
    <article id="title">
        <h1 style="font-size: 24px; font-weight: bold; color: #333; margin-bottom: 5px; margin-left: 220px;">
            Nature Disasters Happened in U.S. Year 2021</h1>
    </article>
    <div class="graph-container">
        <style>
            .graph-container {
                position: relative; 
                height: 100%; 
            }
        </style>
    </div>
    <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">From our graph, it is easy to tell that Texas suffered badly from severe ice storms.&nbsp;</span></p>
<p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">We continued our research to find out what exactly happened back in 2021.</span></p>
<p style="margin-left:0px;"><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;"><strong>"It looked like the end of the world"</strong>: from one of the Texans who lived through 2021's historic winter storm</span></p>
<figure class="image"><img style="aspect-ratio:640/429;" src="https://thumbnails.texastribune.org/2W21ewpVVGm-XgK4SHrHTnWNRgc=/640x429/smart/filters:format(webp):quality(75)/https://static.texastribune.org/media/files/d42b8b7499841139561ee6abd6f6d9b7/ATX%20Snow%20Storm%20MG%20TT%2014.jpg" width="640" height="429"></figure>
<style>
    .image {
        display: block; 
        margin-left: 165px;
        margin-right: auto;
    }

</style>
<p style="margin-left:0px;"><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">Post by The Texas Tribune, "As Texas faced <strong>record-low temperatures</strong> in February 2021 and snow and ice made roads <strong>impassable</strong>, the state's electric grid operator lost control of the power supply, leaving <strong>millions</strong> without access to electricity. As the blackouts extended from <strong>hours to days</strong>, top state lawmakers called for investigations into the Electric Reliability Council of Texas, and Texans demanded accountability for the disaster."</span></p>
<p style="margin-left:0px;"><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">Read more:&nbsp;</span></p>
<p style="margin-left:0px;"><a target="_blank" rel="noopener noreferrer" href="https://www.texastribune.org/series/winter-storm-power-outage/"><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">https://www.texastribune.org/series/winter-storm-power-outage/</span></a></p>
<p style="margin-left:0px;"><a target="_blank" rel="noopener noreferrer" href="https://www.texastribune.org/2022/02/17/texas-winter-storm-2021-stories/"><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">https://www.texastribune.org/2022/02/17/texas-winter-storm-2021-stories/</span></a></p>
</main>