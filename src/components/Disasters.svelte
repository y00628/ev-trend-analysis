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

        // Function to start or pause the animation
        function toggleAnimation() {
            const yearSlider = d3.select("#yearSlider");
            const playPauseButton = d3.select("#playPauseButton");
            const minYear = +yearSlider.attr("min");
            const maxYear = +yearSlider.attr("max");

            if (!isAnimating) {
                playPauseButton.text("Pause");
                isAnimating = true;

                animationInterval = setInterval(() => {
                    let currentYear = +yearSlider.property("value");
                    if (currentYear < maxYear) {
                        currentYear++;
                        
                    } else {
                        /*
                        clearInterval(animationInterval);
                        playPauseButton.text("Play");
                        isAnimating = false;
                        */
                        currentYear = minYear;
                    }

                    yearSlider.property("value", currentYear);
                    yearSlider.dispatch("input"); // Trigger the input event programmatically
                }, 500); // Update interval in milliseconds

                

            } else {
                // Pause the animation
                clearInterval(animationInterval);
                playPauseButton.text("Play");
                isAnimating = false;
            }
        }

        // Attach the toggle function to the button
        d3.select("#playPauseButton").on("click", toggleAnimation);

    // Function to handle the input from the year textbox
        function handleYearInput() {
            const yearInput = document.getElementById('yearInput');
            const warningMessage = document.getElementById('warningMessage');
            const enteredYear = +yearInput.value; // Convert input to a number
            const pause = document.getElementById('playPause');

            // Check if the entered year is within the valid range
            if (enteredYear >= 1960 && enteredYear <= 2023) {
                warningMessage.style.display = 'none'; // Hide warning message

                if (pause.value == 'Yes') {
                    clearInterval(animationInterval); // Stop any ongoing animation
                    d3.select("#playPauseButton").text("Play");
                    isAnimating = false;
                } else {
                    d3.select("#playPauseButton").text("Pause");
                    isAnimating = true;
                }
                

                // Update the slider and the visualization
                d3.select("#yearSlider").property("value", enteredYear);
                d3.select("#yearSlider").dispatch("input");
            } else {
                // Display a warning message if the input is invalid
                warningMessage.style.display = 'block';
                warningMessage.innerText = 'Please enter a year between 1960 and 2023.';
            }
        }

        // Add an event listener to the year input textbox
        document.getElementById('yearInput').addEventListener('change', handleYearInput);


        // More user friendly interface
        document.getElementById('yearInput').addEventListener('keyup', function(event) {
        if (event.key === 'Enter') {
            handleYearInput(); // Directly call the handler function
        }
});


        // use this function to update color
        let selectedType = 'Total'; // i.e. the count of all the disasters as default value
        let selectedYear = 2023;
        
        populate_color(disasters, selectedType, selectedYear);

        const selectElement = document.getElementById('typeSelect');
        selectElement.addEventListener('change', (event) => {
            // get selection from menu
            selectedType = event.target.value;
            // clear out all element before updating
            g.selectAll("path").remove([d3.text, d3.fill]);
            svg.selectAll(".map-legend").remove();
            // Update the map based on the selected gender
            populate_color(disasters, selectedType, selectedYear);
        });

        const yearSlider = document.getElementById('yearSlider');
        const yearValueDisplay = document.getElementById('yearValue');

        yearSlider.addEventListener('input', (event) => {
            selectedYear = event.target.value;
            yearValueDisplay.textContent = selectedYear; // Update the display next to the slider

            // Clear the existing map visualization
            g.selectAll("path").remove([d3.text, d3.fill]);
            svg.selectAll(".map-legend").remove();

            // Update the map visualization based on the new year
            populate_color(disasters, selectedType, selectedYear);
        });
    });

</script>

<main>
    <article id="title">
        <h1 style="font-size: 24px; font-weight: bold; color: #333; margin-bottom: 5px; margin-left: 220px;">
            Nature Disasters Happened in U.S. Over Time</h1>
    </article>
    <div class="graph-container">
        <style>
            .graph-container {
                position: relative; 
                height: 100%; 
            }
        
            .controls-container {
                position: relative; 
                bottom: 33px; 
                left: 0; 
                padding: 0px; 
            }
            
            .slider-container {

            }

            .dropdown-container {
                margin-bottom: 10px; 
            }
        </style>
    </div>
    <div class="controls-container">
        <div class="slider-container">
            <label for="yearSlider">Year: </label>
            <input type="range" id="yearSlider" min="1960" max="2023" value="1960" step="1">
            <span id="yearValue">1960</span>
            <button id="playPauseButton">Play</button>
        </div>

        <div class="year-input-container">
            <label for="yearInput">Skip to: </label>
            <input type="text" id="yearInput" placeholder="1960-2023">

            <label for="playPause">Pause the slider?  </label>
            <select id="playPause">
                <option value="Yes">Yes</option>
                <option value="No">No</option>
            </select>
        </div>
        <div id="warningMessage" style="display: none; color: red;"></div>


        
        <div class="dropdown-container">
            <label for="typeSelect">Select Disaster Type: </label>
            <select id="typeSelect">
                <option value="Total">Total</option>
                <option value="Biological">Biological</option>
                <option value="Coastal Storm">Coastal Storm</option>
                <option value="Dam/Levee Break">Dam/Levee Break</option>
                <option value="Drought">Drought</option>
                <option value="Earthquake">Earthquake</option>
                <option value="Fire">Fire</option>
                <option value="Fishing Losses">Fishing Losses</option>
                <option value="Flood">Flood</option>
                <option value="Freezing">Freezing</option>
                <option value="Human Cause">Human Cause</option>
                <option value="Hurricane">Hurricane</option>
                <option value="Mud/Landslide">Mud/Landslide</option>
                <option value="Other">Other</option>
                <option value="Severe Ice Storm">Severe Ice Storm</option>
                <option value="Severe Storm">Severe Storm</option>
                <option value="Snowstorm">Snowstorm</option>
                <option value="Terrorist">Terrorist</option>
                <option value="Tornado">Tornado</option>
                <option value="Toxic Substances">Toxic Substances</option>
                <option value="Tropical Storm">Tropical Storm</option>
                <option value="Tsunami">Tsunami</option>
                <option value="Typhoon">Typhoon</option>
                <option value="Volcanic Eruption">Volcanic Eruption</option>
                <option value="Winter Storm">Winter Storm</option>
            </select>

    

        </div>

        
    </div>

    <style>
        
    </style>
</main>