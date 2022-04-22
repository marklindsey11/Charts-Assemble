<script lang="ts">
    import DFProfile from './DFProfile.svelte';
    import { dataFramesAndCols } from '../stores';
    import type { ProfileModel } from '../ProfileModel';

    export let profileModel: ProfileModel;

    $: console.log(
        '[SVELTE] root Profiler dataFramesAndCols is: ',
        $dataFramesAndCols
    );
</script>

<main>
    <h1>Dataframe Profiler</h1>

    {#if $dataFramesAndCols}
        <!-- <div id="header-icon" /> -->
        <div class="notebook-info">
            <p>
                {profileModel.name} is a
                <b>
                    {#await profileModel.language}
                        ?
                    {:then lang}
                        {lang}
                    {/await}
                </b>
                notebook.
            </p>
        </div>

        <div class="df-cards">
            {#each Object.keys($dataFramesAndCols) as dfName}
                <DFProfile
                    {dfName}
                    colInfo={$dataFramesAndCols[dfName]}
                    {profileModel}
                />
            {/each}
        </div>
    {:else}
        <p>
            All Pandas dataframes in your jupyter notebook will be profiled
            below (no data yet).
        </p>
    {/if}
</main>

<style>
    main {
        /* text-align: center; */
        padding: 1em;
        /* max-width: 240px; */
        margin: 0 auto;
    }

    .notebook-info {
        margin-bottom: 1em;
    }

    /* .df-cards  {
        margin-bottom: 1em;
    } */
</style>