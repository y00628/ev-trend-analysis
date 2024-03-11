<script>
    import Disasters from './Disasters.svelte';
    import Carbon from './Carbon.svelte';
    let currentPage = 'Disasters';
</script>

<button on:click={() => currentPage = 'Disasters'}>Number of Disasters By State By Year, 1960-2023</button>
<button on:click={() => currentPage = 'Carbon'}>Carbon Usage By State By Year, 1970-2021</button>

{#if currentPage === 'Disasters'}
    <Disasters />
{:else if currentPage === 'Carbon'}
    <Carbon />
{/if}
