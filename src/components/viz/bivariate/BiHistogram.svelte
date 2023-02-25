<script lang="ts">
    import { tweened } from 'svelte/motion';
    import { cubicOut as easing } from 'svelte/easing';
    import { scaleLinear } from 'd3-scale';
    import { bisector } from 'd3-array';
    import _, { create } from 'lodash';
    import type { IHistogram } from '../../../common/exchangeInterfaces';
    import BiHistogramTooltip from './BiHistogramTooltip.svelte';


    // import LayerCake component
    import { LayerCake, Svg } from 'layercake';
    import {scaleBand} from 'd3-scale';
    import Column from './Column.svelte';
    import Bar from './Bar.svelte';
    import AxisX from './AxisX.svelte';
    import AxisY from './AxisY.svelte';

    export let data;
    export let width = 60;
    export let height = 19;
    export let time = 1000;
    export let fillColor: string; //'hsl(340, 70%, 70%)';
    export let hoverColor: string = fillColor;
    export let baselineStrokeColor: string;
    export let separate = true;
    $: separateQuantity = separate ? 0.25 : 0;
    export let showTooltip = false;

    // rowsize for table
    export let left = 60;
    export let right = 56;
    export let top = 20;
    export let bottom = 22;
    export let buffer = 4;

    // dots and labels
    export let vizOffset = 0;
    export let xLabel: string;
    export let yLabel: string;

    let aggrType: string;

    const tw = tweened(0, { duration: time, easing });

    const lowValue = tweened(0, { duration: time / 2, easing });
    const highValue = tweened(0, { duration: time / 2, easing });

    $: minX = Math.min(...data.map(d => d.low));
    $: maxX = Math.max(...data.map(d => d.high));
    $: xScaleMin = left + vizOffset;
    $: xScaleMax = width - right - vizOffset;
    // x is fixed for width has no meaning for categorical
    $: X = (num: number) => {
        return xScaleMin + (num * (xScaleMax - xScaleMin)) / data.length;
    };
    $: XInvert = (x: number) => {
        return Math.floor(
            (x - xScaleMin) / ((xScaleMax - xScaleMin) / data.length)
        );
    };

    $: yVals = data.map(d => d.count);
    $: maxY = Math.max(...yVals);
    $: Y = scaleLinear()
        .domain([0, maxY])
        .range([height - buffer - bottom, top + buffer]);

    $: tw.set(1);

    let tooltipIdx: number = undefined;
    $: tooltipValue = data[tooltipIdx];

    function handleMousemove(event: MouseEvent) {
        xScaleMax - xScaleMin;
        let nearestIdx = XInvert(event.offsetX);
        let np = data[nearestIdx];
        // only show tooltip when mouse x and y are in plot area
        if (
            np &&
            event.offsetX > xScaleMin &&
            event.offsetX < xScaleMax &&
            event.offsetY < Y(0)
        ) {
            tooltipIdx = nearestIdx;
        } else {
            tooltipIdx = undefined;
        }
    }

    function clearMouseMove() {
        tooltipIdx = undefined;
    }

    console.log(xLabel);
    console.log(yLabel);
</script>

<style>
    .layercake-chart{
        width: 95%;
        height: 217px;
    }
</style>

<div class="layercake-chart fill-red-300">
<LayerCake
padding={{ top: 5, bottom: 40, left: 40, right: 10}}
x = {'count'}
y = {'value'}
yScale = {scaleBand().paddingInner([0.05])}
yDomain={data.map(d => d.value)}
xDomain={[0,null]}
data = {data}
>
<Svg>
    <Bar fill={"#fca5a5"}/>
    <AxisX xLabel={xLabel}/>
    <AxisY yLabel={yLabel} gridlines={false}/>
  </Svg>
</LayerCake>
</div>


<!-- <div>
    Aggregation Type:
    <select
        class="rounded border border-6 bg-gray-100 hover:border-gray-300"
    >
        <option value={"count"}>count</option>
    </select>
    <svg
        {width}
        {height}
        on:mousemove={handleMousemove}
        on:mouseleave={clearMouseMove}
    > -->
        <!-- histogram -->
        <!-- <g shape-rendering="crispEdges">
            {#if showTooltip && !_.isUndefined(tooltipValue)}
                <rect
                    x={X(tooltipValue.bucket) + separateQuantity}
                    width={X(tooltipValue.bucket + 1) -
                        X(tooltipValue.bucket) -
                        separateQuantity * 2}
                    y={Y(maxY)}
                    height={height - 2 * buffer - bottom - top}
                    class={'fill-gray-100'}
                />
            {/if}
            {#each data as { count, bucket }, i}
                {@const x = X(bucket) + separateQuantity}
                {@const width =
                    X(bucket + 1) - X(bucket) - separateQuantity * 2}
                {@const y = Y(0) * (1 - $tw) + Y(count) * $tw}
                {@const height = Math.min(Y(0), Y(0) * $tw - Y(count) * $tw)}

                <rect
                    {x}
                    {width}
                    {y}
                    {height}
                    class={tooltipIdx === bucket ? hoverColor : fillColor}
                />
            {/each}
            <line
                x1={left + vizOffset}
                x2={width * $tw - right - vizOffset}
                y1={Y(0) + buffer}
                y2={Y(0) + buffer}
                class={baselineStrokeColor}
            />
        </g>
        {#if showTooltip}
            <BiHistogramTooltip
                textX={left + vizOffset}
                textY={20}
                data={tooltipValue}
            />
        {/if}
        <slot x={X} y={Y} {buffer} />
    </svg>
</div> -->
