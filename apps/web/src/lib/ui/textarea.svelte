<script lang="ts">
  import type { HTMLTextareaAttributes } from 'svelte/elements'

  type Props = HTMLTextareaAttributes

  let { value = $bindable(), ...rest }: Props = $props()
</script>

<textarea bind:value {...rest}></textarea>

<style>
  textarea {
    display: block;
    inline-size: 100%;
    min-block-size: 4.5rem;
    padding: 0.625rem 0.75rem;
    color: var(--foreground-l0);
    background: var(--background-input);
    border: 1px solid var(--border-strong);
    border-radius: var(--radius-md);
    resize: vertical;
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
