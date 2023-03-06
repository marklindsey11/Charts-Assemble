<script lang="ts">
    import { createEventDispatcher } from 'svelte';
    import TimestampDetail from '../timestamp/TimestampDetail.svelte';
    import BiHistogram from './BiHistogram.svelte';

    export let showTooltip = false;
    export let fillColor: string;
    export let hoverColor: string;
    export let baselineStrokeColor: string;
    export let biData;
    export let width = 314;
    export let height = 217;

    //for chart
    export let xLabel: string;
    export let yLabel: string;

    let aggrType: string = 'count';

    let showMenu = false;
    let collapse = false;

    const dispatch = createEventDispatcher();

    const onSelect = () => {
        dispatch('selectAggrType', {
            typ: aggrType
        });
    };
</script>

<div>
    <div class="w-full bg-gray-100 pb-1 pt-1" style="margin-bottom:2px" on:click={()=>{collapse=!collapse}}>
        {xLabel} vs {yLabel}
    </div>
    {#if !collapse}
        <div class="flex" style="z-index:1">
            <span>X: </span>
            <div class="rounded border border-solid variable pl-1 pr-1">
                {xLabel}
            </div>
            <div class="grow" />
            <span>Y: </span>
            <div
                class="rounded border border-solid cursor-grabbing variable pl-1 pr-1"
            >
                {yLabel}
                <span class="aggregation-select">
                    <button
                        on:click={() => {
                            showMenu = !showMenu;
                        }}
                    >
                        {aggrType}
                    </button>
                    {#if showMenu}
                        <div
                            class="absolute right-2 mt-2 w-20 origin-top-right rounded-md bg-white hover:bg-gray-100 dark:bg-zinc-900 shadow-lg border border-gray-50 dark:border-gray-800 ring-1 ring-black ring-opacity-5 focus:outline-none transform scale-100"
                            style="z-index:9; cursor:pointer"
                        >
                            {#each ['count', 'mean', 'sum', 'min', 'max'] as typ}
                                <span
                                    class="text-gray-700 dark:text-gray-200 block px-2 py-1 text-sm"
                                    on:click={() => {
                                        aggrType = typ;
                                        onSelect();
                                    }}>{typ}</span
                                >
                            {/each}
                        </div>
                    {/if}
                </span>
            </div>
        </div>
        {#if biData.chartType === 'histogram'}
            <BiHistogram
                {showTooltip}
                {fillColor}
                {hoverColor}
                {baselineStrokeColor}
                data={biData.data}
                {width}
                {height}
                xLabel={yLabel}
                yLabel={xLabel}
            />
        {:else if biData.chartType === 'linechart'}
            <TimestampDetail
                data={biData.data?.rollup?.results}
                xAccessor="ts_end"
                yAccessor="mean"
                height={160}
                {width}
                interval={biData.data?.interval}
            />
        {/if}
    {/if}
</div>

<style>
    .variable {
        min-width: 80px;
        height: 24px;
        cursor: grab;
    }

    .aggregation-select {
        background-color: #bdbdbd;
        font-size: 10px;
    }
</style>
