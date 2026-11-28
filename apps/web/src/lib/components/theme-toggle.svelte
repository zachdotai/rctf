<script lang="ts">
  import { IconMoonStars, IconSun } from '$lib/icons'

  function toggle() {
    const root = document.documentElement
    const dark =
      root.dataset.theme === 'dark' ||
      (!root.dataset.theme &&
        matchMedia('(prefers-color-scheme: dark)').matches)
    const theme = dark ? 'light' : 'dark'
    root.dataset.theme = theme
    localStorage.setItem('theme', theme)
  }
</script>

<button type="button" onclick={toggle} aria-label="Toggle theme">
  <sun-icon data-theme-visible="dark"><IconSun /></sun-icon>
  <moon-icon data-theme-visible="light"><IconMoonStars /></moon-icon>
</button>

<style>
  button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    inline-size: 2.25rem;
    block-size: 2.25rem;
    font-size: 1.0625rem;
    color: var(--foreground-l3);
    border-radius: var(--radius-full);
    cursor: pointer;
    transition: color 120ms ease;

    &:hover {
      color: var(--foreground-l0);
    }

    &:focus-visible {
      border-radius: var(--radius-full);
    }
  }
</style>
