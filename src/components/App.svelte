<script>
    import Disasters from './Disasters.svelte';
    import Carbon from './Carbon.svelte';
    import IceStormTexas from './IceStormTexas.svelte';
    import WhatHappened from './WhatHappened.svelte';
    import Cause from './Cause.svelte';
    import Correlation from './Correlation.svelte';
    let currentPage = 'Disasters';
</script>

<body>
    <div class="wrapper">
        <br><br>
        <h1><span style="font-family:'Lucida Sans Unicode', 'Lucida Grande', sans-serif;">U.S. Disasters Trend Analysis</span></h1>
        <h2><span style="font-family:'Lucida Sans Unicode', 'Lucida Grande', sans-serif;">By - Nick Jumaoas, Yosen Lin, Jiaqing Yan&nbsp;</span></h2>
        <br><br>
        <p style="line-height:1.38;margin-bottom:12pt;margin-top:0pt;" dir="ltr"><span style="background-color:transparent;font-family:Arial, Helvetica, sans-serif;font-size:20px;"><span style="font-style:normal;font-variant:normal;font-weight:400;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;">Most of the public don't realize how natural disasters are </span><span style="font-style:normal;font-variant:normal;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;"><strong>one of the leading reasons for fatalities</strong></span><span style="font-style:normal;font-variant:normal;font-weight:400;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;">.&nbsp;</span></span></p>
        <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">Imagine a nation <strong>constantly battling the forces of nature</strong> — storms, wildfires, floods, and droughts — each leaving its own mark on the land and its people.&nbsp;</span></p>
        <p><span style="font-family:Arial, Helvetica, sans-serif;font-size:20px;">Imagine you have the power to <strong>travel through time</strong>, witnessing the ebb and flow of these natural disasters across the vast landscapes of the United States. What <strong>patterns</strong> would emerge? What <strong>stories</strong> would the data tell?</span></p>
        <p><span style="font-family:Arial, Helvetica, sans-serif;">
                <meta charset="utf-8">
            </span><span style="background-color:transparent;font-family:Arial, Helvetica, sans-serif;font-size:20px;"><span style="font-style:normal;font-variant:normal;font-weight:400;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;">We hope our analysis can bring the public's attention to those&nbsp;</span><span style="font-style:normal;font-variant:normal;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;"><strong>rapid and horrifying facts</strong></span><span style="font-style:normal;font-variant:normal;font-weight:400;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;">. So those tragedies will never happen again.</span></span></p>

        <p style="line-height:1.2;margin-bottom:0pt;margin-top:0pt;" dir="ltr"><span style="background-color:transparent;color:#FFFFFF;font-family:Nunito,sans-serif;font-size:16pt;"><span style="font-style:normal;font-variant:normal;font-weight:400;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;">Nick Jumaoas, Yosen Lin, Jiaqing Yan</span></span>&nbsp;&nbsp;
            <meta charset="utf-8">
        </p>
        <p style="line-height:1.2;margin-bottom:0pt;margin-top:0pt;" dir="ltr"><span style="background-color:transparent;color:#FFFFFF;font-family:Nunito,sans-serif;font-size:16pt;"><span style="font-style:normal;font-variant:normal;font-weight:400;text-decoration:none;vertical-align:baseline;white-space:pre-wrap;">Nick Jumaoas, Yosen Lin, Jiaqing Yan</span></span></p>
        
        <container>
            <h3><span style="font-family:Arial, Helvetica, sans-serif;">2021 North American winter storm:&nbsp;&nbsp;</span>
                <meta charset="utf-8">
            </h3>

            <button on:click={() => currentPage = 'IceStormTexas'} 
                style="background-color: #FF9C5C; 
                color: black; 
                padding: 10px 24px; 
                border-color: black; 
                border-radius: 4px; 
                font-size: 15px;
                cursor: pointer; 
                margin-right: 8px;">Ice Storm in Texas
            </button>
            
            <button on:click={() => currentPage = 'WhatHappened'}
                style="background-color: #FF9C5C; 
                color: black; 
                padding: 10px 24px; 
                border-color: black; 
                border-radius: 4px; 
                font-size: 15px;
                cursor: pointer; 
                margin-right: 8px;">What Happened?
            </button>

            <button on:click={() => currentPage = 'Cause'} 
                style="background-color: #FF9C5C; 
                color: black; 
                padding: 10px 24px; 
                border-color: black; 
                border-radius: 4px; 
                font-size: 15px;
                cursor: pointer; 
                margin-right: 8px;">Potential Cause
            </button>
            
            <button on:click={() => currentPage = 'Correlation'}
                style="background-color: #FF9C5C; 
                color: black; 
                padding: 10px 24px; 
                border-color: black; 
                border-radius: 4px; 
                font-size: 15px;
                cursor: pointer; 
                margin-right: 8px;">Possible Correlation
            </button>


            <br>
            <h3><span style="font-family:Arial, Helvetica, sans-serif;">Free Explore:&nbsp;&nbsp;</span>
                <meta charset="utf-8">
            </h3>

            <button on:click={() => currentPage = 'Disasters'} 
                style="background-color: #FF9C5C; 
                color: black; 
                padding: 10px 24px; 
                border-color: black; 
                border-radius: 4px; 
                font-size: 15px;
                cursor: pointer; 
                margin-right: 8px;">Number of Disasters By State Each Year 1960 - 2023
            </button>
            
            <button on:click={() => currentPage = 'Carbon'}
                style="background-color: #FF9C5C; 
                color: black; 
                padding: 10px 24px; 
                border-color: black; 
                border-radius: 4px; 
                font-size: 15px;
                cursor: pointer; 
                margin-right: 8px;">Carbon Usage By State Each Year 1970 - 2021
            </button>
        </container>
        
        {#if currentPage === 'Disasters'}
            <div>
                
            </div>
            <Disasters />
        {:else if currentPage === 'Carbon'}
            <Carbon />
        {/if}
    </div>
</body>

<style>
    .wrapper {
    max-width: 970px;
    margin: auto;
    }
</style>