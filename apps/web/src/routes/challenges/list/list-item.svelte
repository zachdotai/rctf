<script lang="ts">
  import { ChallengeScoringKind, type Challenge } from '@rctf/types'
  import { IconAwardFilled, IconCheck } from '$lib/icons'
  import ChallengePointDelta from '../model/point-delta.svelte'

  type BloodTier = 'gold' | 'silver' | 'bronze'

  interface Props {
    challenge: Challenge
    category: string
    solved: boolean
    bloodTier: BloodTier | null
    selected: boolean
    onSelect: () => void
  }

  let { challenge, solved, bloodTier, selected, onSelect }: Props = $props()

  const isDynamic = $derived(
    challenge.scoringKind === ChallengeScoringKind.DYNAMIC
  )
  const showsScore = $derived(challenge.hasFlag || isDynamic)
  const displayPoints = $derived(
    isDynamic ? (challenge.yourScore ?? 0) : challenge.points
  )
  const solvesLabel = $derived(
    `${challenge.solves.toLocaleString()} ${challenge.solves === 1 ? 'solve' : 'solves'}`
  )
</script>

<li id="chall-{challenge.id}">
  <button
    type="button"
    onclick={onSelect}
    data-solved={solved || bloodTier ? '' : undefined}
    data-blood={bloodTier ?? undefined}
    data-selected={selected ? '' : undefined}
  >
    <item-main>
      <item-title>
        <span data-part="name">{challenge.name}</span>
        {#if bloodTier}
          <IconAwardFilled data-indicator aria-label="{bloodTier} blood" />
        {:else if solved}
          <IconCheck data-indicator aria-label="Solved" />
        {/if}
      </item-title>
      <span data-part="author">by {challenge.author}</span>
    </item-main>

    {#if showsScore}
      <item-score>
        <span data-part="points">
          {#if isDynamic && displayPoints === 0}
            <strong>Dynamic</strong>
          {:else}
            <strong>{displayPoints.toLocaleString()}</strong> pts
          {/if}
        </span>
        {#if isDynamic}
          <ChallengePointDelta delta={challenge.yourPointDelta ?? 0} />
        {:else}
          <span data-part="solves">{solvesLabel}</span>
        {/if}
      </item-score>
    {/if}
  </button>
</li>

<style>
  li {
    display: block;
    border-block-end: 1px solid var(--border);
  }

  button {
    position: relative;
    display: flex;
    flex-direction: column;
    gap: 0.375rem;
    inline-size: 100%;
    padding: 0.9375rem var(--list-gutter);
    text-align: start;
    cursor: pointer;
    transition: background-color 120ms ease;

    &:hover {
      background: var(--background-l2);
    }

    &:focus-visible {
      outline: 2px solid var(--ring);
      outline-offset: -2px;
      z-index: 1;
    }

    &[data-selected] {
      background: var(--background-l3);
    }

    &[data-solved] [data-part='name'] {
      color: var(--foreground-l2);
    }
  }

  item-main {
    display: flex;
    flex-direction: column;
    gap: 0.125rem;
    min-inline-size: 0;
  }

  item-title {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    min-inline-size: 0;

    :global(svg[data-indicator]) {
      flex-shrink: 0;
      font-size: 0.9375rem;
      color: var(--foreground-success);
    }
  }

  [data-blood='gold'] :global(svg[data-indicator]) {
    color: var(--foreground-gold-l0);
  }

  [data-blood='silver'] :global(svg[data-indicator]) {
    color: var(--foreground-silver-l0);
  }

  [data-blood='bronze'] :global(svg[data-indicator]) {
    color: var(--foreground-bronze-l0);
  }

  [data-part='name'] {
    overflow: hidden;
    font-size: 1.0625rem;
    font-weight: 500;
    line-height: 1.35;
    color: var(--foreground-l0);
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  [data-part='author'] {
    overflow: hidden;
    font-size: 0.8125rem;
    color: var(--foreground-l3);
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  item-score {
    display: flex;
    flex-wrap: wrap;
    gap: var(--space-2xs);
    align-items: baseline;
    white-space: nowrap;
    font-variant-numeric: tabular-nums;
  }

  [data-part='points'] {
    font-size: 0.9375rem;
    color: var(--foreground-l3);

    strong {
      color: var(--foreground-l0);
      font-weight: 400;
    }
  }

  [data-part='solves'] {
    font-size: 0.8125rem;
    color: var(--foreground-l3);
  }

  @container challenges-list (min-inline-size: 24rem) {
    button {
      flex-direction: row;
      align-items: center;
      justify-content: space-between;
      gap: 1rem;
    }

    item-score {
      flex-direction: column;
      align-items: flex-end;
      gap: 0.125rem;
    }
  }
</style>
