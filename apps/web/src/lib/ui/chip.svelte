<script lang="ts">
  import type { Snippet } from 'svelte'

  type Props = {
    children: Snippet
  }

  let { children }: Props = $props()
</script>

<ui-chip>{@render children()}</ui-chip>

<style>
  ui-chip {
    display: inline-flex;
    flex-shrink: 0;
    align-items: center;
    gap: var(--space-3xs);
    padding: 0.1875rem 0.5rem;
    font-size: 0.8125rem;
    font-weight: 400;
    line-height: 1.3;
    color: var(--foreground-l1);
    white-space: nowrap;
    background: var(--background-l3);
    border-radius: var(--radius-sm);
  }
</style>
