<!-- PROJECT PAGE CONTENT PANE-->
<script>
    import showdown         from 'showdown'
	import { slide }    from 'svelte/transition'
    import { ui }       from '../../../../data/stores.js'
    import HWS_tags     from '../../forms/HWS_tags.svelte'
    import HWS_boxes    from '../../forms/HWS_boxes.svelte'
    import Sources      from './Sources.svelte'

    export let leadProjects = []

    // HTML converter
    const converter = new showdown.Converter()

    // Reactive variables
    $: stakeholderData = $ui.state.focus.stakeholderData 

    // HWS Key Values and conditions data
    $: themesData = {
        name:           "&#8212; Themes",
        array:          [...new Set(leadProjects.map(d => d.hws.themes).flat())]
    }
    $: keyValuesData = {
        name:           "&#8212; Values",
        schemaName:     "hwsValues",
        array:          [...new Set(leadProjects.map(d => d.hws.values).flat())], 
        label:          "Waterway value"
    }
    $: conditionsData = {
        name:           "&#8212; Conditions",
        schemaName:     "hwsConditions",
        array:          [...new Set(leadProjects.map(d => d.hws.conditions).flat())],
        label:          "Waterway condition"
    }

    // Slideable pane visibility
    const visibility = {
        hwsDetails:         true,
        aboutDetails:       false,
    }
    // Pane toggle labels
    $: aboutLabel = !visibility.aboutDetails ? 'Show more details' : 'Hide details' 
    $: hwsLabel = !visibility.hwsDetails ? 'Show more impact details' : 'Hide impact details' 

    function togglePane(){
        visibility[this.id] = !visibility[this.id]
        console.log(`Toggling ${this.id}  to `, visibility[this.id])
    };

    console.log('Themes data: ',    [...new Set(leadProjects.map(d => d.hws.themes).flat())])
    console.log('Values data: ',    [...new Set(leadProjects.map(d => d.hws.values).flat())])
    console.log('Conditions data: ', [...new Set(leadProjects.map(d => d.hws.conditions).flat())])

</script>

<!-- COMPONENT HTML MARKUP-->
<section class = 'content-pane'>
    <!-- HWS IMPACT SECTION-->
    <div class = 'short-desc'>
        {@html converter.makeHtml(stakeholderData.about.shortDescription)}  
    </div>
    {#if visibility.aboutDetails }
        <div transition:slide>
            <h3>&#8212;&#8212; About</h3>
            {@html converter.makeHtml(stakeholderData.about.longDescription)}
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
    <h3>&#8212;&#8212;&#8212; Impact on waterways</h3>
    <p>{stakeholderData.name} is involved with projects that impact our waterways in the following ways:</p>
    {#if visibility.hwsDetails }    
    <HWS_tags data={themesData}/>
    <HWS_boxes data={keyValuesData}/>
    <HWS_boxes data={conditionsData}/>
    {/if}
    <div class = "collapse__body"  transition:slide>
        <div id = "hwsDetails" class="collapse__header" type="button" 
            class:selected="{visibility.hwsDetails}" on:click={togglePane}>
            <div class="toggle-label">{@html hwsLabel}</div>
            <div class="toggle-icon down">&#8595;</div>
        </div>
    </div>
    <hr>
</section>


<!------ STYLE ------->
<style>
    section{
        grid-area:  2 / 1 / 6 / 2; 
        padding: 0 1rem;
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
    .collapse__header .toggle-icon{
	    transition: all 200ms ease-in-out;
    }
    .toggle-label{
        margin-right: 1rem;
        font-size: 0.75rem;
        font-weight: 300;
    }
    .selected .toggle-icon{
        transform: rotate(180deg);
    }

	.collapse__header:hover {
        font-weight: 600;
	}
	.collapse__body {
	    padding: 0rem 0;
        display: grid;
	}
</style>