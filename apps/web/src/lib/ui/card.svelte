<script lang="ts">
  import type { Snippet } from 'svelte'

  type Props = {
    title?: string
    description?: string
    children: Snippet
  }

  let { title, description, children }: Props = $props()
</script>

<ui-card>
  {#if title || description}
    <card-header>
      {#if title}
        <card-title>{title}</card-title>
      {/if}
      {#if description}
        <card-description>{description}</card-description>
      {/if}
    </card-header>
  {/if}
  {@render children()}
</ui-card>

<style>
  ui-card {
    display: flex;
    flex-direction: column;
    gap: var(--space-s);
    padding: var(--space-m);
    background: var(--background-raised);
    border: 1px solid var(--border-strong);
    border-radius: var(--radius-xl);
  }

  card-header {
    display: flex;
    flex-direction: column;
    gap: var(--space-3xs);
  }

  card-title {
    display: block;
    font-family: var(--font-display);
    font-size: var(--step-3);
    font-weight: 400;
    line-height: 1.15;
    letter-spacing: -0.01em;
  }

  card-description {
    display: block;
    font-size: 0.875rem;
    color: var(--foreground-l3);
  }
</style>
