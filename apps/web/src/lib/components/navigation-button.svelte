<script lang="ts">
  import { page } from '$app/state'
  import { IconCaretDown } from '$lib/icons'

  type Props = {
    href?: string
    activePath?: string
    label: string
    caret?: boolean
    [key: string]: unknown
  }

  let { href, activePath, label, caret = false, ...rest }: Props = $props()

  const active = $derived.by(() => {
    if (!activePath) return false
    if (activePath === '/') return page.url.pathname === '/'
    return page.url.pathname.startsWith(activePath)
  })
</script>

{#if href}
  <a
    {href}
    aria-current={active ? 'page' : undefined}
    data-active={active ? '' : undefined}
    {...rest}
  >
    {label}
  </a>
{:else}
  <button type="button" data-active={active ? '' : undefined} {...rest}>
    {label}
    {#if caret}
      <IconCaretDown data-slot="caret" />
    {/if}
  </button>
{/if}

<style>
  a,
  button {
    --underline: transparent;

    display: inline-flex;
    align-items: center;
    gap: 0.25rem;
    padding: 0.5rem 0.25rem;
    font-size: 0.875rem;
    font-weight: 400;
    letter-spacing: 0.025em;
    color: var(--foreground-l0);
    white-space: nowrap;
    text-underline-offset: 0.45em;
    cursor: pointer;
    transition: opacity 120ms ease;

    &:hover {
      opacity: 0.6;
    }

    &[data-active] {
      --underline: currentColor;

      text-decoration: underline;
      text-decoration-thickness: 1px;
      text-decoration-color: currentColor;
    }

    :global([data-slot='caret']) {
      font-size: 0.625rem;
    }
  }
</style>
