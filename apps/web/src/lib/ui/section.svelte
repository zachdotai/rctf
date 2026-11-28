<script lang="ts">
  import type { Snippet } from 'svelte'

  type Props = {
    title: string
    flush?: boolean
    children: Snippet
  }

  let { title, flush = false, children }: Props = $props()
</script>

<ui-section>
  <section-header>{title}</section-header>
  <section-content data-flush={flush || undefined}
    >{@render children()}</section-content
  >
</ui-section>

<style>
  ui-section {
    display: block;
  }

  section-header {
    display: flex;
    gap: var(--space-2xs);
    align-items: center;
    justify-content: space-between;
    padding-block-end: 0.75rem;
    font-size: var(--eyebrow-size);
    font-weight: var(--eyebrow-weight);
    letter-spacing: var(--eyebrow-tracking);
    text-transform: uppercase;
    color: var(--foreground-l3);
  }

  section-content {
    display: block;

    &[data-flush] {
      padding: 0;
    }
  }
</style>
