<script lang="ts">
  import { normalizeProps, useMachine } from '@zag-js/svelte'
  import * as tooltip from '@zag-js/tooltip'
  import Portal from '$lib/ui/portal.svelte'
  import type { Snippet } from 'svelte'

  type Props = {
    label: string
    disabled?: boolean
    children: Snippet<[{ props: Record<string, unknown> }]>
  }

  let { label, disabled = false, children }: Props = $props()

  const id = $props.id()
  const service = useMachine(tooltip.machine, () => ({
    id,
    openDelay: 300,
    disabled,
  }))
  const api = $derived(tooltip.connect(service, normalizeProps))

  const triggerProps = $derived(
    api.getTriggerProps() as Record<string, unknown>
  )
</script>

{@render children({ props: triggerProps })}

{#if api.open}
  <Portal>
    <div {...api.getPositionerProps()}>
      <div {...api.getArrowProps()}>
        <div {...api.getArrowTipProps()}></div>
      </div>
      <div {...api.getContentProps()}>{label}</div>
    </div>
  </Portal>
{/if}

<style>
  [data-part='content'] {
    position: relative;
    z-index: var(--layer-popover);
    padding: var(--space-3xs) var(--space-2xs);
    font-size: 0.8125rem;
    letter-spacing: 0.02em;
    color: var(--foreground-l0);
    background: var(--background-raised);
    border: 1px solid var(--border-strong);
    border-radius: var(--radius-md);
    box-shadow: 0 8px 24px rgb(22 20 15 / 0.08);
  }

  [data-part='arrow'] {
    --arrow-size: 0.625rem;
    --arrow-background: var(--background-raised);
    z-index: calc(var(--layer-popover) + 1);

    &:has(+ [data-part='content'][data-side='top']) [data-part='arrow-tip'] {
      border-inline-end: 1px solid var(--border-strong);
      border-block-end: 1px solid var(--border-strong);
    }

    &:has(+ [data-part='content'][data-side='bottom']) [data-part='arrow-tip'] {
      border-inline-start: 1px solid var(--border-strong);
      border-block-start: 1px solid var(--border-strong);
    }
  }
</style>
