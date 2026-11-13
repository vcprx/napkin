<script lang="ts">
  import { devicePixelRatio } from 'svelte/reactivity/window'

  type Stroke = { color: string; width: number; points: { x: number; y: number }[] }

  let { color, width }: { color: string; width: number } = $props()

  let canvas: HTMLCanvasElement
  let size: DOMRectReadOnly | undefined = $state()
  const strokes: Stroke[] = []
  let active: Stroke | null = null
  let pointerId = -1

  /** Draws a stroke from point `from` onward (continuing from the point before it). */
  function draw(stroke: Stroke, from = 0) {
    const ctx = canvas.getContext('2d')!
    const [first, ...rest] = stroke.points.slice(Math.max(from - 1, 0))
    ctx.lineCap = 'round'
    ctx.lineJoin = 'round'
    ctx.strokeStyle = stroke.color
    ctx.lineWidth = stroke.width
    ctx.beginPath()
    ctx.moveTo(first.x, first.y)
    for (const p of rest) ctx.lineTo(p.x, p.y)
    // Canvas skips zero-length segments, so a lone point needs a tiny one to show as a dot.
    if (rest.length === 0) ctx.lineTo(first.x + 0.01, first.y)
    ctx.stroke()
  }

  /** Sizes the backing store for crisp lines on high-DPI screens; resizing clears it, so redraw. */
  function resize(rect: DOMRectReadOnly, dpr: number) {
    canvas.width = Math.round(rect.width * dpr)
    canvas.height = Math.round(rect.height * dpr)
    canvas.getContext('2d')!.setTransform(dpr, 0, 0, dpr, 0, 0)
    for (const stroke of strokes) draw(stroke)
  }

  // Also reruns when only the pixel ratio changes, e.g. moving the window to another monitor.
  $effect(() => {
    if (size) resize(size, devicePixelRatio.current!)
  })

  function toPoint(e: PointerEvent) {
    const rect = canvas.getBoundingClientRect()
    return { x: e.clientX - rect.left, y: e.clientY - rect.top }
  }

  function onpointerdown(e: PointerEvent) {
    if (active || e.button !== 0) return
    canvas.setPointerCapture(e.pointerId)
    pointerId = e.pointerId
    active = { color, width, points: [toPoint(e)] }
    strokes.push(active)
    draw(active)
  }

  function onpointermove(e: PointerEvent) {
    if (!active || e.pointerId !== pointerId) return
    const from = active.points.length
    const events = e.getCoalescedEvents?.() ?? []
    for (const event of events.length > 0 ? events : [e]) active.points.push(toPoint(event))
    draw(active, from)
  }

  function onpointerend(e: PointerEvent) {
    if (e.pointerId === pointerId) active = null
  }
</script>

<canvas
  class="block size-full touch-none"
  bind:this={canvas}
  bind:contentRect={size}
  {onpointerdown}
  {onpointermove}
  onpointerup={onpointerend}
  onpointercancel={onpointerend}
></canvas>
