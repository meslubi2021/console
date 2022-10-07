<script lang="ts">
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';
    import { Button } from '$lib/elements/forms';
    import { Empty, Pagination, Copy, GridItem1, CardContainer } from '$lib/components';
    import { Pill } from '$lib/elements';
    import { Query, type Models } from '@aw-labs/appwrite-console';
    import Create from './_create.svelte';
    import { Container } from '$lib/layout';
    import { base } from '$app/paths';
    import { bucketList } from './store';
    import { tooltip } from '$lib/actions/tooltip';
    import { cardLimit } from '$lib/stores/layout';

    let showCreate = false;
    let offset = 0;

    const project = $page.params.project;
    const bucketCreated = async (event: CustomEvent<Models.Bucket>) => {
        showCreate = false;
        await goto(`${base}/console/project-${project}/storage/bucket/${event.detail.$id}`);
    };

    $: bucketList.load([Query.limit($cardLimit), Query.offset(offset)]);
</script>

<Container>
    <div class="u-flex u-gap-12 common-section u-main-space-between">
        <h2 class="heading-level-5">Buckets</h2>

        <Button on:click={() => (showCreate = true)}>
            <span class="icon-plus" aria-hidden="true" /> <span class="text">Create bucket</span>
        </Button>
    </div>

    {#if $bucketList?.total}
        <CardContainer total={$bucketList.total} {offset} on:click={() => (showCreate = true)}>
            {#each $bucketList.buckets as bucket}
                <GridItem1 href={`${base}/console/project-${project}/storage/bucket/${bucket.$id}`}>
                    <svelte:fragment slot="eyebrow">XX Files</svelte:fragment>
                    <svelte:fragment slot="title">{bucket.name}</svelte:fragment>
                    <svelte:fragment slot="status">
                        {#if !bucket.enabled}
                            <Pill>Disabled</Pill>
                        {/if}
                    </svelte:fragment>

                    <Copy value={bucket.$id}>
                        <Pill button><i class="icon-duplicate" />Bucket ID</Pill>
                    </Copy>

                    <svelte:fragment slot="icons">
                        <li>
                            <span
                                class:u-opacity-0-2={!bucket.encryption}
                                class="icon-lock-closed"
                                aria-hidden="true"
                                use:tooltip={{
                                    content: bucket.encryption
                                        ? 'Encryption enabled'
                                        : 'Encryption disabled'
                                }} />
                        </li>
                        <li>
                            <span
                                class:u-opacity-0-2={!bucket.antivirus}
                                class="icon-shield-check"
                                aria-hidden="true"
                                use:tooltip={{
                                    content: bucket.antivirus
                                        ? 'Antivirus enabled'
                                        : 'Antivirus disabled'
                                }} />
                        </li>
                    </svelte:fragment>
                </GridItem1>
            {/each}
            <svelte:fragment slot="empty">
                <p>Add a new bucket</p>
            </svelte:fragment>
        </CardContainer>

        <div class="u-flex u-margin-block-start-32 u-main-space-between">
            <p class="text">Total results: {$bucketList.total}</p>
            <Pagination limit={$cardLimit} bind:offset sum={$bucketList.total} />
        </div>
    {:else}
        <Empty isButton single on:click={() => (showCreate = true)}>
            <p>Add your first bucket to get started</p>
        </Empty>
    {/if}
</Container>

<Create bind:showCreate on:created={bucketCreated} />