<script lang="ts">
    import { goto } from '$app/navigation';
    import { page } from '$app/stores';
    import { user } from '$lib/stores/user';
    import { redirectTo } from '$lib/stores/organization';
    import { onMount } from 'svelte';
    import { base } from '$app/paths';
    import { browser, dev } from '$app/environment';
    import { app } from '$lib/stores/app';
    import Notifications from '$lib/layout/notifications.svelte';
    import Loading from './_loading.svelte';
    import { webVitals } from '$lib/helpers/vitals';

    let loaded = false;

    if (browser) {
        window.VERCEL_ANALYTICS_ID = import.meta.env.VERCEL_ANALYTICS_ID?.toString() ?? false;
        window.GOOGLE_ANALYTICS = import.meta.env.VITE_GOOGLE_ANALYTICS?.toString() ?? false;
    }

    const acceptedRoutes = ['/login', '/register', '/recover', '/invite'];

    onMount(async () => {
        try {
            if (!$user) {
                await user.fetchUser();
            }

            if (!$page.url.pathname.startsWith('/console')) {
                await redirectTo();
            }
        } catch (error) {
            if (acceptedRoutes.includes($page.url.pathname)) {
                await goto(`${base}${$page.url.pathname}${$page.url.search}`);
            } else await goto(`${base}/login`);
        } finally {
            loaded = true;
        }
    });

    $: if (!dev && browser && window.VERCEL_ANALYTICS_ID) {
        webVitals({
            path: $page.url.pathname,
            params: $page.params,
            analyticsId: window.VERCEL_ANALYTICS_ID
        });
    }

    $: {
        if (browser) {
            if ($app.theme === 'auto') {
                const darkThemeMq = window.matchMedia('(prefers-color-scheme: dark)');
                if (darkThemeMq.matches) {
                    document.body.setAttribute('class', `theme-dark`);
                    $app.themeInUse = 'dark';
                } else {
                    document.body.setAttribute('class', `theme-light`);
                    $app.themeInUse = 'light';
                }
            } else {
                document.body.setAttribute('class', `theme-${$app.theme}`);
                $app.themeInUse = $app.theme;
            }
        }
    }
</script>

<svelte:head>
    {#if browser && window.GOOGLE_ANALYTICS}
        <script
            async
            src={`https://www.googletagmanager.com/gtag/js?id=${window.GOOGLE_ANALYTICS}`}></script>
        <script>
            window.dataLayer = window.dataLayer || [];
            function gtag() {
                dataLayer.push(arguments);
            }
            gtag('js', new Date());
            gtag('config', window.GOOGLE_ANALYTICS);
        </script>
    {/if}
</svelte:head>

<Notifications />

{#if loaded}
    <slot />
{:else}
    <Loading />
{/if}

<style lang="scss" global>
    .tippy-box {
        --p-tooltip-text-color: var(--color-neutral-10);
        --p-tooltip--bg-color: var(--color-neutral-100);

        background-color: hsl(var(--p-tooltip--bg-color));
        color: hsl(var(--p-tooltip-text-color));
        font-size: var(--font-size-0);
        line-height: 1.5;

        &[data-placement^='top'] > .tippy-arrow::before {
            border-top-color: hsl(var(--p-tooltip--bg-color));
        }
        &[data-placement^='bottom'] > .tippy-arrow::before {
            border-bottom-color: hsl(var(--p-tooltip--bg-color));
        }
        &[data-placement^='left'] > .tippy-arrow::before {
            border-left-color: hsl(var(--p-tooltip--bg-color));
        }
        &[data-placement^='right'] > .tippy-arrow::before {
            border-right-color: hsl(var(--p-tooltip--bg-color));
        }
    }
</style>