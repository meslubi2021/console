<script lang="ts">
    import { toLocaleDateTime } from '$lib/helpers/date';
    import { humanFileSize } from '$lib/helpers/sizeConvertion';
    import { log } from '$lib/stores/logs';
    import { Status, Tab, Tabs } from '../components';
    import { Button } from '$lib/elements/forms';
    import { base } from '$app/paths';
    import { app } from '$lib/stores/app';
    import { sdkForConsole } from '$lib/stores/sdk';
    import { page } from '$app/stores';
    import { calculateTime } from '$lib/helpers/timeConversion';
    import type { Models } from '@aw-labs/appwrite-console';

    let selectedTab: string;
    let rawData: string;

    function isDeployment(data: Models.Deployment | Models.Execution): data is Models.Deployment {
        if ('buildId' in data) {
            selectedTab = 'logs';
            rawData = `${sdkForConsole.client.config.endpoint}/functions/${$log.func.$id}/deployment/${$log.data.$id}?mode=admin&project=${$page.params.project}`;
            return true;
        }
    }

    function isExecution(data: Models.Deployment | Models.Execution): data is Models.Execution {
        if ('trigger' in data) {
            selectedTab = 'response';
            rawData = `${sdkForConsole.client.config.endpoint}/functions/${$log.func.$id}/execution/${$log.data.$id}?mode=admin&project=${$page.params.project}`;
            return true;
        }
    }

    function scrollToTop() {
        document
            .getElementsByClassName('code-panel-content')[0]
            .scrollTo({ top: 0, left: 0, behavior: 'smooth' });
    }

    function handleKeydown(event: KeyboardEvent) {
        if (event.key === 'Escape') {
            event.preventDefault();
            $log.show = false;
        }
    }
</script>

<svelte:window on:keydown={handleKeydown} />

{#if $log.data}
    <section class="cover-frame">
        <header class="cover-frame-header u-flex u-gap-16 u-main-space-between u-cross-center">
            <h1 class="body-text-1" name="title">Function ID: {$log.func.$id}</h1>
            <button on:click={() => ($log.show = false)} class="x-button" aria-label="close popup">
                <span class="icon-x" aria-hidden="true" />
            </button>
        </header>
        {#if isDeployment($log.data)}
            {@const size = humanFileSize($log.data.size)}
            <div class="cover-frame-content u-flex u-flex-vertical">
                <div class="u-flex u-gap-16">
                    <div class="avatar is-size-large">
                        <img
                            height="28"
                            width="28"
                            src={`${base}/icons/${$app.themeInUse}/color/${
                                $log.func.runtime.split('-')[0]
                            }.svg`}
                            alt="technology" />
                    </div>
                    <div>
                        <h2 class="body-text-2">Deployment ID: {$log.data.$id}</h2>
                        <time class="u-block"
                            >Created at: {toLocaleDateTime($log.data.$createdAt)}</time>
                        <div>Size: {size.value} {size.unit}</div>
                    </div>
                    <div class="status u-margin-inline-start-auto">
                        <Status status={$log.data.status}>{$log.data.status}</Status>

                        <time>TBI</time>
                    </div>
                </div>
                <div class="tabs u-margin-block-start-48 u-sep-block-end">
                    <Tabs>
                        <Tab
                            selected={selectedTab === 'logs'}
                            on:click={() => (selectedTab = 'logs')}>
                            Logs
                        </Tab>
                        <Tab
                            selected={selectedTab === 'errors'}
                            on:click={() => (selectedTab = 'errors')}>
                            Errors
                        </Tab>
                    </Tabs>
                </div>
                <div class="theme-dark u-stretch u-margin-block-start-32 u-overflow-hidden">
                    <section class="code-panel">
                        <header class="code-panel-header">
                            <div class="u-flex u-gap-16 u-margin-inline-start-auto">
                                <Button text external href={rawData}>
                                    <span class="icon-external-link" aria-hidden="true" />
                                    <span class="text">Raw data</span>
                                </Button>
                                <Button secondary on:click={scrollToTop}>
                                    <span class="text">Scroll to top</span>
                                </Button>
                            </div>
                        </header>
                        {#if selectedTab === 'logs'}
                            <code class="code-panel-content">
                                {$log.data.buildStdout ?? 'No logs recorded'}
                            </code>
                        {:else}
                            <code class="code-panel-content">
                                {$log.data.buildStderr ?? 'No errors recorded'}
                            </code>
                        {/if}
                    </section>
                </div>
            </div>
        {:else if isExecution($log.data)}
            <div class="cover-frame-content u-flex u-flex-vertical">
                <div class="u-flex u-gap-16">
                    <div class="avatar is-size-large">
                        <img
                            height="28"
                            width="28"
                            src={`${base}/icons/${$app.themeInUse}/color/${
                                $log.func.runtime.split('-')[0]
                            }.svg`}
                            alt="technology" />
                    </div>
                    <div>
                        <h2 class="body-text-2">Execution ID: {$log.data.$id}</h2>
                        <time class="u-block">
                            Created at: {toLocaleDateTime($log.data.$createdAt)}</time>
                    </div>
                    <div>
                        <p>Triggered by: {$log.data.trigger}</p>
                        <p>Type: TBI</p>
                    </div>
                    <div class="status u-margin-inline-start-auto">
                        <Status status={$log.data.status}>{$log.data.status}</Status>

                        <time>{calculateTime($log.data.duration)}</time>
                    </div>
                </div>
                <div class="tabs u-margin-block-start-48 u-sep-block-end">
                    <Tabs>
                        <Tab
                            selected={selectedTab === 'response'}
                            on:click={() => (selectedTab = 'response')}>
                            Response
                        </Tab>
                        <Tab
                            selected={selectedTab === 'errors'}
                            on:click={() => (selectedTab = 'errors')}>
                            Errors
                        </Tab>
                        <Tab
                            selected={selectedTab === 'logs'}
                            on:click={() => (selectedTab = 'logs')}>
                            Logs
                        </Tab>
                    </Tabs>
                </div>
                <div class="theme-dark u-stretch u-margin-block-start-32 u-overflow-hidden">
                    <section class="code-panel">
                        <header class="code-panel-header">
                            <div class="u-flex u-gap-16 u-margin-inline-start-auto">
                                <Button text external href={rawData}>
                                    <span class="icon-external-link" aria-hidden="true" />
                                    <span class="text">Raw data</span>
                                </Button>
                                <Button secondary on:click={scrollToTop}>
                                    <span class="text">Scroll to top</span>
                                </Button>
                            </div>
                        </header>
                        {#if selectedTab === 'logs'}
                            <code class="code-panel-content">
                                {$log.data.stdout ?? 'No logs recorded'}
                            </code>
                        {:else if selectedTab === 'errors'}
                            <code class="code-panel-content">
                                {$log.data.stderr ?? 'No errors recorded'}
                            </code>
                        {:else}
                            <code class="code-panel-content">
                                {$log.data.response ?? 'No response recorded'}
                            </code>
                        {/if}
                    </section>
                </div>
            </div>
        {/if}
    </section>
{/if}