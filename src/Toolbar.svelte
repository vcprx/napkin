<script lang="ts">
  import BrushStyleButton from './BrushStyleButton.svelte'

  let { color = $bindable(), width = $bindable() }: { color: string; width: number } = $props()

  const colors = ['#1e1e1e', '#e03131', '#1971c2', '#2f9e44', '#f08c00']
  const widths = [1, 2, 4, 8, 16]

  // Opens above its trigger; mb-4 clears the toolbar's padding plus a gap.
  const drawer =
    'absolute bottom-full left-1/2 mb-4 flex -translate-x-1/2 flex-col gap-1 rounded-xl bg-white p-2 shadow-md'

  let toolbar: HTMLDivElement
  let open: 'color' | 'width' | null = $state(null)

  const toggle = (drawer: 'color' | 'width') => (open = open === drawer ? null : drawer)

  // Capture phase, so a press outside only closes the drawer and never reaches the canvas.
  function onpointerdowncapture(e: PointerEvent) {
    if (!open || toolbar.contains(e.target as Node)) return
    e.stopPropagation()
    open = null
  }

  function onkeydown(e: KeyboardEvent) {
    if (e.key === 'Escape') open = null
  }
</script>

<svelte:window {onpointerdowncapture} {onkeydown} />

{#snippet colorSwatch(value: string)}
  <span class="size-5 rounded-full" style:background={value}></span>
{/snippet}

{#snippet widthSwatch(value: number)}
  <span class="w-5 rounded-full" style:background={color} style:height="{value}px"></span>
{/snippet}

<div
  bind:this={toolbar}
  class="absolute bottom-3 left-1/2 flex -translate-x-1/2 gap-1 rounded-xl bg-white p-2 shadow-md"
>
  <div class="relative">
    <BrushStyleButton onclick={() => toggle('color')}>{@render colorSwatch(color)}</BrushStyleButton>
    {#if open === 'color'}
      <div class={drawer}>
        {#each colors as value (value)}
          <BrushStyleButton onclick={() => ((color = value), (open = null))}>
            {@render colorSwatch(value)}
          </BrushStyleButton>
        {/each}
      </div>
    {/if}
  </div>
  <div class="relative">
    <BrushStyleButton onclick={() => toggle('width')}>{@render widthSwatch(width)}</BrushStyleButton>
    {#if open === 'width'}
      <div class={drawer}>
        {#each widths as value (value)}
          <BrushStyleButton onclick={() => ((width = value), (open = null))}>
            {@render widthSwatch(value)}
          </BrushStyleButton>
        {/each}
      </div>
    {/if}
  </div>
</div>
