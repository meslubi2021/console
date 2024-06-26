<script lang="ts">
    import { page } from '$app/stores';
    import { Button, Form, FormList, InputTextarea } from '$lib/elements/forms';
    import ResetSms from './resetSms.svelte';
    import { baseSmsTemplate, smsTemplate } from './store';
    import { addNotification } from '$lib/stores/notifications';
    import { sdk } from '$lib/stores/sdk';
    import deepEqual from 'deep-equal';
    import { Submit, trackError, trackEvent } from '$lib/actions/analytics';
    import { Box } from '$lib/components';

    export let loading = false;
    const projectId = $page.params.project;
    let openResetModal = false;

    let eventType = Submit.SmsUpdateInviteTemplate;

    async function saveSmsTemplate() {
        if (!$smsTemplate.locale) {
            addNotification({
                type: 'error',
                message: 'Locale is required'
            });
            return;
        }
        try {
            switch ($smsTemplate.type) {
                case 'invitation':
                    eventType = Submit.SmsUpdateInviteTemplate;
                    break;
                case 'login':
                    eventType = Submit.SmsUpdateLoginTemplate;
                    break;
                case 'verification':
                    eventType = Submit.SmsUpdateVerificationTemplate;
                    break;
            }
            await sdk.forConsole.projects.updateSmsTemplate(
                projectId,
                $smsTemplate.type,
                $smsTemplate.locale,
                $smsTemplate.message
            );

            $baseSmsTemplate = {
                ...$smsTemplate
            };
            addNotification({
                type: 'success',
                message: `SMS ${$smsTemplate.type} template for ${$smsTemplate.locale} updated`
            });
            trackEvent(eventType, {
                locale: $smsTemplate.locale
            });
        } catch (e) {
            trackError(e, eventType);
            addNotification({
                type: 'error',
                message: e.message
            });
        }
    }

    $: isButtonDisabled = deepEqual($smsTemplate, $baseSmsTemplate);
</script>

<Box class="u-position-relative">
    {#if loading}
        <div
            class="u-position-absolute u-width-full-line u-flex u-flex-vertical u-main-center u-cross-center u-gap-16 u-margin-block-start-32"
            style="inset-inline-start: 0;">
            <div class="loader" />
            <p class="text">Loading template...</p>
        </div>
    {/if}
    <div class:u-opacity-0={loading} style={loading ? 'pointer-events: none' : ''}>
        <Form onSubmit={saveSmsTemplate}>
            <FormList gap={8}>
                <InputTextarea
                    bind:value={$smsTemplate.message}
                    id="message"
                    label="Message"
                    placeholder="Enter your message" />
            </FormList>

            <div class="u-sep-block-start u-margin-block-start-24" />

            <div class="u-flex u-gap-16 u-main-end u-margin-block-start-24">
                <Button on:click={() => (openResetModal = true)} text>Reset changes</Button>
                <Button submit disabled={isButtonDisabled}>Update</Button>
            </div>
        </Form>
    </div>
</Box>

<ResetSms bind:show={openResetModal} />
