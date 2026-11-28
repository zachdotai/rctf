<script lang="ts">
  import { normalizeProps, useMachine } from '@zag-js/svelte'
  import * as tabs from '@zag-js/tabs'
  import type { Component, Snippet } from 'svelte'

  type Tab = {
    value: string
    label: string
    count?: number
    icon?: Component
    invalid?: boolean
  }

  type Props = {
    value?: string | null
    onValueChange?: (value: string) => void
    tabs: Tab[]
    content: Snippet<[{ value: string }]>
  }

  let {
    value = $bindable(null),
    onValueChange,
    tabs: items,
    content,
  }: Props = $props()

  const id = $props.id()
  const service = useMachine(tabs.machine, () => ({
    id,
    value,
    onValueChange(details: { value: string }) {
      value = details.value
      onValueChange?.(details.value)
    },
  }))
  const api = $derived(tabs.connect(service, normalizeProps))
</script>

<div {...api.getRootProps()}>
  <div {...api.getListProps()}>
    {#each items as tab (tab.value)}
      <button
        {...api.getTriggerProps({ value: tab.value })}
        data-invalid={tab.invalid ? '' : undefined}
      >
        {#if tab.icon}
          <tab.icon data-slot="tab-icon" />
        {/if}
        {tab.label}{#if tab.count !== undefined}<tab-count
            >({tab.count})</tab-count
          >{/if}{#if tab.invalid}<tab-invalid aria-hidden="true"
          ></tab-invalid>{/if}
      </button>
    {/each}
  </div>
  {#each items as tab (tab.value)}
    <div {...api.getContentProps({ value: tab.value })}>
      {@render content({ value: tab.value })}
    </div>
  {/each}
</div>

<style>
  [data-part='list'] {
    display: flex;
    gap: 1.75rem;
    border-block-end: 1px solid var(--border);
  }

  [data-part='trigger'] {
    display: inline-flex;
    align-items: center;
    gap: 0.375rem;
    margin-block-end: -1px;
    padding-block: 0.75rem;
    font-size: 0.875rem;
    letter-spacing: 0.02em;
    color: var(--foreground-l3);
    white-space: nowrap;
    cursor: pointer;
    border-block-end: 1px solid transparent;
    transition: color 120ms ease;

    &:hover {
      color: var(--foreground-l0);
    }

    &[data-selected] {
      color: var(--foreground-l0);
      border-block-end-color: var(--foreground-l0);
    }

    &:focus-visible {
      outline-offset: 2px;
    }

    :global([data-slot='tab-icon']) {
      display: none;
    }
  }

  tab-count {
    color: var(--foreground-l4);
    font-variant-numeric: tabular-nums;
  }

  tab-invalid {
    display: inline-block;
    inline-size: 0.375rem;
    block-size: 0.375rem;
    border-radius: var(--radius-full);
    background: var(--foreground-destructive);
  }

  [data-part='content'] {
    &:focus-visible {
      outline: none;
    }
  }
</style>
