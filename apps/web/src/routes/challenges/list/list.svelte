<script lang="ts">
  import type { Challenge } from '@rctf/types'
  import { IconCaretDown, IconQuestion } from '$lib/icons'
  import Accordion from '$lib/ui/accordion.svelte'
  import EmptyState from '$lib/ui/empty-state.svelte'
  import { getCategoryConfig } from '$lib/utils/categories'
  import ChallengesListHeader from './list-header.svelte'
  import ChallengesListItem from './list-item.svelte'
  import {
    computeStats,
    deriveAccordionValue,
    filterChallenges,
    groupChallenges,
    resolveCategory,
  } from './list-logic'
  import { loadPreferences, savePreferences } from './preferences'

  interface Props {
    challenges: Challenge[]
    solvedIds: ReadonlySet<string>
    bloodIds: { gold: Set<string>; silver: Set<string>; bronze: Set<string> }
    selectedId: string | null
    onSelect: (challenge: Challenge) => void
    deepLinkTarget: string | null
  }

  let {
    challenges,
    solvedIds,
    bloodIds,
    selectedId,
    onSelect,
    deepLinkTarget,
  }: Props = $props()

  const initialPrefs = loadPreferences()
  let searchQuery = $state('')
  let hideSolved = $state(initialPrefs.hideSolved)
  let collapsedCategories = $state<string[]>(initialPrefs.collapsedCategories)

  const searching = $derived(searchQuery.trim().length > 0)

  const visibleChallenges = $derived.by(() => {
    const searched = filterChallenges(challenges, {
      query: searchQuery,
      hideSolved: false,
      solvedIds,
    })
    if (!hideSolved) return searched
    return searched.filter(
      challenge =>
        challenge.id === deepLinkTarget || !solvedIds.has(challenge.id)
    )
  })

  const groups = $derived(groupChallenges(visibleChallenges))
  const groupByCategory = $derived(
    new Map(groups.map(group => [group.category, group]))
  )
  const allCategories = $derived(groups.map(group => group.category))

  const deepLinkCategory = $derived.by(() => {
    if (!deepLinkTarget) return null
    const target = challenges.find(challenge => challenge.id === deepLinkTarget)
    return target ? resolveCategory(target.category) : null
  })

  const accordionValue = $derived(
    deriveAccordionValue({
      allCategories,
      collapsedCategories,
      searching,
      deepLinkCategory,
    })
  )
  const anyOpen = $derived(accordionValue.length > 0)

  const stats = $derived(computeStats(challenges, solvedIds))

  const emptySubtitle = $derived.by(() => {
    if (searching && hideSolved)
      return 'Try a different search or show solved challenges'
    if (searching) return 'Try a different search term'
    if (hideSolved) return 'All challenges have been solved!'
    return 'No challenges available'
  })

  function bloodTierOf(id: string): 'gold' | 'silver' | 'bronze' | null {
    if (bloodIds.gold.has(id)) return 'gold'
    if (bloodIds.silver.has(id)) return 'silver'
    if (bloodIds.bronze.has(id)) return 'bronze'
    return null
  }

  function toggleHideSolved() {
    hideSolved = !hideSolved
    savePreferences({ hideSolved, collapsedCategories })
  }

  function handleValueChange(open: string[]) {
    if (searching) return
    const openSet = new Set(open)
    const before = new Set(accordionValue)
    const next = [...collapsedCategories]
    let changed = false
    for (const category of allCategories) {
      const nowOpen = openSet.has(category)
      if (nowOpen === before.has(category)) continue
      const index = next.indexOf(category)
      if (nowOpen && index !== -1) {
        next.splice(index, 1)
        changed = true
      } else if (!nowOpen && index === -1) {
        next.push(category)
        changed = true
      }
    }
    if (changed) {
      collapsedCategories = next
      savePreferences({ hideSolved, collapsedCategories: next })
    }
  }

  function toggleCollapseAll() {
    const rendered = new Set(allCategories)
    const preserved = collapsedCategories.filter(
      category => !rendered.has(category)
    )
    const next = anyOpen ? [...preserved, ...allCategories] : preserved
    const deduped = [...new Set(next)]
    collapsedCategories = deduped
    savePreferences({ hideSolved, collapsedCategories: deduped })
  }
</script>

<challenges-list>
  <ChallengesListHeader
    pointsEarned={stats.pointsEarned}
    pointsTotal={stats.pointsTotal}
    solvedCount={stats.solvedCount}
    totalCount={stats.totalCount}
    {searchQuery}
    {hideSolved}
    {anyOpen}
    onSearchChange={value => (searchQuery = value)}
    onToggleHideSolved={toggleHideSolved}
    onToggleCollapse={toggleCollapseAll}
  />

  <list-scroll tabindex="-1">
    {#if groups.length === 0}
      <EmptyState
        icon={IconQuestion}
        title="No challenges found"
        subtitle={emptySubtitle}
      />
    {:else}
      <Accordion
        items={allCategories}
        value={accordionValue}
        onValueChange={handleValueChange}
      >
        {#snippet header({ value, props, expanded })}
          {@const config = getCategoryConfig(value)}
          {@const entries = groupByCategory.get(value)?.challenges ?? []}
          {@const solvedInCategory = entries.filter(c =>
            solvedIds.has(c.id)
          ).length}
          <challenges-list-group-header>
            <button {...props} data-expanded={expanded || undefined}>
              <span data-slot="name">{config.name}</span>
              <span data-slot="count">
                <strong>{solvedInCategory}</strong> / {entries.length}
              </span>
              <IconCaretDown data-slot="chevron" />
            </button>
          </challenges-list-group-header>
        {/snippet}

        {#snippet content({ value, props })}
          {@const entries = groupByCategory.get(value)?.challenges ?? []}
          <challenges-list-group-body {...props}>
            <ul>
              {#each entries as challenge (challenge.id)}
                <ChallengesListItem
                  {challenge}
                  category={value}
                  solved={solvedIds.has(challenge.id)}
                  bloodTier={bloodTierOf(challenge.id)}
                  selected={selectedId === challenge.id}
                  onSelect={() => onSelect(challenge)}
                />
              {/each}
            </ul>
          </challenges-list-group-body>
        {/snippet}
      </Accordion>
    {/if}
  </list-scroll>
</challenges-list>

<style>
  challenges-list {
    --list-gutter: 1.5rem;

    container-type: inline-size;
    container-name: challenges-list;
    display: flex;
    flex: 1;
    flex-direction: column;
    min-block-size: 0;
  }

  list-scroll {
    flex: 1;
    min-block-size: 0;
    overflow-y: auto;
    overscroll-behavior: none;
    padding-block-end: var(--space-l);
  }

  challenges-list-group-header {
    position: sticky;
    inset-block-start: 0;
    z-index: 2;
    display: block;
    background: var(--background-l0);

    button {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      inline-size: 100%;
      padding: 1.75rem var(--list-gutter) 0.625rem;
      text-align: start;
      cursor: pointer;
      border-block-end: 1px solid var(--border);

      &:focus-visible {
        outline: 2px solid var(--ring);
        outline-offset: -2px;
      }

      &:hover [data-slot='name'] {
        opacity: 0.6;
      }

      &[data-expanded] :global([data-slot='chevron']) {
        rotate: 0deg;
      }
    }

    [data-slot='name'] {
      font-size: var(--eyebrow-size);
      font-weight: var(--eyebrow-weight);
      letter-spacing: var(--eyebrow-tracking);
      text-transform: uppercase;
      color: var(--foreground-l0);
      transition: opacity 120ms ease;
    }

    [data-slot='count'] {
      margin-inline-start: auto;
      font-size: 0.75rem;
      letter-spacing: 0.02em;
      color: var(--foreground-l4);
      white-space: nowrap;
      font-variant-numeric: tabular-nums;

      strong {
        color: var(--foreground-l2);
        font-weight: 400;
      }
    }

    :global([data-slot='chevron']) {
      flex-shrink: 0;
      font-size: 0.625rem;
      color: var(--foreground-l4);
      rotate: -90deg;
      transition: rotate 150ms ease;
    }
  }

  challenges-list-group-body {
    display: block;

    ul {
      display: flex;
      flex-direction: column;
      margin: 0;
      padding: 0;
      list-style: none;
    }
  }
</style>
