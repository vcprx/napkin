<script lang="ts">
  import type { Snippet } from 'svelte'

  let { onclick, children }: { onclick: () => void; children: Snippet } = $props()
</script>

<button
  type="button"
  class="grid size-9 cursor-pointer place-items-center rounded-lg hover:bg-gray-100 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600"
  {onclick}
>
  {@render children()}
</button>
