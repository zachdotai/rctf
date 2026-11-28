<script lang="ts">
  import type { Snippet } from 'svelte'
  import type {
    HTMLAnchorAttributes,
    HTMLButtonAttributes,
  } from 'svelte/elements'

  type BaseProps = {
    variant?:
      | 'default'
      | 'destructive'
      | 'outline'
      | 'secondary'
      | 'ghost'
      | 'link'
    size?: 'default' | 'sm' | 'lg' | 'icon' | 'icon-sm' | 'icon-lg'
    disabled?: boolean
    children?: Snippet
  }

  type AnchorProps = Omit<HTMLAnchorAttributes, 'href'> & { href: string }
  type ButtonProps = HTMLButtonAttributes & { href?: undefined }

  type Props = BaseProps & (AnchorProps | ButtonProps)

  let {
    variant = 'default',
    size = 'default',
    disabled,
    children,
    ...rest
  }: Props = $props()
</script>

{#if rest.href !== undefined}
  {@const { href, ...anchorRest } = rest as AnchorProps}
  <a
    data-variant={variant}
    data-size={size}
    href={disabled ? undefined : href}
    aria-disabled={disabled || undefined}
    role={disabled ? 'link' : undefined}
    tabindex={disabled ? -1 : undefined}
    {...anchorRest}
  >
    {@render children?.()}
  </a>
{:else}
  {@const { type = 'button', ...buttonRest } = rest as Omit<
    ButtonProps,
    'href'
  >}
  <button
    data-variant={variant}
    data-size={size}
    {type}
    {disabled}
    {...buttonRest}
  >
    {@render children?.()}
  </button>
{/if}

<style>
  a,
  button {
    display: inline-flex;
    flex-shrink: 0;
    align-items: center;
    justify-content: center;
    gap: var(--space-2xs);
    block-size: 2.5rem;
    padding-inline: 1.25rem;
    font-size: 0.875rem;
    letter-spacing: 0.02em;
    color: var(--foreground-l0);
    text-decoration: none;
    white-space: nowrap;
    cursor: pointer;
    border: 1px solid transparent;
    border-radius: var(--radius-full);
    transition:
      background-color 120ms ease,
      border-color 120ms ease;

    :global(svg) {
      inline-size: 1em;
      block-size: 1em;
      flex-shrink: 0;
      pointer-events: none;
    }

    /* Solid action: ink pill with an italic serif label. */
    &[data-variant='default'] {
      color: var(--ink-contrast);
      background: var(--ink);
      font-family: var(--font-display);
      font-size: 0.9375rem;
      font-style: italic;
      font-weight: 400;
      letter-spacing: 0.01em;

      &:hover {
        background: var(--ink-hover);
      }
    }

    &[data-variant='destructive'] {
      color: var(--foreground-destructive);
      border-color: color-mix(
        in srgb,
        var(--foreground-destructive) 40%,
        transparent
      );

      &:hover {
        background: var(--background-destructive);
      }
    }

    &[data-variant='outline'] {
      border-color: var(--border-strong);

      &:hover {
        border-color: var(--foreground-l3);
      }
    }

    &[data-variant='secondary'] {
      background: var(--background-l2);

      &:hover {
        background: var(--background-l3);
      }
    }

    &[data-variant='ghost']:hover {
      background: var(--background-l2);
    }

    &[data-variant='link'] {
      padding-inline: 0;
      text-decoration: underline;
      text-decoration-color: var(--border-strong);
      text-underline-offset: 4px;

      &:hover {
        text-decoration-color: currentColor;
      }
    }

    &[data-size='sm'] {
      block-size: 2rem;
      padding-inline: 0.875rem;
      font-size: 0.8125rem;
    }

    &[data-size='lg'] {
      block-size: 3rem;
      padding-inline: 1.75rem;
      font-size: 1rem;
    }

    &[data-size='icon'] {
      inline-size: 2.5rem;
      padding-inline: 0;
    }

    &[data-size='icon-sm'] {
      inline-size: 2rem;
      block-size: 2rem;
      padding-inline: 0;
    }

    &[data-size='icon-lg'] {
      inline-size: 3rem;
      block-size: 3rem;
      padding-inline: 0;
    }

    &:disabled,
    &[aria-disabled='true'] {
      pointer-events: none;
      opacity: 0.4;
    }

    &:focus-visible {
      outline: 2px solid var(--ring);
      outline-offset: 2px;
      border-radius: var(--radius-full);
    }
  }
</style>
