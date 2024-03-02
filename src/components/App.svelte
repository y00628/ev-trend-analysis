<script>
    import * as d3 from "d3";
    import * as topojson from "topojson";
    import { onMount } from 'svelte';

    // stores all data here
    //
    let disasters = [];

    //topojson
    let states = [];
    let statemesh = [];

    onMount(async () => {
        // read in disasters_cleaned
        const res1 = await fetch('disasters_cleaned.csv'); 
        const csv = await res1.text();
        let tobacco = d3.csvParse(csv, d3.autoType)
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
        const path = d3.geoPath().projection(projection);

        // creates map border
        svg
            .append("path")
            .datum({ type: "Sphere" })
            .attr("fill", "white")
            .attr("stroke", "currentColor")
            .attr("d", path);

        const g = svg.append("g");

        // creates color code for our map
        const globalValueExtent = d3.extent(disasters, d => d.FactValueNumeric);
        // Define a global color scale
        const globalColorScale = d3.scaleSequential(globalValueExtent, d3.interpolateYlGnBu);
        const defaultColor = "#cccccc";

        function populate_color(dataset, sex, year){
            console.log(dataset);
            const filtered = dataset.filter(
            (entry) => entry.Dim1 == sex && entry.Period == year && entry.Indicator == 'Estimate of current tobacco use prevalence (%) (age-standardized rate)'
            );
            console.log(filtered);

            const valuemap = new Map(
                filtered.map((d) => [d.Location, +d.FactValueNumeric])
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
                    return dataValue !== undefined ? `${stateName}\n${dataValue}%` : `${stateName}\nNo Data`;
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
            const colorScale = d3.scaleSequential(d3.extent(disasters.map(d => +d.FactValueNumeric)), d3.interpolateYlGnBu);

            // Define the legend scale
            const legendScale = d3.scaleLinear()
                .domain(d3.extent(tobacco.map(d => +d.FactValueNumeric)))
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
                .text("Estimate of Year " + year + " Tobacco Use Prevalence in " + sex + " (%) (Age-standardized Rate)");
        };

        

        // use this function to update color
        let selectedType = 'Total'; // i.e. the count of all the disasters as default value
        let selectedYear = 2023;
        
        populate_color(tobacco, selectedType, selectedYear);
        const selectElement = document.getElementById('typeSelect');
        selectElement.addEventListener('change', (event) => {
            // get selection from menu
            selectedSex = event.target.value;
            // clear out all element before updating
            g.selectAll("path").remove([d3.text, d3.fill]);
            svg.selectAll(".map-legend").remove();
            // Update the map based on the selected gender
            populate_color(tobacco, selectedSex, selectedYear); 
        });

        const yearSlider = document.getElementById('yearSlider');
        const yearValueDisplay = document.getElementById('yearValue');

        yearSlider.addEventListener('input', (event) => {
            const years = [1953, 1954, 1955, 1956, 1957, 1958, 1959, 1960, 1961, 1962, 1963,
       1964, 1965, 1966, 1967, 1968, 1969, 1970, 1971, 1972, 1973, 1974,
       1975, 1976, 1977, 1978, 1979, 1980, 1981, 1982, 1983, 1984, 1985,
       1986, 1987, 1988, 1989, 1990, 1991, 1992, 1993, 1994, 1995, 1996,
       1997, 1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007,
       2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017, 2018,
       2019, 2020, 2021, 2022, 2023]; 
            selectedYear = years[event.target.value];
            yearValueDisplay.textContent = selectedYear; // Update the display next to the slider

            // Clear the existing map visualization
            g.selectAll("path").remove([d3.text, d3.fill]);
            svg.selectAll(".map-legend").remove();

            // Update the map visualization based on the new year
            populate_color(disasters, selectedType, selectedYear);
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

</script>

<main>
    <article id="title">
        <h1 style="font-size: 24px; font-weight: bold; color: #333; margin-bottom: 5px; margin-left: 220px;">
            World-Wide Tobacco Usage Per Gender Over Time</h1>
    </article>
    <div class="graph-container">
        <style>
            .graph-container {
                position: relative; 
                height: 100%; 
            }
        
            .controls-container {
                position: relative; 
                bottom: 63px; 
                left: 0; 
                padding: 0px; 
            }
        
            .dropdown-container, .slider-container {
                margin-bottom: 10px; 
            }
        </style>
    </div>
    <div class="controls-container">
        <div class="slider-container">
            <label for="yearSlider">Select Year: </label>
            <input type="range" id="yearSlider" min="0" max="9" value="0" step="1">
            <span id="yearValue">2000</span>
        </div>

        <div class="dropdown-container">
            <label for="genderSelect">Select Gender: </label>
            <select id="genderSelect">
                <option value="Both sexes">Both sexes</option>
                <option value="Male">Male</option>
                <option value="Female">Female</option>
            </select>
        </div>
    </div>
    <button type="button" class="collapsible">Team Writeup (Click to open)</button>
        <div class="content">
        <p>
            <p>
                <strong>DSC106 Project 3 Writeup</strong>
              </p>
              <p>Group members: Nicholas Jumaoas, Jiaqing Yan, Yosen Lin</p>
              <p>After deciding to use Jiaqing’s WHO tobacco usage dataset for our visualization, we very quickly settled on using a choropleth map as the base structure for our visualization since it allows differences in geographic distributions to be conveyed directly and evocatively. Building off of that, we took inspiration from examples from the labs and the D3 website and decided to implement filters for gender and time, as well as adding a hover-based tooltip that displays country name and its percentage of tobacco usage. While a dropdown menu felt like the most intuitive interaction technique for the gender display, we chose a slider over the dropdown menu showcased on the WHO website for the time slider because it allows the viewer to more seamlessly experience how the distribution of tobacco usage has changed over time.</p>
              <p>Our development process consisted of data cleaning, the construction of the base choropleth, and then the implementation of the tooltip as well as gender and time series filters and their corresponding interactive elements. After considering scheduling limitations of group members, the work was assigned as follows:</p>
              <div>
                <table>
                  <tbody>
                    <tr>
                      <td>
                        <strong>Name&nbsp;&nbsp;</strong>
                      </td>
                      <td>
                        <strong>&nbsp; Responsibilities</strong>
                      </td>
                    </tr>
                    <tr>
                      <td>Nicholas</td>
                      <td>&nbsp; Base choropleth, tooltip, writeup</td>
                    </tr>
                    <tr>
                      <td>Jiaqing</td>
                      <td>&nbsp; Gender filter, data cleaning, gender dropdown menu, writeup collapsible</td>
                    </tr>
                    <tr>
                      <td>Yosen</td>
                      <td>&nbsp; Time series filter, year slider</td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <p>Including both development of the visualization itself as well as external work such as the data cleaning and writeup, our group spent approximately 22 hours working on this project, encountering various obstacles ranging from missing data to messy file structures. In particular, the missing data caused various iterations of the slider implementation to be plagued by bugs. We fixed it by making the slider more discrete. Additionally, there were general struggles involving straightening out the logic behind the interactive graph itself as well as the JavaScript/Svelte code behind it.</p>
        </div>
        <style>
            .collapsible {
            background-color: #eee;
            color: #444;
            cursor: pointer;
            padding: 18px;
            width: 100%;
            border: none;
            text-align: left;
            outline: none;
            font-size: 15px;
            }

            .active, .collapsible:hover {
            background-color: #ccc;
            }

            .content {
            padding: 0 18px;
            display: none;
            overflow: hidden;
            background-color: #f1f1f1;
            }
        </style>
</main>