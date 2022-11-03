<script lang="ts">
    import BaseStyle from './BaseStyle.svelte';

    import { onDestroy, setContext } from 'svelte';
    import _ from 'lodash';
    import type { ProfileModel } from '../dataAPI/ProfileModel';
    import type { IColumnProfileMap } from '../common/exchangeInterfaces';

    import DFProfile from './DFProfile.svelte';
    import Parquet from './icons/Parquet.svelte';
    import { Circle } from 'svelte-loading-spinners';

    export let profileModel: ProfileModel;
    setContext('autoprofiler:profileModel', profileModel);

    let isReady: boolean;
    let isLoading: boolean;
    let columnProfiles: IColumnProfileMap;
    let name: string;
    let varsInCurrentCell: string[];

    const readyUnsub = profileModel.ready.subscribe(val => {
        isReady = val;
    });

    const loadingUnsub = profileModel.loading.subscribe(val => {
        isLoading = val;
    });

    const cpUnsub = profileModel.columnProfiles.subscribe(val => {
        columnProfiles = val;
    });

    const nameUnsub = profileModel.name.subscribe(val => {
        name = val;
    });

    const varsInCellUnsub = profileModel.variablesInCurrentCell.subscribe(
        val => {
            varsInCurrentCell = val;
        }
    );

    onDestroy(() => {
        readyUnsub();
        loadingUnsub();
        cpUnsub();
        nameUnsub();
        varsInCellUnsub();
    });
</script>

<BaseStyle />

<main class="p-4 m-0">
    <h1 class="text-lg">AutoProfiler</h1>

    {#if isReady}
        <!-- <div id="header-icon" /> -->
        <div class="mb-4">
            <p>
                Connected to:
                <span class="font-semibold">
                    {name}
                </span>
            </p>
        </div>

        {#if !_.isEmpty(columnProfiles)}
            <div>
                <div class="inline-block align-middle">
                    <Parquet size="16px" />
                </div>
                <h2 class="inline-block align-middle">DataFrames</h2>
                {#if isLoading}
                    <div class="inline-block align-middle pl-2">
                        <Circle
                            size="1"
                            color="#FF3E00"
                            unit="rem"
                            duration="1s"
                        />
                    </div>
                {/if}
            </div>

            <div>
                {#each Object.keys(columnProfiles) as dfName}
                    <DFProfile
                        {dfName}
                        isInFocus={varsInCurrentCell.includes(dfName)}
                        dataframeProfile={columnProfiles[dfName]}
                    />
                {/each}
            </div>
        {:else}
            <p class="mb-4">
                All Pandas dataframes in your jupyter notebook will be profiled
                below.
            </p>
            <p class="italic">No DataFrames detected yet!</p>
        {/if}
    {:else}
        <p>No notebook connection or executions yet.</p>
    {/if}
</main>
