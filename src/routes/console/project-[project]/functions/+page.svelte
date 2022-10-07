<script lang="ts">
    import { page } from '$app/stores';
    import { sdkForProject } from '$lib/stores/sdk';
    import { Button, InputSearch } from '$lib/elements/forms';
    import { Card, Empty, Pagination, Tile, Tiles } from '$lib/components';
    import Create from './_create.svelte';
    import { Container } from '$lib/layout';
    import { base } from '$app/paths';
    import { Query } from '@aw-labs/appwrite-console';

    let search = '';
    let showCreate = false;
    let offset = 0;

    const limit = 25;
    const project = $page.params.project;

    $: request = sdkForProject.functions.list([Query.limit(0), Query.offset(0)], search);
    $: if (search) offset = 0;
</script>

<Container>
    <h1>Functions</h1>
    <Card>
        <InputSearch bind:value={search} />
    </Card>

    {#await request}
        <div aria-busy="true" />
    {:then response}
        {#if response.total}
            <p>{response.total} functions found</p>
            <Tiles>
                {#each response.functions as func}
                    <Tile
                        href={`${base}/console/project-${project}/functions/function/${func.$id}`}
                        title={func.name} />
                {/each}
            </Tiles>

            <Pagination {limit} bind:offset sum={response.total} />
        {:else if search}
            <Empty>
                <div class="u-flex u-flex-vertical">
                    No results found for <b>{search}</b>
                </div>
            </Empty>
        {:else}
            <Empty>
                <div class="u-flex u-flex-vertical">
                    <div class="common-section">No Functions Found</div>
                    <div class="common-section">
                        You haven't created any functions for your project yet.
                    </div>
                </div>
            </Empty>
        {/if}
    {/await}

    <Button on:click={() => (showCreate = true)}>Create Function</Button>
</Container>

<Create bind:showCreate />