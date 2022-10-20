<script lang="ts">
    import type {
        IColumnProfileMap,
        IColumnProfileWrapper
    } from '../common/exchangeInterfaces';
    import DFProfile from './DFProfile.svelte';

    export let columnProfiles: IColumnProfileMap;

    let selectedDFName: string;
    let compareProfile: IColumnProfileWrapper;

    $: if (selectedDFName) {
        compareProfile = columnProfiles[selectedDFName];
    }
</script>

<div class="compareView">
    <div>
        <p class="inline-block">Compare to...</p>

        <select class="inline-block" bind:value={selectedDFName}>
            <option value={undefined}> None </option>
            {#each Object.keys(columnProfiles) as dfName}
                <option value={dfName}>
                    {dfName}
                </option>
            {/each}
        </select>
    </div>

    {#if compareProfile}
        <DFProfile
            dfName={selectedDFName}
            isInFocus={false}
            dataframeProfile={compareProfile}
        />
    {/if}
</div>

<style>
    .compareView {
        border: 1px dashed black;
    }
</style>
