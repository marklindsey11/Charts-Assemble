<script lang="ts">
    import { createEventDispatcher } from 'svelte';
    import type { ColumnProfileData } from '../../../common/exchangeInterfaces';

    const dispatch = createEventDispatcher();

    export let title: string;
    export let variable: ColumnProfileData;
    export let selected: boolean;
    export let optionColumns: ColumnProfileData[];

    $: displayColumns = optionColumns;
    let inputValue = '';
    let selectedColumn = '';

    let showMenu = false;

    let w: number, h: number;

    function handleInput() {
        if (inputValue.length == 0) {
            displayColumns = optionColumns;
        } else {
            displayColumns = optionColumns.filter(column =>
                column.colName.toLowerCase().match(inputValue.toLowerCase())
            );
        }
    }
</script>

<div class="bivariate-menu" bind:clientWidth={w} bind:clientHeight={h}>
    <span>{title}</span>
    <div
        class="dropdown-menu-container rounded border border-6 bg-gray-100 hover:border-gray-300"
        on:click={() => {
            showMenu = !showMenu;
        }}
    >
        <div class="dropdown-menu-option pl-1" style="float:left;max-width:75%">{selectedColumn}</div>
        <svg
            class="pt-1 pb-1 pl-1 pr-1"
            width={'20px'}
            height={'20px'}
            viewBox="0 0 1024 1024"
            fill="#bdbdbd"
            xmlns="http://www.w3.org/2000/svg"
            ><path
                d="M525.873548 897.156129l-383.174193-761.723871 763.045161-1.981935-379.870968 763.705806z"
            /></svg
        >
        {#if showMenu}
            <div
                class="dropdown-menu-options rounded border border-6 bg-gray-100 hover:border-gray-300"
            >
                <div>
                    <input
                        type="text"
                        placeholder="Search..."
                        bind:value={inputValue}
                        on:input={handleInput}
                        on:click={event => {
                            event.stopPropagation();
                        }}
                    />
                </div>
                {#each displayColumns as column}
                    <div
                        class="dropdown-menu-option"
                        on:click={() => {
                            selected = !selected;
                            selectedColumn = column.colName;
                            dispatch('select', column);
                        }}
                    >
                        {column.colName}
                    </div>
                {/each}
            </div>
        {/if}
    </div>

    <!-- <select
        class="bivariate-menu rounded border border-6 bg-gray-100 hover:border-gray-300"
        bind:value={variable}
        on:change={() => {
            selected = true;
            dispatch('select');
        }}
    >
        <option value={undefined}>-Select...-</option>
        {#each optionColumns as column}
            <option value={column}>{column.name}</option>
        {/each}
    </select> -->
</div>

<style>
    svg {
        float: right;
    }

    input {
        max-width: 100%;
    }

    .bivariate-menu {
        position: relative;
        display: flex;
        padding: 0.2em;
    }

    .dropdown-menu-container {
        width:50%;
        position: relative;
        display: inline-block;
        height: 20px;
        margin-left: 2px;
    }

    .dropdown-menu-options {
        position: absolute;
        display: block;
        z-index: 10;
        bottom: 20px;
        max-width: 100%;
        max-height: 200px;
        overflow: scroll;
    }

    .dropdown-menu-option {
        max-width: 100%;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }
</style>
