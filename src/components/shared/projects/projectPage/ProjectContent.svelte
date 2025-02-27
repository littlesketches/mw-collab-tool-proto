<!-- PROJECT PAGE CONTENT PANE-->
<script>
    import showdown         from 'showdown'
	import { slide }        from 'svelte/transition'
    import { ui }           from '../../../../data/stores.js'
    import HWS_tags         from '../../forms/HWS_tags.svelte'
    import HWS_boxes        from '../../forms/HWS_boxes.svelte'
    import ProjectSources   from './ProjectSources.svelte'
    import ProjectLearnings from './ProjectLearnings.svelte'

    // HTML converter
    const converter = new showdown.Converter()
    // Reactive variables
    $: projectData = $ui.state.focus.projectData

    // HWS Key Values and conditions data
    $: themesData = {
        name:           "&mdash; Themes",
        schemaName:     "themes",
        label:          "Themes",
        array:          projectData.hws.themes
    }
    $: keyValuesData = {
        name:           "&mdash; Values",
        schemaName:     "hwsValues",
        label:          "Waterway value",
        array:          projectData.hws.values
    }
    $: conditionsData = {
        name:           "&mdash; Conditions",
        schemaName:     "hwsConditions",
        label:          "Waterway condition",
        array:          projectData.hws.conditions
    }

    // Slideable pane visibility
    const visibility = {
        hwsDetails:         true,
        aboutDetails:       false,
        learningsDetails:   false,
        sourcesDetails:     false
    }
    // Pane toggle labels
    $: aboutLabel       = !visibility.aboutDetails ? 'Show more project details' : 'Hide project details' 
    $: hwsLabel         = !visibility.hwsDetails ? 'Show more impact details' : 'Hide impact details' 
    $: learningsLabel   = !visibility.learningsDetails ? 'Show learnings' : 'Hide learnings details' 
    $: sourcesLabel     = !visibility.sourcesDetails ? 'Show more sources details' : 'Hide sources details' 

    function togglePane(){
        visibility[this.id] = !visibility[this.id]
        console.log(`Toggling ${this.id} to `, visibility[this.id])
    };

</script>

<!-- COMPONENT HTML MARKUP-->
<section id = "project-content" class = 'content-pane'>
    <!-- HWS IMPACT SECTION-->
    <div class = 'short-desc'>
        {@html converter.makeHtml(projectData.about.shortDescription)}  
    </div>
    {#if visibility.aboutDetails }
        <div transition:slide="{{duration: 1200}}">
            <h3>&mdash;&mdash; Project details</h3>
            {#if projectData.about.longDescription}
                {@html converter.makeHtml(projectData.about.longDescription)}
            {/if}
            {#if projectData.about.history}
                <h3 class ='margin-top'>&mdash;&mdash; Project history</h3>
                {@html converter.makeHtml(projectData.about.history)}
            {/if}
            {#if !projectData.about.longDescription && !projectData.about.history}
            <p>Unfortunately more descriptive details about this project have not been added (yet).</p>
            {/if}
        </div>
    {/if}
    <div class = "collapse__body"  transition:slide>
        <div id = "aboutDetails" class="collapse__header" type="button" 
            class:selected="{visibility.aboutDetails}" on:click={togglePane}>
            <div class="toggle-label">{@html aboutLabel}</div>
            <div class="toggle-icon down">&#8595;</div>
        </div>
    </div>
    <hr>

    <!-- HWS IMPACT SECTION-->
    <h3>&mdash;&mdash;&mdash; Waterways impact</h3>
    {#if visibility.hwsDetails }
        <div transition:slide="{{duration: 1200}}">
            <p>Every waterways project can be categorised by the waterways themes, key values an conditions which they impact.</p>
            <HWS_tags data={themesData}/>
            <HWS_boxes data={keyValuesData}/>
            <HWS_boxes data={conditionsData}/>
        </div>
    {/if}
    <div class = "collapse__body"  transition:slide>
        <div id = "hwsDetails" class="collapse__header" type="button" 
            class:selected="{visibility.hwsDetails}" on:click={togglePane}>
            <div class="toggle-label">{@html hwsLabel}</div>
            <div class="toggle-icon down">&#8595;</div>
        </div>
    </div>
    <hr>

    <!-- PROJECT LEARNINGS: optional -->
    <h3>&mdash;&mdash; Project learnings</h3>
    {#if projectData.learnings.cultural || projectData.learnings.innovation || projectData.learnings.general.length > 0 || projectData.learnings.worked.length > 0 || projectData.learnings.failed.length > 0 }
        <p>{@html projectData.leadOrg} has shared learnings from this project.</p>

        {#if visibility.learningsDetails }
        <div transition:slide="{{duration: 1200}}">
            {#if projectData.learnings.innovation}
                <h3 class ='margin-top'>&mdash;&mdash; Sharing project innovation</h3>
                {@html converter.makeHtml(projectData.learnings.innovation)}
            {/if}
            {#if projectData.learnings.cultural}
                <h3 class ='margin-top'>&mdash;&mdash; Sharing cultural practices</h3>
                {@html converter.makeHtml(projectData.learnings.cultural)}
            {/if}
            <ProjectLearnings/>
        </div>
        {/if}
        <div class = "collapse__body"  transition:slide>
            <div id = "learningsDetails" class="collapse__header" type="button" 
                class:selected="{visibility.learningsDetails}" on:click={togglePane}>
                <div class="toggle-label">{@html learningsLabel}</div>
                <div class="toggle-icon down">&#8595;</div>
            </div>
        </div>
        <hr>
    {:else}
        <p>{@html projectData.leadOrg} has not (yet) shared any insights and lessons from <i>{@html projectData.name}</i>.</p>
    {/if}


    <!-- MORE INFO AND SOURCES: optional-->
    <h3>&mdash;&mdash; Where to find more information</h3>
    {#if projectData.links.length > 0 || [...new Set(Object.values(projectData.leadContact))][0]}
        <!-- Message about the information provided for lnks and contacts-->
        {#if projectData.links.length > 0 && [...new Set(Object.values(projectData.leadContact))][0]}
        <p>{@html projectData.leadOrg} has provided links to further information and contact details to learn more about {@html projectData.name}.</p>
        {:else if projectData.links.length > 0}
        <p>{@html projectData.leadOrg} has provided links to more information about <i>{@html projectData.name}</i>.</p>
        {:else if [...new Set(Object.values(projectData.leadContact))][0]}
        <p>{@html projectData.leadOrg} has provided contact details to get in touch about {@html projectData.name} in {@html new Date(projectData.status.dates.lastUpdate).toLocaleDateString('en-GB', { year: 'numeric', month: 'long', day: 'numeric' }) }.</p>
        {/if}

        {#if visibility.sourcesDetails }
            <ProjectSources/>
        {/if}
        <div class = "collapse__body"  transition:slide>
            <div id = "sourcesDetails" class="collapse__header" type="button" 
                class:selected="{visibility.sourcesDetails}" on:click={togglePane}>
                <div class="toggle-label">{@html sourcesLabel}</div>
                <div class="toggle-icon down">&#8595;</div>
            </div>
        </div>
        <hr>
    {:else}
     <p>At this stage, {@html projectData.leadOrg} has not provided links to any further reources.</p>
    {/if}

</section>


<!------ STYLE ------->
<style>
    section{
        grid-area:          2 / 1 / 6 / 2; 
        padding:            0 1rem;
    }
    h3.margin-top{
        margin-top:         3rem;
    }
    .short-desc{
        padding-bottom:     0.5rem;
    }
    /* COLLAPSIBLE PANE STYLING */
	.collapse__header {
        display:            flex;
        justify-content:    right;
	    transition:         background 200ms ease-in-out;
        cursor:             pointer;
	}
    .collapse__header{
	    transition:         all 200ms ease-in-out;
    }
	.collapse__header:hover {
        font-weight:        600;
	}
	.collapse__body {
        display:            grid;
	    padding:            0rem 0;
	}
    .toggle-label{
        margin-right:       1rem;
        font-size:          0.75rem;
        font-weight:        300;
    }
    .toggle-icon{
	    transition:         all 800ms ease-in-out;
    }
    .selected .toggle-icon{
        transform:          rotate(180deg);
    }

</style>