<script lang="ts">
    import { WizardStep } from '$lib/layout';
    import Repositories from '../components/repositories.svelte';

    let hasInstallations: boolean;
    let selectedRepository: string;
    async function beforeSubmit() {
        if (!hasInstallations) {
            throw new Error('Please connect a Git provider');
        }
        if (!selectedRepository) {
            throw new Error('Please select a repository');
        }
    }
</script>

<WizardStep {beforeSubmit}>
    <svelte:fragment slot="title">Repository</svelte:fragment>
    <svelte:fragment slot="subtitle">
        Select a Git repository that will trigger your function deployments when updated.
    </svelte:fragment>
    <Repositories bind:hasInstallations bind:selectedRepository />
</WizardStep>
