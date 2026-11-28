<script lang="ts">
  import { IconSearch } from '$lib/icons'

  interface Props {
    pointsEarned: number
    pointsTotal: number
    solvedCount: number
    totalCount: number
    searchQuery: string
    hideSolved: boolean
    anyOpen: boolean
    onSearchChange: (value: string) => void
    onToggleHideSolved: () => void
    onToggleCollapse: () => void
  }

  let {
    pointsEarned,
    pointsTotal,
    solvedCount,
    totalCount,
    searchQuery,
    hideSolved,
    anyOpen,
    onSearchChange,
    onToggleHideSolved,
    onToggleCollapse,
  }: Props = $props()
</script>

<challenges-list-header>
  <search-box>
    <IconSearch />
    <input
      type="search"
      value={searchQuery}
      oninput={event => onSearchChange(event.currentTarget.value)}
      placeholder="Search challenges"
      aria-label="Search challenges"
      autocomplete="off"
      autocapitalize="off"
      spellcheck="false"
    />
  </search-box>

  <list-meta>
    <list-stats>
      <span
        ><strong>{pointsEarned.toLocaleString()}</strong> of {pointsTotal.toLocaleString()}
        pts</span
      >
      <span aria-hidden="true">·</span>
      <span
        ><strong>{solvedCount.toLocaleString()}</strong> of {totalCount.toLocaleString()}
        solved</span
      >
    </list-stats>

    <list-toggles>
      <button
        type="button"
        onclick={onToggleHideSolved}
        data-active={hideSolved || undefined}
        aria-pressed={hideSolved}
      >
        Hide solved
      </button>
      <button type="button" onclick={onToggleCollapse}>
        {anyOpen ? 'Collapse all' : 'Expand all'}
      </button>
    </list-toggles>
  </list-meta>
</challenges-list-header>

<style>
  challenges-list-header {
    display: flex;
    flex-shrink: 0;
    flex-direction: column;
    gap: 0.75rem;
    padding: 1.25rem var(--list-gutter) 0.25rem;
  }

  search-box {
    position: relative;
    display: flex;
    align-items: center;
    block-size: 2.5rem;

    :global(svg) {
      position: absolute;
      inset-inline-start: 0.75rem;
      font-size: 1rem;
      color: var(--foreground-l4);
      pointer-events: none;
    }
  }

  input {
    inline-size: 100%;
    block-size: 100%;
    padding-inline: 2.25rem 0.75rem;
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
  }

  list-meta {
    display: flex;
    flex-wrap: wrap;
    align-items: baseline;
    justify-content: space-between;
    gap: 0.25rem 1rem;
    font-size: 0.8125rem;
    color: var(--foreground-l3);
  }

  list-stats {
    display: flex;
    gap: 0.375rem;
    white-space: nowrap;
    font-variant-numeric: tabular-nums;

    strong {
      color: var(--foreground-l0);
      font-weight: 400;
    }
  }

  list-toggles {
    display: flex;
    gap: 1rem;

    button {
      font-size: 0.8125rem;
      letter-spacing: 0.02em;
      color: var(--foreground-l3);
      white-space: nowrap;
      cursor: pointer;
      transition: color 120ms ease;

      &:hover {
        color: var(--foreground-l0);
      }

      &[data-active] {
        color: var(--foreground-l0);
        text-decoration: underline;
        text-decoration-thickness: 1px;
        text-underline-offset: 0.3em;
      }
    }
  }
</style>
