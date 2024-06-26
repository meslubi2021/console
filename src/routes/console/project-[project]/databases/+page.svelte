<script lang="ts">
    import { goto } from '$app/navigation';
    import { base } from '$app/paths';
    import { page } from '$app/stores';
    import { Empty, PaginationWithLimit } from '$lib/components';
    import { Button } from '$lib/elements/forms';
    import { Container, GridHeader } from '$lib/layout';
    import type { Models } from '@appwrite.io/console';

    import type { PageData } from './$types';
    import Create from './create.svelte';
    import Grid from './grid.svelte';
    import { columns } from './store';
    import Table from './table.svelte';
    import { registerCommands } from '$lib/commandCenter';

    export let data: PageData;

    let showCreate = false;
    const project = $page.params.project;

    async function handleCreate(event: CustomEvent<Models.Database>) {
        showCreate = false;
        await goto(`${base}/console/project-${project}/databases/database-${event.detail.$id}`);
    }

    $: $registerCommands([
        {
            label: 'Create database',
            callback: () => {
                showCreate = true;
            },
            keys: ['c'],
            disabled: showCreate,
            icon: 'plus',
            group: 'databases',
            rank: 10
        }
    ]);
</script>

<Container>
    <GridHeader
        title="Databases"
        {columns}
        view={data.view}
        hideColumns={!data.databases.total}
        hideView={!data.databases.total}>
        <Button on:click={() => (showCreate = true)} event="create_database">
            <span class="icon-plus" aria-hidden="true" />
            <span class="text">Create database</span>
        </Button>
    </GridHeader>

    {#if data.databases.total}
        {#if data.view === 'grid'}
            <Grid {data} bind:showCreate />
        {:else}
            <Table {data} />
        {/if}

        <PaginationWithLimit
            name="Databases"
            limit={data.limit}
            offset={data.offset}
            total={data.databases.total} />
    {:else}
        <Empty
            single
            href="https://appwrite.io/docs/databases"
            target="database"
            on:click={() => (showCreate = true)} />
    {/if}
</Container>

<Create bind:showCreate on:created={handleCreate} />
