<script>
    import Uplot from "./uplot_v5.svelte";
    import Table from "./simple_table.svelte";
    import {fetch_ids, Timeout, get_sensor_list, load_calibrations, read_sensor} from "./load_cal.js";
    // import MyCollapse from "./MyCollapse.svelte";
    import {merge_data} from "./merge_v3.js";
    import Loader from "./Loader.svelte";
    // import {nipy_spectral}  from './js-colormaps-mod.js';
    // let colormap = nipy_spectral;
    import {colors} from "./colors.js";
    // calculate colors
    let data = [
        [1546300800, 1546387200],    // x-values (timestamps)
        [        35,         15],    // y-values (series 1) [        90,         15],    // y-values (series 2)
    ];
    import {data_fixed} from "./ten_days.js";
    data = data_fixed
    var labels = labels_fixed;
    import {labels_fixed} from "./labels.js";
    console.log('data_fixed.length', data_fixed.length)
    console.log('labels_fixed', labels_fixed);
    let count = 0;
    let cursor_data = [];
    var repeat;

    /*
    let data_ready = true;
    let table_data = [0, 1, 2];
    let labels = ['TIME', 'a', 'b'];
    let loading_id = -1;
    */ 
    import { onMount } from 'svelte';

    var last_ts = 0;
    var data_ready = false;
    var show_curves = labels.map(x=>{return true;});

    let fetchEvent = new Event('fetch');

    let ids = [100, 101, 102, 103, 104, 105, 106, 107, 108, 109,
        202,  206, 210, 214, 216, 217, 218, 219, 220,
        300, 301, 302, 303, 304, 305, 306];

    var plot_ids;
    var sensor_names;
    var table_data;
    var loading_message = 'loading';
    $: { 
        console.log('change loading_message', loading_message);
    }
    // $: { console.log('show_curves', show_curves); }
    onMount( async() => {
        loading_message = 'loading data'
        var history_v2 = data;

        plot_ids = [...labels];
        plot_ids.shift();
        sensor_names = [...plot_ids];
        console.log('labels', labels);
        console.log('show_curves', show_curves);
        data_ready = true;

        last_ts = history_v2[0][history_v2[0].length-1];
        console.log('latest ts',last_ts);
        var last_data = history_v2.map(x=>x[x.length-1]);
        console.log('last_data', last_data);
        last_data = last_data.map( (x,i) => Array.isArray(x) ? Number(x[0].toFixed(2)): ((x==null) ? -1 : Number(x.toFixed(2))))
        console.log('last_data', last_data);
        table_data = last_data.map(
            (x,i)=>  (i>0) ? (Array.isArray(x) ? x[0].toFixed(2): x.toFixed(2)): (new Date(x.toFixed(0)*1000)).toLocaleString()
        );
        
    });
    $: { 
        console.log('App.svelte ' + 'plot_ids', plot_ids);
        // console.log('App.svelte ' + 'show_curves', show_curves);
        if (plot_ids !== undefined) {
            for (const label of plot_ids) {
                // let id_list = sensor_list.find(elt => elt[1]==label)
                // console.log(id_list);
            }
        }
    }
    $: {
        // update table with cursor_data
        // console.log('cursor_data', cursor_data);
        if (cursor_data.length > 0) { 
            table_data = cursor_data.map(
                // (x,i)=>  (i>0) ? x[0].toFixed(2) : (new Date(x.toFixed(0)*1000)).toLocaleString()
                (x,i)=>  (x == null) ? -1 : ((i>0) ? (Array.isArray(x) ?
                    x[0].toFixed(2): x.toFixed(2)): (new Date(x.toFixed(0)*1000)).toLocaleString())
            );
            /*
            table_data[0] = (new Date(cursor_data[0].toFixed(0)*1000)).toLocaleString()
            for (let i=1; i<table_data.length; i++) {
                table_data[i] = cursor_data[i];
            }
            */
        }
    }
    function tableClick(e) {
        console.log('tableClick', e.detail);
        console.log('labels', labels);
        console.log('show_curves', show_curves);
        let idx =  labels.findIndex((elt)=>elt==e.detail.label);
        console.log(e.detail['label'], idx); // labels.findIndex((elt)=>elt==e.detail.label) );
        console.log('before tableClick changes ',show_curves);
        console.log(show_curves[idx]);
        show_curves[idx] = !show_curves[idx];
        console.log(show_curves[idx]);
        console.log('after tableClick changes ',show_curves);
        // just_finished_toggle_logy = true;
    }
</script>

        <body>
<style>

    .sidebar {
        height: 90vh;
        width: 250px;
        position: fixed;
        top: 0;
        left: 0;
        padding-top: 40px;
        font-size: 12px;
        overflow-y: scroll;
        /* background-color: lightblue; */
    }

    .sidebar div {
        padding: 8px;
        font-size: 12px;
        display: block;
    }
    .body-text {
        margin-left: 250px;
        font-size: 16px;
    }
    /* table thead {display: block;} */
    /* table tbody {height:300px; overflow-y:scroll; display:block;} */

</style>
<div class="sidebar">
    {#if data_ready}
        <Table table_data={table_data} labels={labels} show={show_curves}
            colors={colors} on:click={tableClick}/>
    <!--
    <MyCollapse label={'plot'} bind:choices={plot_ids} menu={sensor_names}/>
    -->
    {/if}
</div>

<div class="body-text">
    {#if data_ready}
        <Uplot data={data} labels={labels} show={show_curves} colors={colors} bind:cursor_data={cursor_data} />
    {:else}
        <p id="message"> {loading_message} </p>
        <!--
        <Loader loading={!data_ready}/>
        -->
    {/if}
</div>

        </body>
