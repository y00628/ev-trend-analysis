<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import * as topojson from "topojson";
    import Scroller from "@sveltejs/svelte-scroller";

    let activeSection = 0; // Reactive variable to track the active story section

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


    onMount(async () => {
            // read in disasters_cleaned
            const res1 = await fetch('disasters_by_decade.csv'); 
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

            function populate_color(dataset, disaster_type, decade){

                // creates color code for our map
                const globalValueExtent = d3.extent(disasters, d => d[disaster_type]);
                // Define a global color scale
                const globalColorScale = d3.scaleSequential(globalValueExtent, d3.interpolateReds);
                const defaultColor = "#cccccc";

                const filtered = dataset.filter(
                (entry) => entry.decade == decade);

                const valuemap = new Map(
                    filtered.map((d) => [d.state, +d[disaster_type]])
                );

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
                        // Check if the state was found in the dataset and has a valid data value
                        return dataValue !== undefined ? `${stateName}\n${dataValue} Times This Decade` : `${stateName}\nNo Disasters Recorded`;
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
                    .text("Decade " + decade + " " + disaster_type);
            };

            

            // use this function to update color
            let selectedType = 'Total'; // i.e. the count of all the disasters as default value
            let selectedDecade = 2020;
            
            populate_color(disasters, selectedType, selectedDecade);

            const selectElement = document.getElementById('typeSelect');
            selectElement.addEventListener('change', (event) => {
                // get selection from menu
                selectedType = event.target.value;
                // clear out all element before updating
                g.selectAll("path").remove([d3.text, d3.fill]);
                svg.selectAll(".map-legend").remove();
                // Update the map based on the selected decade
                populate_color(disasters, selectedType, selectedDecade);
            });


            // to be continued

            const yearSlider = document.getElementById('yearSlider');
            const yearValueDisplay = document.getElementById('yearValue');

            yearSlider.addEventListener('input', (event) => {
                selectedDecade = event.target.value;
                yearValueDisplay.textContent = selectedDecade; // Update the display next to the slider

                // Clear the existing map visualization
                g.selectAll("path").remove([d3.text, d3.fill]);
                svg.selectAll(".map-legend").remove();

                // Update the map visualization based on the new year
                populate_color(disasters, selectedType, selectedDecade);
            });
            

            // add collapsible for our team writeup
            var coll = document.getElementsByClassName("collapsible");
            var i;

            for (i = 0; i < coll.length; i++) {
            coll[i].addEventListener("click", function() {
                this.classList.toggle("active");
                var content = this.nextElementSibling;
                if (content.style.display === "block") {
                content.style.display = "none";
                } else {
                content.style.display = "block";
                }
            });
            }
        });

    // Function to update map based on activeSection
    function updateMap(section) {
        // Use D3 to update the choropleth map based on the section's data
    }

    // Reactive statement to watch activeSection changes
    $: {
        if (activeSection !== undefined) {
        updateMap(activeSection);
        }
    }
</script>

<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    outline: green solid 3px;
  }

  .foreground {
    width: 50%;
    margin: 0 auto;
    height: auto;
    position: relative;
    outline: red solid 3px;
  }

  .progress-bars {
    position: absolute;
    background: rgba(170, 51, 120, 0.2) /*  40% opaque */;
    visibility: visible;
  }

  section {
    height: 80vh;
    background-color: rgba(0, 0, 0, 0.2); /* 20% opaque */
    /* color: white; */
    outline: magenta solid 3px;
    text-align: center;
    max-width: 750px; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }
</style>


<Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
  bind:offset
  bind:progress
>
<!-- Story sections -->
<div id="1960" on:enter={() => (activeSection = 1)}>1960's </div>
<div id="1970" on:enter={() => (activeSection = 2)}>1970's </div>
<div id="1980" on:enter={() => (activeSection = 3)}>1980's </div>
<div id="1990" on:enter={() => (activeSection = 4)}>1990's </div>
<div id="2000" on:enter={() => (activeSection = 5)}>2000's </div>
<div id="2010" on:enter={() => (activeSection = 6)}>2010's </div>
<div id="2020" on:enter={() => (activeSection = 7)}>2020's </div>

<!-- Choropleth map container -->
<div id="map"></div>
</Scroller>
