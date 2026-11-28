<script lang="ts">
  import { ChallengeScoringKind, type Challenge } from '@rctf/types'
  import Chip from '$lib/ui/chip.svelte'
  import { getCategoryConfig } from '$lib/utils/categories'
  import ChallengePointDelta from '../model/point-delta.svelte'

  interface Props {
    challenge: Challenge
    isSolved: boolean
  }

  let { challenge, isSolved }: Props = $props()

  const config = $derived(getCategoryConfig(challenge.category))
  const tags = $derived(challenge.tags ?? [])
  const isDynamic = $derived(
    challenge.scoringKind === ChallengeScoringKind.DYNAMIC
  )
  const showsScore = $derived(challenge.hasFlag || isDynamic)
  const displayPoints = $derived(
    isDynamic ? (challenge.yourScore ?? 0) : challenge.points
  )
</script>

<details-header>
  <header-main>
    <header-eyebrow>
      <span data-slot="category">{config.name}</span>
      {#each tags as tag (tag)}
        <Chip>{tag}</Chip>
      {/each}
      {#if isSolved}
        <span data-slot="solved">Solved</span>
      {/if}
    </header-eyebrow>
    <h2>{challenge.name}</h2>
    <span data-slot="author">by {challenge.author}</span>
  </header-main>

  {#if showsScore}
    <header-stats>
      <stat-item>
        <stat-value>{displayPoints.toLocaleString()}</stat-value>
        <stat-label>{isDynamic ? 'your score' : 'points'}</stat-label>
      </stat-item>
      {#if isDynamic}
        <stat-item>
          <ChallengePointDelta delta={challenge.yourPointDelta ?? 0} />
        </stat-item>
      {:else}
        <stat-item>
          <stat-value>{challenge.solves.toLocaleString()}</stat-value>
          <stat-label>{challenge.solves === 1 ? 'solve' : 'solves'}</stat-label>
        </stat-item>
      {/if}
    </header-stats>
  {/if}
</details-header>

<style>
  details-header {
    display: flex;
    flex-wrap: wrap;
    align-items: flex-end;
    justify-content: space-between;
    gap: 1.25rem 2rem;
    padding: 2rem var(--detail-gutter) 1.5rem;
  }

  header-main {
    display: flex;
    flex: 1 1 18rem;
    flex-direction: column;
    gap: 0.5rem;
    min-inline-size: 0;
  }

  header-eyebrow {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 0.375rem 0.625rem;

    :global(ui-chip) {
      font-size: 0.6875rem;
      letter-spacing: 0.1em;
      text-transform: uppercase;
    }
  }

  [data-slot='category'],
  [data-slot='solved'] {
    font-size: var(--eyebrow-size);
    font-weight: var(--eyebrow-weight);
    letter-spacing: var(--eyebrow-tracking);
    text-transform: uppercase;
    color: var(--foreground-l3);
  }

  [data-slot='solved'] {
    color: var(--foreground-success);
  }

  h2 {
    margin: 0;
    font-size: clamp(1.75rem, 1.4rem + 1.4cqi, 2.5rem);
    line-height: 1.1;
    overflow-wrap: anywhere;
  }

  [data-slot='author'] {
    font-size: 0.875rem;
    color: var(--foreground-l3);
  }

  header-stats {
    display: flex;
    flex-shrink: 0;
    gap: 2rem;
  }

  stat-item {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    gap: 0.125rem;
  }

  stat-value {
    font-family: var(--font-display);
    font-size: 1.75rem;
    line-height: 1;
    font-variant-numeric: lining-nums tabular-nums;
    color: var(--foreground-l0);
  }

  stat-label {
    font-size: var(--eyebrow-size);
    font-weight: var(--eyebrow-weight);
    letter-spacing: var(--eyebrow-tracking);
    text-transform: uppercase;
    color: var(--foreground-l3);
  }
</style>
