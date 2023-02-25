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

    let aggrType: string;

    const dispatch = createEventDispatcher();

    const onSelect = () => {
        dispatch('selectAggrType',{
            typ: aggrType
        })
    };
</script>

<div>
    Aggregation Type:
    <select
        class="rounded border border-6 bg-gray-100 hover:border-gray-300"
        bind:value={aggrType}
        on:change={onSelect}
    >
        <option value={'mean'}>mean</option>
        <option value={'count'}>count</option>
        <option value={'sum'}>sum</option>
        <option value={'min'}>min</option>
        <option value={'max'}>max</option>
    </select>
    {#if biData.chartType === 'histogram'}
        <BiHistogram
            {showTooltip}
            {fillColor}
            {hoverColor}
            {baselineStrokeColor}
            data={biData.data}
            {width}
            {height}
            xLabel={xLabel}
            yLabel={yLabel}
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
</div>

<div>
    {#if biData.chartType === "histogram"}
    <BiHistogram
    showTooltip={showTooltip}
    fillColor={fillColor}
    hoverColor={hoverColor}
    baselineStrokeColor={baselineStrokeColor}
    data={biData.data}
    width={width}
    height={height}
    />
    {:else if biData.chartType === "linechart"}
    <TimestampDetail
    data={biData.data?.rollup?.results}
    xAccessor="ts_end"
    yAccessor="mean"
    height={160}
    width={width}
    interval={biData.data?.interval}
    />
    {/if}
</div>