<script lang="ts">
    import { page } from '$app/stores';
    import { Empty } from '$lib/components';
    import { CARD_LIMIT } from '$lib/constants';
    import { preferences } from '$lib/stores/preferences';

    export let offset = 0;
    export let total = 0;
    export let event: string = null;

    $: limit = preferences.get($page.route)?.limit ?? CARD_LIMIT;
</script>

<ul
    class="grid-box common-section u-margin-block-start-32"
    style={`--grid-gap:1.5rem; --grid-item-size:${total > 3 ? '22rem' : '25rem'};`}
    data-private>
    <slot />

    {#if total > 3 ? total < limit + offset : total % 2 !== 0}
        <Empty on:click target={event}>
            <slot name="empty" />
        </Empty>
    {/if}
</ul>
