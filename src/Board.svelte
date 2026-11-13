<script lang="ts">
  import { untrack } from 'svelte'
  import { devicePixelRatio } from 'svelte/reactivity/window'

  type Stroke = { color: string; width: number; points: { x: number; y: number }[] }

  let { color, width }: { color: string; width: number } = $props()

  let canvas: HTMLCanvasElement
  let size: DOMRectReadOnly | undefined = $state()
  // Raw, so only reassigning the list is tracked; points pushed during a stroke stay plain.
  let strokes: Stroke[] = $state.raw([])
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

  function redraw() {
    const ctx = canvas.getContext('2d')!
    // Clear in device pixels; CSS units would miss part of the canvas when the pixel ratio is below 1.
    ctx.save()
    ctx.setTransform(1, 0, 0, 1, 0, 0)
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    ctx.restore()
    for (const stroke of strokes) draw(stroke)
  }

  /** Sizes the backing store for crisp lines on high-DPI screens; resizing clears it, so redraw. */
  function resize(rect: DOMRectReadOnly, dpr: number) {
    canvas.width = Math.round(rect.width * dpr)
    canvas.height = Math.round(rect.height * dpr)
    canvas.getContext('2d')!.setTransform(dpr, 0, 0, dpr, 0, 0)
    redraw()
  }

  // Also reruns when only the pixel ratio changes, e.g. moving the window to another monitor.
  // Untracked, so changes to `strokes` don't trigger a full redraw.
  $effect(() => {
    const rect = size
    const dpr = devicePixelRatio.current!
    if (rect) untrack(() => resize(rect, dpr))
  })

  export const canUndo = () => strokes.length > 0

  // Ignored mid-stroke, e.g. when a second finger taps the toolbar.
  export function undo() {
    if (active || strokes.length === 0) return
    strokes = strokes.slice(0, -1)
    redraw()
  }

  function toPoint(e: PointerEvent) {
    const rect = canvas.getBoundingClientRect()
    return { x: e.clientX - rect.left, y: e.clientY - rect.top }
  }

  function onpointerdown(e: PointerEvent) {
    if (active || e.button !== 0) return
    canvas.setPointerCapture(e.pointerId)
    pointerId = e.pointerId
    active = { color, width, points: [toPoint(e)] }
    strokes = [...strokes, active]
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
