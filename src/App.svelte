<script lang="ts">
  import Board from './Board.svelte'
  import Toolbar from './Toolbar.svelte'

  let color = $state('#00e5ff')
  let width = $state(2)
</script>

<main class="fixed inset-0 bg-zinc-950">
  <Board {color} {width} />
  <Toolbar bind:color bind:width />
</main>
