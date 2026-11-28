<script lang="ts">
  import Markdown from '$lib/components/markdown.svelte'
  import Button from '$lib/ui/button.svelte'
  import type { PageProps } from './$types'

  const { data }: PageProps = $props()

  const titleWords = $derived(data.clientConfig.ctfName.trim().split(/\s+/))
  const leadWord = $derived(titleWords[0] ?? '')
  const restWords = $derived(titleWords.slice(1).join(' '))
</script>

<home-page>
  <home-hero>
    <h1><em>{leadWord}</em> {restWords}</h1>
    <hero-copy>
      <Markdown content={data.clientConfig.homeContent} />
    </hero-copy>
    <hero-actions>
      <Button href="/challenges" size="lg">View challenges</Button>
      <a href="/scores">Scoreboard</a>
    </hero-actions>
  </home-hero>

  {#if data.clientConfig.sponsors.length > 0}
    <home-sponsors>
      <sponsors-label>Sponsored by</sponsors-label>
      <sponsor-grid>
        {#each data.clientConfig.sponsors as sponsor (sponsor.name)}
          {@const lightIcon = sponsor.iconLight || sponsor.icon}
          {@const darkIcon = sponsor.iconDark}
          <svelte:element
            this={sponsor.url ? 'a' : 'article'}
            href={sponsor.url}
            target={sponsor.url ? '_blank' : undefined}
            rel={sponsor.url ? 'noopener noreferrer' : undefined}
          >
            {#if lightIcon || darkIcon}
              <sponsor-icon
                data-theme-visible="light"
                data-invert={lightIcon ? undefined : ''}
              >
                <img
                  src={lightIcon || darkIcon}
                  alt={sponsor.name}
                  loading="lazy"
                />
              </sponsor-icon>
              <sponsor-icon
                data-theme-visible="dark"
                data-invert={darkIcon ? undefined : ''}
              >
                <img
                  src={darkIcon || lightIcon}
                  alt={sponsor.name}
                  loading="lazy"
                />
              </sponsor-icon>
            {/if}
            <h3>{sponsor.name}</h3>
            <Markdown content={sponsor.description} />
          </svelte:element>
        {/each}
      </sponsor-grid>
    </home-sponsors>
  {/if}

  <footer>
    Powered by
    <a href="https://rctf.osec.io" target="_blank" rel="noopener noreferrer"
      >rCTF</a
    >
  </footer>
</home-page>

<style>
  home-page {
    display: flex;
    flex: 1;
    flex-direction: column;
    inline-size: 100%;
    padding-inline: 1rem;

    @media (width >= 48rem) {
      padding-inline: 2.25rem;
    }
  }

  home-hero {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: var(--space-m);
    min-block-size: calc(78dvh - var(--header-height));
    padding-block: var(--space-2xl) var(--space-xl);
    text-align: center;
  }

  h1 {
    max-inline-size: 16ch;
    margin: 0;
    font-size: clamp(2.75rem, 1.6rem + 5vw, 5.5rem);
    line-height: 1.04;
    letter-spacing: -0.02em;

    em {
      font-style: italic;
    }
  }

  hero-copy {
    display: block;
    max-inline-size: 34rem;

    :global(prose-content) {
      font-size: 1.1875rem;
      line-height: 1.6;
      color: var(--foreground-l1);
    }
  }

  hero-actions {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
    gap: var(--space-m);
    margin-block-start: var(--space-2xs);

    a:not([data-variant]) {
      --underline: currentColor;

      font-size: 0.9375rem;
      letter-spacing: 0.02em;
      color: var(--foreground-l0);
      text-underline-offset: 0.3em;
      transition: opacity 120ms ease;

      &:hover {
        opacity: 0.6;
      }
    }
  }

  home-sponsors {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: var(--space-m);
    inline-size: 100%;
    max-inline-size: 64rem;
    margin-inline: auto;
    padding-block: var(--space-l);
    border-block-start: 1px solid var(--border);
  }

  sponsors-label {
    font-size: var(--eyebrow-size);
    font-weight: var(--eyebrow-weight);
    letter-spacing: 0.3em;
    text-transform: uppercase;
    color: var(--foreground-l3);
  }

  sponsor-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(14rem, 1fr));
    gap: var(--space-m);
    inline-size: 100%;

    a,
    article {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: var(--space-2xs);
      padding: var(--space-s);
      text-align: center;
      text-decoration: none;
      color: inherit;
      border-radius: var(--radius-xl);
    }

    a:hover {
      background: var(--background-l2);
    }

    sponsor-icon[data-invert] img {
      filter: invert(1);
    }

    img {
      inline-size: 100%;
      block-size: auto;
      max-block-size: 4rem;
      object-fit: contain;
    }

    h3 {
      font-size: var(--step-1);
    }
  }

  footer {
    margin-block-start: auto;
    padding-block: var(--space-m);
    text-align: center;
    font-size: 0.8125rem;
    color: var(--foreground-l4);

    a {
      --underline: currentColor;
    }
  }
</style>
