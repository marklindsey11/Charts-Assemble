<script lang="ts">
    import { createEventDispatcher } from 'svelte';
    import BiHistogram from './BiHistogram.svelte';
    import BiLine from './BiLine.svelte';
    import { TIMESTAMPS } from '../../data-types/pandas-data-types';

    import BivariateIcon from '../../icons/BivariateIcon.svelte';

    export let biData;

    //for chart
    export let xLabel: string;
    export let yLabel: string;
    export let xType: string;
    export let yType: string;
    export let timeOffset: string; // such as "Y","M"

    let aggrType: string = 'count';

    let showAggrMenu = false;
    let showTimeStepMenu = false;
    let collapse = false;

    let offsetAliases = {
        year: 'Y',
        month: 'M',
        week: 'W',
        day: 'D',
        hour: 'H',
        minute: 'T',
        second: 'S'
    };

    const dispatch = createEventDispatcher();

    const onSelectAggrType = () => {
        dispatch('selectAggrType', {
            typ: aggrType
        });
    };

    const onDelete = () => {
        dispatch('delete');
    }

    const onSelectTimeOffset = () => {
        dispatch('selectTimeOffset', {
            timeOffset: offsetAliases[timestep]
        });
    };

    let timestep = Object.keys(offsetAliases).filter(
        d => offsetAliases[d] === timeOffset
    )[0];
</script>

<div>
    <div
        class="w-full bg-gray-100 flex gap-2"
        style="margin-bottom:2px"
        on:click={() => {
            collapse = !collapse;
        }}
    >
        <BivariateIcon size="16px" />
        <span>{xLabel} vs {yLabel}</span>
    </div>
    {#if !collapse}
        <div class="flex" style="z-index:1">
            <span>X: </span>
            <div class="rounded border border-solid variable pl-1 pr-1">
                {xLabel}
                {#if TIMESTAMPS.has(xType)}
                    <span class="timestep-select">
                        <button
                            on:click={event => {
                                event.stopPropagation();
                                showTimeStepMenu = !showTimeStepMenu;
                            }}
                            >{timestep}
                        </button>
                        {#if showTimeStepMenu}
                            <div
                                class="absolute mt-2 w-20 origin-top-right rounded-md bg-white hover:bg-gray-100 dark:bg-zinc-900 shadow-lg border border-gray-50 dark:border-gray-800 ring-1 ring-black ring-opacity-5 focus:outline-none transform scale-100"
                                style="z-index:9; cursor:pointer"
                            >
                                {#each ['year', 'month', 'week', 'day', 'hour', 'minute', 'second'] as step}
                                    <span
                                        class="text-gray-700 dark:text-gray-200 block px-2 py-1 text-sm"
                                        on:click={event => {
                                            event.stopPropagation();
                                            timestep = step;
                                            onSelectTimeOffset();
                                        }}>{step}</span
                                    >
                                {/each}
                            </div>
                        {/if}
                    </span>
                {/if}
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
                            showAggrMenu = !showAggrMenu;
                        }}
                    >
                        {aggrType}
                    </button>
                    <!-- {#if showAggrMenu}
                        <div
                            class="absolute right-2 mt-2 w-20 origin-top-right rounded-md bg-white hover:bg-gray-100 dark:bg-zinc-900 shadow-lg border border-gray-50 dark:border-gray-800 ring-1 ring-black ring-opacity-5 focus:outline-none transform scale-100"
                            style="z-index:9; cursor:pointer"
                        >
                            {#each ['count', 'mean', 'sum', 'min', 'max'] as typ}
                                <span
                                    class="text-gray-700 dark:text-gray-200 block px-2 py-1 text-sm"
                                    on:click={() => {
                                        aggrType = typ;
                                        onSelectAggrType();
                                    }}>{typ}</span
                                >
                            {/each}
                        </div>
                    {/if} -->
                </span>
            </div>
        </div>
        {#if biData.chartType === 'histogram'}
            <BiHistogram data={biData.data} xLabel={yLabel} yLabel={xLabel} />
        {:else if biData.chartType === 'linechart'}
            <BiLine data={biData.data} {xLabel} {yLabel} />
        {/if}
        <div class="flex" style="z-index:1">
            <div
                class="rounded border border-solid cursor-grabbing variable pl-1 pr-1"
            >
                Aggregation Type:
                <span class="aggregation-select">
                    <button
                        on:click={() => {
                            showAggrMenu = !showAggrMenu;
                        }}
                    >
                        {aggrType}
                    </button>
                    {#if showAggrMenu}
                        <div
                            class="absolute right-2 mt-2 w-20 origin-top-right rounded-md bg-white hover:bg-gray-100 dark:bg-zinc-900 shadow-lg border border-gray-50 dark:border-gray-800 ring-1 ring-black ring-opacity-5 focus:outline-none transform scale-100"
                            style="z-index:9; cursor:pointer"
                        >
                            {#each ['count', 'mean', 'sum', 'min', 'max'] as typ}
                                <span
                                    class="text-gray-700 dark:text-gray-200 block px-2 py-1 text-sm"
                                    on:click={() => {
                                        aggrType = typ;
                                        onSelectAggrType();
                                    }}>{typ}</span
                                >
                            {/each}
                        </div>
                    {/if}
                </span>
            </div>
            <div class="grow" />
            <div class="flex border border-solid rounded">
                <button
                    class="flex pl-2 pr-2 text-gray-500 hover:bg-gray-100"
                    on:click={() => {
                        onDelete();
                    }}
                    >Delete
                </button>
            </div>
        </div>
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
    .timestep-select {
        background-color: #bdbdbd;
        font-size: 10px;
    }
</style>
