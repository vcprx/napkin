<script lang="ts">
  import type { Snippet } from 'svelte'

  let {
    onclick,
    disabled = false,
    label,
    children,
  }: { onclick: () => void; disabled?: boolean; label?: string; children: Snippet } = $props()
</script>

<button
  type="button"
  class="grid size-9 cursor-pointer place-items-center rounded-lg hover:bg-zinc-800 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-cyan-400 disabled:cursor-default disabled:opacity-40 disabled:hover:bg-transparent"
  {disabled}
  aria-label={label}
  {onclick}
>
  {@render children()}
</button>
