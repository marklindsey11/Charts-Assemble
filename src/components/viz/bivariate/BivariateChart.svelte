<script lang='ts'>
    import TimestampDetail from "../timestamp/TimestampDetail.svelte";
    import BiHistogram from "./BiHistogram.svelte";

    export let showTooltip = false;
    export let fillColor: string;
    export let hoverColor: string;
    export let baselineStrokeColor: string;
    export let biData;
    export let width = 314;
    export let height = 217;
</script>


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