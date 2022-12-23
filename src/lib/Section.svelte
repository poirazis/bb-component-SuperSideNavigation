<script>
    export let collapsible = false
    export let collapsed = false
    export let value
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<li class="spectrum-SideNav-item">
    <h2 on:click={() => collapsed = collapsible ? !collapsed : false } class:collapsible={collapsible} class="spectrum-SideNav-heading">{value}</h2>
    {#if $$slots.default && !collapsed}
        <ul class="spectrum-SideNav" aria-labelledby="nav-heading-section-{value}">
            <slot />
        </ul>
    {/if}
</li>

<style>
    .collapsible:hover {
        cursor: pointer;
        color: var(--primaryColor);
    }
</style>