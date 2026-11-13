<script lang="ts">
  import Board from './Board.svelte'
  import Toolbar from './Toolbar.svelte'

  let color = $state('#00e5ff')
  let width = $state(2)
  let board: Board | undefined = $state()
</script>

<main class="fixed inset-0 bg-zinc-950">
  <Board bind:this={board} {color} {width} />
  <Toolbar
    bind:color
    bind:width
    canUndo={board?.canUndo() ?? false}
    onundo={() => board?.undo()}
  />
</main>
