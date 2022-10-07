<script lang="ts">
    import { base } from '$app/paths';
    import { GridItem1, Empty, Pagination, AvatarGroup, CardContainer } from '$lib/components';
    import { Button } from '$lib/elements/forms';
    import { Container } from '$lib/layout';
    import CreateOrganization from '../../_createOrganization.svelte';
    import { organizationList } from '$lib/stores/organization';
    import { onMount } from 'svelte';
    import { sdkForConsole } from '$lib/stores/sdk';
    import { cardLimit } from '$lib/stores/layout';

    onMount(async () => {
        await organizationList.load();
    });

    const getAvatar = (name: string) => {
        return sdkForConsole.avatars.getInitials(name, 80, 80).toString();
    };

    const getMemberships = async (teamId: string) => {
        const memberships = await sdkForConsole.teams.listMemberships(teamId);
        return memberships.memberships.map((team) => {
            return {
                name: team.userName,
                img: getAvatar(team.userName)
            };
        });
    };

    let addOrganization = false;
    let offset = 0;
</script>

<Container>
    <div class="u-flex u-gap-12 common-section u-main-space-between">
        <h2 class="heading-level-5">Organizations</h2>

        <Button
            on:click={() => {
                addOrganization = true;
            }}>
            <span class="icon-plus" aria-hidden="true" />
            <span class="text">Create organization</span>
        </Button>
    </div>

    {#if $organizationList?.teams?.length}
        <CardContainer
            total={$organizationList.total}
            {offset}
            on:click={() => (addOrganization = true)}>
            {#each $organizationList.teams as organization, index}
                {@const avatarList = getMemberships(organization.$id)}
                {#if index >= offset && index < $cardLimit + offset}
                    <GridItem1 href={`${base}/console/organization-${organization.$id}`}>
                        <svelte:fragment slot="eyebrow"
                            >{organization?.total ? organization?.total : 'No'} projects</svelte:fragment>
                        <svelte:fragment slot="title">
                            {organization.name}
                        </svelte:fragment>
                        {#await avatarList}
                            <span class="avatar  is-color-empty" />
                        {:then avatars}
                            <AvatarGroup size={40} {avatars} />
                        {/await}
                    </GridItem1>
                {/if}
            {/each}
            <svelte:fragment slot="empty">
                <p>Create a new organization</p>
            </svelte:fragment>
        </CardContainer>

        <div class="u-flex u-margin-block-start-32 u-main-space-between">
            <p class="text">Total results: {$organizationList?.total}</p>
            <Pagination limit={$cardLimit} bind:offset sum={$organizationList?.total} />
        </div>
    {:else}
        <Empty isButton single on:click={() => (addOrganization = true)}>
            <p>Create a new organization</p>
        </Empty>
        <div class="u-flex u-margin-block-start-32 u-main-space-between">
            <p class="text">Total results: {$organizationList?.total}</p>
            <Pagination limit={$cardLimit} bind:offset sum={$organizationList?.total} />
        </div>
    {/if}
</Container>

<CreateOrganization bind:show={addOrganization} />