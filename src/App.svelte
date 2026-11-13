<script lang="ts">
  import Board from './Board.svelte'
  import Toolbar from './Toolbar.svelte'

  let color = $state('#1e1e1e')
  let width = $state(2)
</script>

<main class="fixed inset-0">
  <Board {color} {width} />
  <Toolbar bind:color bind:width />
</main>
