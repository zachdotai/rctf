<script lang="ts">
  import * as splitter from '@zag-js/splitter'
  import { normalizeProps, useMachine } from '@zag-js/svelte'
  import type { Snippet } from 'svelte'

  type Panel = { id: string; minSize?: number; maxSize?: number }

  type Props = {
    panels: [Panel, Panel]
    defaultSize: [number, number]
    a: Snippet
    b: Snippet
  }

  let { panels, defaultSize, a, b }: Props = $props()

  const id = $props.id()
  const service = useMachine(splitter.machine, () => ({
    id,
    defaultSize,
    panels,
  }))
  const api = $derived(splitter.connect(service, normalizeProps))

  const resizeTriggerId = $derived(
    `${panels[0].id}:${panels[1].id}` as `${string}:${string}`
  )
</script>

<div {...api.getRootProps()}>
  <div {...api.getPanelProps({ id: panels[0].id })}>
    {@render a()}
  </div>
  <div {...api.getResizeTriggerProps({ id: resizeTriggerId })}>
    <span {...api.getResizeTriggerIndicator({ id: resizeTriggerId })}></span>
  </div>
  <div {...api.getPanelProps({ id: panels[1].id })}>
    {@render b()}
  </div>
</div>

<style>
  [data-part='root'] {
    block-size: 100%;
  }

  [data-part='panel'] {
    overflow: hidden;
  }

  [data-part='resize-trigger'] {
    display: flex;
    align-items: stretch;
    justify-content: flex-start;
    inline-size: var(--splitter-handle-size, var(--space-2xs));

    &::before {
      content: '';
      inline-size: 1px;
      background: var(--border);
      transition: background-color 120ms ease;
    }

    &[data-dragging]::before,
    &:hover::before {
      background: var(--foreground-l4);
    }
  }

  [data-part='resize-trigger-indicator'] {
    display: none;
  }
</style>
