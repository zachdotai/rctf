<script lang="ts">
  import type { HTMLInputAttributes } from 'svelte/elements'

  type Props = HTMLInputAttributes

  let { value = $bindable(), ...rest }: Props = $props()
</script>

<input bind:value {...rest} />

<style>
  input {
    display: block;
    inline-size: 100%;
    block-size: 2.5rem;
    padding-inline: 0.75rem;
    color: var(--foreground-l0);
    background: var(--background-input);
    border: 1px solid var(--border-strong);
    border-radius: var(--radius-md);
    transition: border-color 120ms ease;

    &::placeholder {
      color: var(--foreground-l4);
    }

    &:hover {
      border-color: var(--foreground-l4);
    }

    &:focus-visible {
      outline: 2px solid transparent;
      outline-offset: -1px;
      border-color: var(--foreground-l0);
      border-radius: var(--radius-md);
      box-shadow: 0 0 0 1px var(--foreground-l0);
    }

    &[aria-invalid='true'] {
      border-color: var(--foreground-destructive);
    }
  }
</style>
