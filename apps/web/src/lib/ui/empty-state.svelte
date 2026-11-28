<script lang="ts">
  import type { Component } from 'svelte'

  type Props = {
    icon?: Component
    title: string
    subtitle?: string
  }

  let { icon: Icon, title, subtitle }: Props = $props()
</script>

<empty-state>
  {#if Icon}
    <Icon />
  {/if}
  <span data-part="title">{title}</span>
  {#if subtitle}
    <span data-part="subtitle">{subtitle}</span>
  {/if}
</empty-state>

<style>
  empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 0.375rem;
    max-inline-size: 24rem;
    margin-inline: auto;
    padding-block: var(--space-2xl);
    text-align: center;

    :global(svg) {
      display: none;
    }
  }

  [data-part='title'] {
    font-family: var(--font-display);
    font-size: var(--step-2);
    font-style: italic;
    color: var(--foreground-l2);
  }

  [data-part='subtitle'] {
    font-size: 0.875rem;
    color: var(--foreground-l4);
  }
</style>
