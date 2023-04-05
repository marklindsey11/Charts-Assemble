<script lang="ts">
    import CollapsibleCard from './nav/CollapsibleCard.svelte';
    import { createEventDispatcher, getContext, setContext } from 'svelte';
    import ColumnProfile from './ColumnProfile.svelte';
    import ExpanderButton from './nav/ExpanderButton.svelte';
    import type {
        ColumnProfileData,
        IDFProfileWState
    } from '../common/exchangeInterfaces';
    import type { ProfileModel } from '../dataAPI/ProfileModel';
    import _, { countBy } from 'lodash';
    import Pin from './icons/Pin.svelte';
    import BivariateButton from './icons/BivariateButton.svelte';
    import Tooltip from './tooltip/Tooltip.svelte';
    import TooltipContent from './tooltip/TooltipContent.svelte';
    import { formatInteger } from './utils/formatters';
    import { NUMERIC_TOKENS } from './data-types/pandas-data-types';
    import BivariateChart from './viz/bivariate/BivariateChart.svelte';
    import DropdownMenu from './viz/bivariate/DropdownMenu.svelte';
    import Column from './viz/bivariate/Column.svelte';
    import AddVariable from './icons/AddVariable.svelte';
    import Done from './icons/Done.svelte';

    export let dfName: string;
    export let dataframeProfile: IDFProfileWState;
    export let isInFocus = false;
    export let isPinned = false;
    export let clickBivariateButton = false;

    const profileModel: ProfileModel = getContext('autoprofiler:profileModel');

    $: setContext('variableFilterCondition', {});

    // locals
    $: warningMessage = _.isEmpty(dataframeProfile.warnings)
        ? ''
        : dataframeProfile.warnings.map(w => w.warnMsg).join(', ');

    // view variables
    let profileWidth: number;
    let expanded = false;
    let headerHover = false;

    // Bivariate variables
    let showAddButton = false;
    let showBivariateMenu = false;
    let biDataPromise = undefined;
    let biDataStorage = [];
    let xSelected = false;
    let ySelected = false;
    let xVariable: ColumnProfileData;
    let xVariables: ColumnProfileData[] = [];
    let yVariable: ColumnProfileData;
    let yVariables: ColumnProfileData[] = [];

    //Dropdown Menu
    $: xOptionColumns = dataframeProfile?.profile;
    $: yOptionColumns = dataframeProfile?.profile;

    // view
    let wrapperDivWidth: number;

    // dispatches
    const dispatch = createEventDispatcher();

    function handlePin() {
        dispatch('message', {
            dfName
        });
    }

    function handleHeaderHover(event) {
        headerHover = event?.detail?.over;
    }

    function handleBivariateButton(event) {
        showAddButton = !showAddButton;
        showBivariateMenu = !showBivariateMenu;
        clickBivariateButton = !clickBivariateButton;
    }

    function handleAggrType(event, i: number) {
        let timeOffsets = biDataStorage.map(d => d.timeOffset);
        let aggrType = event?.detail?.typ;
        fetchBivariateData(
            dfName,
            xVariables[i].column,
            yVariables[i].column,
            aggrType,
            timeOffsets[i]
        ).then(biData => (biDataStorage[i] = biData));
    }

    function handleTimeOffset(event, i: number) {
        let aggrTypes = biDataStorage.map(d => d.aggrType);
        let timeOffset = event?.detail?.timeOffset;
        fetchBivariateData(
            dfName,
            xVariables[i].column,
            yVariables[i].column,
            aggrTypes[i],
            timeOffset
        ).then(biData => (biDataStorage[i] = biData));
    }

    function handleDelete(event, i:number){
        xVariables.splice(i,1);
        yVariables.splice(i,1);
        biDataStorage.splice(i,1);
        updateBivariate();
    }

    // // AddButton version
    // function handleVariable(event) {
    //     variables.push({
    //         colName: event?.detail?.colName,
    //         colType: event?.detail?.colType
    //     });
    //     if (variables.length == 2) {
    //         biDataPromise = fetchBivariateData(dfName, variables);
    //     }
    // }

    function handleBivariate(event, variable: string) {
        if (variable === 'x') {
            xVariable = event?.detail; // in the form of {"column":ColumnProfileData,"timestep":string}
        } else if (variable === 'y') {
            yVariable = event?.detail;
        };
    }

    function sendToBivariate() {
        if (xSelected === true && ySelected === true) {
            // avoid generating charts of same x and y variables
            let duplicated = false;
            for (let i = 0; i < xVariables.length; i++) {
                if (
                    xVariable.column === xVariables[i].column &&
                    yVariable.column === yVariables[i].column
                ) {
                    duplicated = true;
                }
            }
            if (!duplicated) {
                biDataPromise = fetchBivariateData(
                    dfName,
                    xVariable.column,
                    yVariable.column,
                    'count',
                    xVariable.timestep
                );
                xVariables.push(xVariable);
                yVariables.push(yVariable);

                biDataPromise.then(d => {
                    biDataStorage.push(d);
                });
            }
            xSelected = false;
            ySelected = false;
            xVariable = null;
            yVariable = null;
        }
    }


    function logAction(name: string) {
        profileModel.logger.log(name, { dfName });
    }

    let baseClasses = 'grid place-items-center rounded hover:bg-gray-100 ';

    async function fetchBivariateData(
        dfName: string,
        xVariable: ColumnProfileData,
        yVariable: ColumnProfileData,
        aggrType: string,
        timestep: string
    ) {
        let biData;
        biData = await profileModel.getBivariateData(
            dfName,
            xVariable.colName,
            xVariable.colType,
            yVariable.colName,
            yVariable.colType,
            aggrType,
            timestep
        );
        return biData;
    }

    // update bivariate chart whenever the associated data is updated
    function updateBivariate() {
        let aggrTypes = biDataStorage.map(d => d.aggrType);
        let biDataPromises = xVariables.map((_, i) => {
            return fetchBivariateData(
                dfName,
                xVariables[i].column,
                yVariables[i].column,
                aggrTypes[i],
                xVariables[i].timestep
            );
        });
        Promise.all(biDataPromises).then(biDataArr => {
            biDataStorage = biDataArr;
        });
    }

    $: {
        dataframeProfile.profile;
        updateBivariate();
    }
</script>

<div>
    <CollapsibleCard
        bind:open={expanded}
        on:header-hover={handleHeaderHover}
        on:open={() => logAction('UI.ToggleDFOpen')}
        on:close={() => logAction('UI.ToggleDFClose')}
    >
        <div slot="header" class="dfprofile-header flex gap-1 items-center">
            <ExpanderButton rotated={expanded} />

            <div class="font-bold">
                {#if dfName == profileModel.currentOutputName}
                    Output from [{profileModel.currentOutputName.slice(1)}]
                {:else}
                    {dfName}
                {/if}
            </div>

            <p class="grow">
                {formatInteger(dataframeProfile?.shape?.[0])} x {formatInteger(
                    dataframeProfile?.shape?.[1]
                )}
            </p>

            {#if isInFocus}
                <div class="focusIndicator justify-end" />
            {/if}
        </div>

        <div slot="header-no-collapse">
            <Tooltip location="right" alignment="center" distance={8}>
                <button
                    class={baseClasses +
                        (isPinned
                            ? 'text-black'
                            : headerHover
                            ? 'text-gray-400'
                            : 'text-transparent')}
                    style="width: 16px; height: 16px;"
                    on:click={handlePin}
                >
                    <Pin size="16px" />
                </button>

                <TooltipContent slot="tooltip-content">
                    {#if isPinned}
                        Unpin
                    {:else}
                        Pin
                    {/if}
                </TooltipContent>
            </Tooltip>
        </div>

        <div slot="body" class="dfprofile-body">
            <div bind:clientWidth={profileWidth} class="col-profiles">
                {#if !_.isEmpty(warningMessage)}
                    <div class="pl-2 pr-2 pb-2">
                        <span class="bg-amber-500 rounded-md p-[3px]"
                            >Warning
                        </span>
                        {warningMessage}
                    </div>
                {/if}
                {#if dataframeProfile?.shape?.[1] > 0}
                    {#each dataframeProfile?.profile as column (column.colName)}
                        <ColumnProfile
                            {dfName}
                            colName={column.colName}
                            type={column.colType}
                            summary={column.summary}
                            nullCount={column.nullCount}
                            containerWidth={profileWidth}
                            totalRows={dataframeProfile?.shape?.[0]}
                            isIndex={column.colIsIndex}
                            {showAddButton}
                        />
                    {/each}
                {:else}
                    <p class="pl-8">No columns!</p>
                {/if}
                <Tooltip location="right" alignment="center" distance={8}>
                    <button
                        class={baseClasses +
                            'pl-2 pr-2 flex space-between gap-2 justify-between w-full ' +
                            'text-gray-400'}
                        style="width: 16px; height: 16px;"
                        on:click={handleBivariateButton}
                    >
                        <BivariateButton size="16px" />
                    </button>

                    <TooltipContent slot="tooltip-content">
                        Add New Chart
                    </TooltipContent>
                </Tooltip>
                {#if !_.isUndefined(biDataPromise)}
                    <!-- svelte-ignore empty-block -->
                    {#await biDataPromise then biData}
                        {#each biDataStorage as previousBiData, idx}
                            <div
                                class="pt-1 pb-1 pl-2 pr-4 w-full"
                                bind:clientWidth={wrapperDivWidth}
                            >
                                <BivariateChart
                                    biData={previousBiData}
                                    xLabel={previousBiData.xName}
                                    xType={previousBiData.xType}
                                    yLabel={previousBiData.yName}
                                    yType={previousBiData.yType}
                                    timeOffset={previousBiData.timeOffset}
                                    on:selectAggrType={event => {
                                        handleAggrType(event, idx);
                                    }}
                                    on:selectTimeOffset={event => {
                                        handleTimeOffset(event, idx);
                                    }}
                                    on:delete={event => {
                                        handleDelete(event,idx);
                                    }}
                                />
                            </div>
                        {/each}
                        <!-- Bivariate Chart is displayed here -->
                    {/await}
                {/if}
                {#if showBivariateMenu}
                    <div
                        class="flex p-2 rounded bg-gray-50"
                        style="flex-direction:column"
                    >
                        <div style="width:100%">Adding new chart</div>
                        <div class="flex" style="flex-direction: row">
                            <DropdownMenu
                                title={''}
                                bind:selected={xSelected}
                                bind:optionColumns={xOptionColumns}
                                on:select={event => {
                                    handleBivariate(event, 'x');
                                }}
                                clickable={true}
                            />
                            <div class="grow" />
                            <DropdownMenu
                                title={''}
                                bind:selected={ySelected}
                                bind:optionColumns={yOptionColumns}
                                on:select={event => {
                                    handleBivariate(event, 'y');
                                }}
                                bind:clickable={xSelected}
                                bind:filteringColumn={xVariable}
                            />
                        </div>
                        <div class="flex">
                            <div class="grow" />
                            <button class="flex pr-2 text-gray-500 hover:bg-gray-100" on:click={sendToBivariate}>Done
                            <Done/>
                            </button>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
    </CollapsibleCard>
</div>

<style>
    .col-profiles {
        width: 100%;
    }

    .dfprofile-header {
        margin: 0;
        padding: 0.5em;
    }

    .dfprofile-body {
        display: flex;
    }

    .focusIndicator {
        height: 10px;
        width: 10px;
        background-color: #1976d2;
        border-radius: 2px;
    }

    .bivariate-menu {
        width: 50%;
        padding: 0.2em;
    }
</style>
