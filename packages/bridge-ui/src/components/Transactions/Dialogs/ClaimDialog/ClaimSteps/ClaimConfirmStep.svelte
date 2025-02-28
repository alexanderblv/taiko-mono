<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { t } from 'svelte-i18n';

  import ActionButton from '$components/Button/ActionButton.svelte';
  import { Icon, type IconType } from '$components/Icon';
  import { Spinner } from '$components/Spinner';
  import { connectedSourceChain } from '$stores/network';
  import { theme } from '$stores/theme';

  import { TWO_STEP_STATE } from '../types';

  export let canClaim = false;

  export let claimingDone = false;

  export let claiming = false;

  export let proveOrClaimStep: TWO_STEP_STATE;

  const dispatch = createEventDispatcher();

  const handleClaimClick = async () => {
    dispatch('claim');
  };

  const getSuccessTitle = () => {
    if (proveOrClaimStep === TWO_STEP_STATE.PROVE) {
      return $t('bridge.step.confirm.success.prove');
    }

    return $t('bridge.step.confirm.success.claim');
  };

  const getSuccessDescription = () => {
    if (proveOrClaimStep === TWO_STEP_STATE.PROVE) {
      return $t('bridge.step.confirm.success.prove_description');
    }

    return $t('transactions.actions.claim.success.message', { values: { network: $connectedSourceChain.name } });
  };

  $: claimOrProveActionButton =
    proveOrClaimStep === TWO_STEP_STATE.CLAIM
      ? $t('transactions.claim.steps.confirm.claim_button')
      : $t('transactions.claim.steps.confirm.prove');

  $: proceedText =
    proveOrClaimStep === TWO_STEP_STATE.CLAIM
      ? $t('transactions.claim.steps.confirm.proceed')
      : $t('transactions.claim.steps.confirm.prove');

  $: proceedDescription =
    proveOrClaimStep === TWO_STEP_STATE.CLAIM
      ? $t('transactions.claim.steps.confirm.claim_description')
      : $t('transactions.claim.steps.confirm.prove_description');

  $: bridgeIcon = `bridge-${$theme}` as IconType;
  $: successIcon = `success-${$theme}` as IconType;

  $: statusTitle = getSuccessTitle();
  $: statusDescription = getSuccessDescription();

  $: claimDisabled = !canClaim || claiming;
</script>

<div class="space-y-[18px]">
  <div class="mt-[30px]">
    <section id="txStatus">
      <div class="flex flex-col justify-content-center items-center">
        {#if claimingDone}
          <Icon type={successIcon} size={160} />
          <div id="text" class="f-col my-[30px] text-center">
            <!-- eslint-disable-next-line svelte/no-at-html-tags -->
            <h1>{@html statusTitle}</h1>
            <!-- eslint-disable-next-line svelte/no-at-html-tags -->
            <span class="">{@html statusDescription}</span>
          </div>
        {:else if claiming}
          <Spinner class="!w-[160px] !h-[160px] text-primary-brand" />
          <div id="text" class="f-col my-[30px] text-center">
            <h1 class="mb-[16px]">{$t('bridge.step.confirm.processing')}</h1>
            <span>{$t('bridge.step.confirm.approve.pending')}</span>
          </div>
        {:else if !claiming && !claimingDone}
          <Icon type={bridgeIcon} size={160} />
          <div id="text" class="f-col my-[30px] text-center">
            <h1 class="mb-[16px]">{proceedText}</h1>
            <span>{proceedDescription}</span>
          </div>
        {/if}
      </div>
    </section>
    {#if !claimingDone}
      <section id="actions" class="f-col w-full">
        <div class="h-sep mb-[30px]" />
        <ActionButton
          onPopup
          priority="primary"
          loading={claiming}
          on:click={() => handleClaimClick()}
          disabled={claimDisabled}>{claimOrProveActionButton}</ActionButton>
      </section>
    {/if}
  </div>
</div>
