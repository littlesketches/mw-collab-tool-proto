<!-- DISCOVER PAGE COMPONENT-->
<script>
	import { slide, fly }       from "svelte/transition";
	import MultiSelect          from '../../shared/forms/MultiSelect.svelte';
	import Map                  from '../../shared/map/Map.svelte';
    import { ui }               from '../../../data/stores.js'
    import { app }              from '../../../data/realm.js'
    import { componentContent, infoModal } from '../../../data/content.js'
    import { keyValues, conditions, themes, catchments, subcatchments, leadOrg, leadOrgType, partnerOrg, projectType, projectStage, projectScale }  from '../../../data/selectorLists.js'

    export let newSearch = true

    ////// COLLAPSIBLE SEARCH PANES AND UI ////
	const paneVisbility= {
        byOutcomes:             false,
        byLocation:             false,
        byCharacteristics:      false,
        byStakeholders:         false
    }

    function togglePane(){
        Object.keys(paneVisbility).map( key => {
            if(this.id !== key){ paneVisbility[key] = false }
        })
        paneVisbility[this.id] = ! paneVisbility[this.id]
    };

    ////// SEARCH PROJECTION OBJECT /////
    if(newSearch){
        $ui.search.criteria.project = {
            name:               [],
            hws: {
                themes:         [],
                conditions:     [],
                values:         [],
            },
            meta: {
                type:           [],
                scale:          [],
            },
            status: {
                stage:          []
            },
            location: {
                catchments:     [],
                subcatchments:  [],
            },
            leadOrg:            [],
            leadOrgType:        [],
            partnerOrgs:        []
        }
    }


    async function handleSearch(){
        // Setup query
        const query = {}
        if($ui.search.criteria.project.hws.values.length > 0)               query["hws.values"]     = {$in: $ui.search.criteria.project.hws.values}
        if($ui.search.criteria.project.hws.conditions.length > 0)           query["hws.conditions"] = {$in: $ui.search.criteria.project.hws.conditions}
        if($ui.search.criteria.project.hws.themes.length > 0)               query["hws.themes"]     = {$in: $ui.search.criteria.project.hws.themes}
        if($ui.search.criteria.project.status.stage.length > 0)             query["status.stage"]   = {$in: $ui.search.criteria.project.status.stage}
        if($ui.search.criteria.project.location.catchments.length > 0)      query["location.catchments"]    = {$in: $ui.search.criteria.project.location.catchments}
        if($ui.search.criteria.project.location.subcatchments.length > 0)   query["location.subcatchments"] = {$in: $ui.search.criteria.project.location.subcatchments}
        if($ui.search.criteria.project.leadOrg.length > 0)                  query["leadOrg"]        = {$in: $ui.search.criteria.project.leadOrg}
        if($ui.search.criteria.project.leadOrgType.length > 0)              query["leadOrgType"]    = {$in: $ui.search.criteria.project.leadOrgType}
        if($ui.search.criteria.project.partnerOrgs.length > 0)              query["partnerOrgs"]    = {$in: $ui.search.criteria.project.partnerOrgs}

        const countResults = await app.data.collections.projects.aggregate([
                { $match:       query },
                { $count:       "name"}
            ])

        if(countResults[0].name > 50){
            const confirm = window.confirm(`You're search criteria came back with ${countResults[0].name } results. Tap "OK' to see them all (it might a few seconds) or "Cancel" to search again) `)
            if (confirm) {
                $ui.search.results.project  = await app.data.collections.projects.aggregate([
                    { $match:       query },
                    { $sort:       { name: 1,  leadOrg: 1 } }
                ])

                console.log("Realm project search results: ", $ui.search.results.project )

                // Temporary info box for project search
                if($ui.infoModal.showNotes && componentContent.messageModal.stakeholderSearch){
                    $ui.infoModal.message = infoModal.projectSearch
                    componentContent.messageModal.projectSearch = null
                }

                // Set UI based on curent page
                switch($ui.page){
                    case 'discover':
                        $ui.byPage.discover.main = 'list'
                        $ui.byPage.discover.projectSearch.isMade = true
                        break

                    case 'share':
                        $ui.byPage.share.main = 'list'
                        $ui.byPage.share.projectSearch.isMade = true
                        break
                };
                window.scrollTo({top: 0, behavior: 'smooth'});
            }
        }
    };

    function handleClose(){
        $ui.byPage.discover.main = 'list'
        window.scrollTo({top: 0, behavior: 'smooth'});
    };

    function handleClearSearch(){
        console.log('Clearing the search')
        $ui.search.criteria.project = null
        for (const key of Object.keys(paneVisbility)){ paneVisbility[key] = false}
        window.scrollTo({top: 0, behavior: 'smooth'});
    };

    // Keep count of search param number
    $: noSearchParams = typeof Object.values($ui.search.results.project) === 'undefined' ? 0 : Object.values($ui.search.results.project).flat().length

</script>


<!-- COMPONENT MARKUP-->
<section id = 'project-search' in:fly="{{duration: 800, x: -100}}" out:fly="{{duration: 200, x: -100}}">
    <!-- Close search option-->
    {#if $ui.page === 'discover'}
    <div class='close-container'>
        <div on:click={handleClose} class='close-button'>
            &#8592; Close search filter
        </div>
    </div>
    {/if}

    <!-- SEARCH THEMES-->
    <!-- by Waterways impact -->
    <div class = "container">
        <div id = "byOutcomes" class="collapse__header" type="button" 
            class:selected="{paneVisbility.byOutcomes}" on:click={togglePane}>
            <h3>Waterway outcomes</h3>
            <div class="toggle-icon down">&#8595;</div>
        </div>
        {#if paneVisbility.byOutcomes}
        <div class = "collapse__body"  transition:slide>
            <div class = "group-description">
                <p>Waterway <em>key values</em>, <em>conditions</em> and <em>themes</em> all relate to their definitions and use in the Healthy Waterways Strategy.</p>
            </div>
            <div class = 'multi-select-container' style="z-index:18">
                <h4>{@html keyValues.label}</h4>
                <MultiSelect id={keyValues.name} bind:value={$ui.search.criteria.project.hws.values} placeholder={keyValues.placeholder} >
                    <option disabled selected value></option>
                    {#each keyValues.list as name}
                    <option value={name}>{@html name}</option>
                    {/each}
                </MultiSelect>
            </div>

            <div class = 'multi-select-container'  style="z-index:17">
                <h4>{@html conditions.label}</h4>
                <MultiSelect id={conditions.name} bind:value={$ui.search.criteria.project.hws.conditions}  placeholder={conditions.placeholder} >
                    <option disabled selected value></option>
                    {#each conditions.list as name}
                    <option value={name}>{@html name}</option>
                    {/each}
                </MultiSelect>
            </div>

            <div class = 'multi-select-container' style="z-index:16">
                <h4>{@html themes.label}</h4>
                <MultiSelect id={themes.name} bind:value={$ui.search.criteria.project.hws.poThemes}   placeholder={themes.placeholder} >
                    <option disabled selected value></option>
                    {#each themes.list as name}
                    <option value={name}>{@html name}</option>
                    {/each}
                </MultiSelect>
            </div>
        </div>
        {/if}
    </div>

    <!-- by Project location -->
    <div class = "container">
        <div id = "byLocation" class="collapse__header" type="button" 
            class:selected="{paneVisbility.byLocation}" on:click={togglePane}>
            <h3>Location</h3>
            <div class="toggle-icon down">&#8595;</div>
        </div>
        {#if paneVisbility.byLocation}
        <div class = "collapse__body"  transition:slide>
            <div class = "group-description">
                <p>Location search fields allow you to narrow down projects to only those that affect only certain <strong>catchments or subcatchment(s)</strong>.
                    Because these search fields are treated independently, a selection in one will turn off (hide) the other.
                    Note: In the <em>production<em/> version, location searches will be 'smarter' and linked to the map view. This is not currenly functional, however a map is shown as an indicative interface
                </p>
            </div>
            {#if $ui.search.criteria.project.location.catchments.length === 0}
            <div class = "collapse__body"  transition:slide>
                <div class = 'multi-select-container' style="z-index:15">
                    <h4>{@html subcatchments.label}</h4>
                    <MultiSelect id={subcatchments.name} bind:value={$ui.search.criteria.project.location.subcatchments} placeholder={subcatchments.placeholder} >
                        <option disabled selected value></option>
                        {#each subcatchments.list as name}
                        <option value={name}>{@html name}</option>
                        {/each}                
                    </MultiSelect>
                </div>
            </div>
            {/if}
            {#if $ui.search.criteria.project.location.subcatchments.length === 0}
            <div class = "collapse__body"  transition:slide>
                <div class = 'multi-select-container' style="z-index:14">
                    <h4>{@html catchments.label}</h4>
                    <MultiSelect id={catchments.name} bind:value={$ui.search.criteria.project.location.catchments} placeholder={catchments.placeholder} >
                        <option disabled selected value></option>
                        {#each catchments.list as name}
                        <option value={name}>{@html name}</option>
                        {/each}                
                    </MultiSelect>
                </div>
            </div>
            {/if}
            <div style="z-index:13">
                <Map/>
            </div>
        </div>
        {/if}
    </div>

    <!-- by Project characeristics -->
    <div class = "container">
        <div id = "byCharacteristics" class="collapse__header" type="button" 
            class:selected="{paneVisbility.byCharacteristics}" on:click={togglePane}>
            <h3>Project characteristics</h3>
            <div class="toggle-icon down">&#8595;</div>
        </div>
        {#if paneVisbility.byCharacteristics}
        <div class = "collapse__body"  transition:slide>
            <div class = "group-description">
                <p>These high level characteristics help to classify projects into logical and useful groupings.
                    Note: In a <em>production<em/> version where more timing information is collected on projects, date filters are expected to be added.
                </p>
            </div>
            <div class = 'multi-select-container' style="z-index:12">
                <h4>{@html projectType.label}</h4>
                <MultiSelect id = {projectType.name} bind:value={$ui.search.criteria.project.meta.type} placeholder={projectType.placeholder} >
                    <option disabled selected value></option>
                    {#each projectType.list as name}
                    <option value={name}>{@html name}</option>
                    {/each}                 
                </MultiSelect>
            </div>
            <div class = 'multi-select-container' style="z-index:11">
                <h4>{@html projectStage.label}</h4>
                <MultiSelect id = {projectStage.name} bind:value={$ui.search.criteria.project.status.stage} placeholder={projectStage.placeholder} >
                    <option disabled selected value></option>
                    {#each projectStage.list as name}
                    <option value={name}>{@html name}</option>
                    {/each}                
                </MultiSelect>
            </div>
            <div class = 'multi-select-container' style="z-index:10">
                <h4>{@html projectScale.label}</h4>
                <MultiSelect id = {projectScale.name} bind:value={$ui.search.criteria.project.meta.scale} placeholder={projectScale.placeholder} >
                    <option disabled selected value></option>
                    {#each projectScale.list as name}
                    <option value={name}>{@html name}</option>
                    {/each}                
                </MultiSelect>
            </div>
        </div>
        {/if}
    </div>

    <!-- by Stakeholders -->
    <div class = "container">
        <div id = "byStakeholders" class="collapse__header" type="button" 
            class:selected="{paneVisbility.byStakeholders}" on:click={togglePane}>
            <h3>Stakeholders</h3>
            <div class="toggle-icon down">&#8595;</div>
        </div>
        {#if paneVisbility.byStakeholders}
        <div class = "collapse__body" transition:slide>
            <div class = "group-description">
                <p>Every project is assigned one organisation as the 'lead', but can also be assigned any number of partner organisations.</p>
            </div>

            <div class = 'multi-select-container' style="z-index:9">
                <h4>{@html leadOrg.label}</h4>
                <MultiSelect id = {leadOrg.name} bind:value={$ui.search.criteria.project.leadOrg} placeholder={leadOrg.placeholder} >
                    <option disabled selected value></option>
                    {#each leadOrg.list as name}
                    <option value={name}>{@html name}</option>
                    {/each}                
                </MultiSelect>
            </div>
            <div class = 'multi-select-container' style="z-index:8">
                <h4>{@html leadOrgType.label}</h4>
                <MultiSelect id = {leadOrgType.name} bind:value={$ui.search.criteria.project.leadOrgType} placeholder={leadOrgType.placeholder} >
                    <option disabled selected value></option>
                    {#each leadOrgType.list as name}
                    <option value={name}>{@html name}</option>
                    {/each}                
                </MultiSelect>
            </div>
            <div class = 'multi-select-container' style="z-index:7">
                <h4>{@html partnerOrg.label}</h4>
                <MultiSelect id = {partnerOrg.name} bind:value={$ui.search.criteria.project.partnerOrgs} placeholder={partnerOrg.placeholder} >
                    <option disabled selected value></option>
                    {#each partnerOrg.list as name}
                    <option value={name}>{@html name}</option>
                    {/each}                
                </MultiSelect>
            </div>
        </div>
        {/if}
    </div>

    <div class = "button-container">
        <button on:click|preventDefault={handleSearch}>Search for projects</button>
    </div>
    {#if noSearchParams > 0}
    <div class = "clear-search-container" on:click={handleClearSearch}>
        Clear the search parameters
    </div>
    {/if}

</section>


<!------ STYLE ------->
<style> 
    section{
        grid-area:              main;
    }
    h3, h4{ 
        margin-block-start:     0;
        margin-block-end:       0;
        padding:                0 1rem 0 0.5rem;
        line-height:            1.25;
    }
    h4{
        margin-block-start:     0.75rem;        
        font-size:              1rem;
        font-weight:            600;
    }
    .multi-select-container{
        display:                grid;
        grid-template-columns:  1fr 3fr;
        padding:                1rem 0;
    }
    .container * {
        box-sizing:             border-box;
    }
    .close-container{
        display:                flex;        
        justify-content:        flex-end;
        margin-bottom:          1rem;
    }
    .close-button, 
    .clear-search-container{
        font-weight:            600;
        font-size:              0.8rem;
        cursor:                 pointer;
    }
    .close-button:hover, 
    .clear-search-container:hover{
        text-decoration:        underline;
    }
    .button-container, 
    .clear-search-container{
        margin-top:             1rem;
    }
    .clear-search-container{
        text-align:             right;
    }

    /* COLLAPSIBLE PANE STYLING */
	.collapse__header {
        pointer-events:         bounding-box;
        cursor:                 pointer;
        display:                flex;
        justify-content:        space-between;
	    padding:                1rem 0rem;
	    border-top:             2px dotted var(--darkGrey);
	    transition:             background 200ms ease-in-out;
	}
    .collapse__header .toggle-icon{
        margin-right:           1rem;
	    transition:             all 200ms ease-in-out;
    }
    .selected .toggle-icon{
        transform:              rotate(180deg);
    }
	.collapse__header.selected,
	.collapse__header:hover {
	    background:             var(--darkGrey);
        color:                  #fff;
	}
	.collapse__body {
	    padding:                1rem 0rem;
        display:                grid;
	}
    .group-description{
        font-size:              0.8rem;
        font-weight:            300;
        margin:                 0 0.75rem;
    }
</style>