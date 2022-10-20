<script lang="ts">
    import DFProfile from './DFProfile.svelte';
    import CompareView from './CompareView.svelte';
    import type {
        IColumnProfileWrapper,
        IColumnProfileMap
    } from '../common/exchangeInterfaces';

    // DFProfile props
    export let dfName: string;
    export let isInFocus: boolean;
    export let dataframeProfile: IColumnProfileWrapper;

    // compare props
    export let columnProfiles: IColumnProfileMap;
</script>

<div class="flex space-between gap-2 justify-between w-full">
    <div class="w-1/2">
        <DFProfile {dfName} {isInFocus} {dataframeProfile} />
    </div>
    <div class="w-1/2">
        <CompareView {columnProfiles} />
    </div>
</div>
