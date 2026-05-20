# Examples

Copy-pasteable skeletons. Pair with [SKILL.md](SKILL.md) for the workflow and [REFERENCE.md](REFERENCE.md) for the gotchas.

## Building blocks

Small reusable shapes used across all three archetypes.

### Caption

A right-side eyebrow + name + sub line, with a dividing border. Used in the architecture diagram and most figures with paired explanations.

```tsx
function Caption({ title, name, sub }: { title: string; name: string; sub: string }) {
  return (
    <aside className="flex flex-col gap-1 border-l border-black/10 pl-5 max-[640px]:border-l-0 max-[640px]:border-t max-[640px]:border-black/10 max-[640px]:pl-0 max-[640px]:pt-2.5">
      <span className="font-mono text-[10px] font-semibold uppercase tracking-[0.28em] text-[#2e7d32]">
        {title}
      </span>
      <span className="font-sans text-base font-semibold tracking-[-0.01em] text-[color:var(--ink)]">
        {name}
      </span>
      <span className="text-xs text-[color:var(--ink-muted)]">{sub}</span>
    </aside>
  );
}
```

### Dark slab

A high-contrast container for "shared / durable" things (Modal Dict, Queue, DB). White slabs for in-process things.

```tsx
function Slab({ title, active, children }: { title: string; active: boolean; children: React.ReactNode }) {
  return (
    <section
      className={`rounded-[0.85rem] border bg-[#0e0f11] text-[#f3f3f0] px-4 py-3.5 transition-[border-color,box-shadow,transform] duration-200 will-change-transform ${
        active ? "border-[#2e7d32] shadow-[0_0_0_3px_rgba(46,125,50,0.12)] -translate-y-px" : "border-[#0e0f11]"
      }`}
    >
      <div className="mb-2.5 font-mono text-[10px] font-semibold uppercase tracking-[0.22em] text-white/85">
        {title}
      </div>
      {children}
    </section>
  );
}
```

### Token (animated)

A small badge that travels between slots. Uses asymmetric transitions so the return motion is hidden by the fade-out.

```tsx
function Token({
  x, y, visible, letter, accent,
}: {
  x: number;       // % of stage width
  y: number;       // % of stage height
  visible: boolean;
  letter: string;
  accent: string;  // e.g. "#2e7d32"
}) {
  return (
    <span
      className="pointer-events-none absolute z-[3] inline-flex items-center justify-center rounded-full border-2 border-white text-[10px] font-semibold text-white! shadow-[0_6px_16px_-8px_rgba(20,20,19,0.35)] will-change-transform h-[1.45rem] w-[1.45rem]"
      style={{
        left: `${x}%`,
        top: `${y}%`,
        background: accent,
        opacity: visible ? 1 : 0,
        transform: "translate(-50%, -50%)",
        // Asymmetric: visible state slides smoothly, hidden state fades fast + delays transform reset
        transition: visible
          ? "left 0.45s cubic-bezier(0.4,0,0.2,1), top 0.45s cubic-bezier(0.4,0,0.2,1), opacity 0.22s ease, background-color 0.2s ease"
          : "left 0s 0.22s, top 0s 0.22s, opacity 0.18s ease",
      }}
      aria-hidden
    >
      {letter}
    </span>
  );
}
```

---

## Static diagram

Layered concept overview, no animation. Three rows, each with a layer card on the left and a caption on the right.

```tsx
import { ArrowDownUp } from "lucide-react";

export function ArchitectureDiagram() {
  const row = "grid grid-cols-[minmax(0,1.35fr)_minmax(0,1fr)] items-center gap-7 max-[640px]:grid-cols-1 max-[640px]:gap-3";
  return (
    <figure
      className="not-prose my-9 rounded-3xl border border-black/10 bg-white p-8 pb-9 font-mono text-[color:var(--ink)] shadow-[0_22px_60px_-45px_rgba(20,20,19,0.35)]"
      aria-label="Architecture diagram"
    >
      <div className="grid gap-0">
        <div className={row}>
          <LayerCard>Layer 1 content</LayerCard>
          <Caption title="L1" name="In-process" sub="cheapest · per-container" />
        </div>
        <Rule />
        <div className={row}>
          <LayerCard>Layer 2 content</LayerCard>
          <Caption title="L2" name="Shared store" sub="network · coordination" />
        </div>
        <Rule />
        <div className={row}>
          <LayerCard>Layer 3 content</LayerCard>
          <Caption title="L3" name="Source of truth" sub="durable · slow" />
        </div>
      </div>
    </figure>
  );
}

function LayerCard({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex min-h-16 items-center justify-center rounded-[0.85rem] border border-black/10 bg-black/[0.02] px-4 py-4">
      {children}
    </div>
  );
}

function Rule() {
  return (
    <div className="grid grid-cols-[minmax(0,1.35fr)_minmax(0,1fr)] gap-7 max-[640px]:grid-cols-1" aria-hidden>
      <div className="flex flex-col items-center self-stretch py-1.5">
        <RuleSegment />
        <span className="my-0.5 inline-flex h-[22px] w-[22px] items-center justify-center rounded-full border border-black/20 bg-white text-[color:var(--ink-muted)]">
          <ArrowDownUp size={12} strokeWidth={1.5} />
        </span>
        <RuleSegment />
      </div>
      <span className="max-[640px]:hidden" />
    </div>
  );
}

function RuleSegment() {
  return (
    <span
      className="block h-3.5 w-px bg-repeat-y bg-[length:1px_5px]"
      style={{
        backgroundImage:
          "linear-gradient(to bottom, rgba(20,20,19,0.2) 0%, rgba(20,20,19,0.2) 50%, transparent 50%, transparent 100%)",
      }}
    />
  );
}
```

---

## Phase machine

Auto-advancing sequence with play/pause/restart + clickable timeline. The user can scrub to any phase.

```tsx
import { Pause, Play, RotateCcw } from "lucide-react";
import { useCallback, useEffect, useRef, useState } from "react";

type Phase = "idle" | "step1" | "step2" | "done";

const FRAMES: { name: Phase; duration: number; title: string; short: string; caption: string }[] = [
  { name: "idle",  duration: 1200, title: "idle",  short: "idle",  caption: "Starting state." },
  { name: "step1", duration: 1500, title: "step 1", short: "step1", caption: "First thing happens." },
  { name: "step2", duration: 1500, title: "step 2", short: "step2", caption: "Second thing happens." },
  { name: "done",  duration: 1500, title: "done",  short: "done",  caption: "Sequence complete." },
];

const NODES = {
  start: { x: 10, y: 50 },
  mid:   { x: 50, y: 50 },
  end:   { x: 90, y: 50 },
};

const TRACKS: Record<Phase, { x: number; y: number; visible: boolean }> = {
  idle:  { ...NODES.start, visible: false },
  step1: { ...NODES.start, visible: true },
  step2: { ...NODES.mid,   visible: true },
  done:  { ...NODES.end,   visible: true },
};

export function PhaseDiagram() {
  const [frameIndex, setFrameIndex] = useState(0);
  const [playing, setPlaying] = useState(true);
  const timerRef = useRef<number | null>(null);
  const phase = FRAMES[frameIndex].name;

  useEffect(() => {
    if (!playing) return;
    const next = (frameIndex + 1) % FRAMES.length;
    timerRef.current = window.setTimeout(() => setFrameIndex(next), FRAMES[frameIndex].duration);
    return () => { if (timerRef.current) window.clearTimeout(timerRef.current); };
  }, [playing, frameIndex]);

  const pt = TRACKS[phase];

  return (
    <figure className="not-prose my-9 rounded-3xl border border-black/10 bg-white px-6 pb-5 pt-6 font-mono text-[color:var(--ink)]">
      <header className="mb-4 flex items-start justify-between gap-4 border-b border-black/10 pb-4">
        <div>
          <span className="mb-1.5 inline-block font-mono text-[10px] font-semibold uppercase tracking-[0.28em] text-[#2e7d32]">
            {FRAMES[frameIndex].title}
          </span>
          <p className="m-0 min-h-[3.1em] max-w-[56ch] font-sans text-[13.5px] leading-[1.55] text-[color:var(--ink)]">
            {FRAMES[frameIndex].caption}
          </p>
        </div>
        <div className="flex shrink-0 gap-1.5">
          <button
            type="button"
            className="inline-flex items-center gap-1.5 rounded-lg border border-black/20 bg-white px-2.5 py-1.5 font-mono text-[11px]"
            onClick={() => setPlaying((p) => !p)}
          >
            {playing ? <Pause size={12} strokeWidth={2} /> : <Play size={12} strokeWidth={2} />}
            {playing ? "pause" : "play"}
          </button>
          <button
            type="button"
            className="inline-flex items-center gap-1.5 rounded-lg border border-black/20 bg-white px-2.5 py-1.5 font-mono text-[11px]"
            onClick={() => { setFrameIndex(0); setPlaying(true); }}
          >
            <RotateCcw size={12} strokeWidth={2} /> restart
          </button>
        </div>
      </header>

      {/* stage */}
      <div className="relative w-full overflow-hidden rounded-[0.85rem] border border-black/10 bg-black/[0.015] aspect-[16/7] max-[640px]:aspect-[4/5]">
        <Token x={pt.x} y={pt.y} visible={pt.visible} letter="A" accent="#2e7d32" />
      </div>

      {/* timeline */}
      <ol className="m-0 mt-[1.1rem] flex list-none gap-1 p-0">
        {FRAMES.map((f, i) => (
          <li key={f.name} className="min-w-0 flex-shrink list-none" style={{ flex: f.duration }}>
            <button
              type="button"
              className={`flex w-full cursor-pointer flex-col items-start gap-1 border-0 border-t-2 bg-transparent px-2 pb-2.5 pt-2 text-left font-mono transition-colors ${
                i === frameIndex ? "border-t-[#2e7d32]! text-[color:var(--ink)]" : "border-t-black/10 text-[color:var(--ink-muted)]"
              }`}
              onClick={() => { setFrameIndex(i); setPlaying(false); }}
            >
              <span className="font-mono text-[9px] uppercase tracking-[0.22em]">
                {String(i + 1).padStart(2, "0")}
              </span>
              <span className="text-[10.5px]">{f.short}</span>
            </button>
          </li>
        ))}
      </ol>
    </figure>
  );
}
```

---

## Interactive simulator

User-driven state machine. Fire actions, observe state evolve, watch the log. The skeleton below is intentionally complete — copy it and replace the domain types with yours.

```tsx
import { useCallback, useEffect, useRef, useState } from "react";

type LogTone = "info" | "hit" | "miss";
type LogEntry = { id: number; tone: LogTone; text: string };

type State = {
  cache: Map<string, string>;
  // …other domain fields
};

const LOG_TONE: Record<LogTone, string> = {
  info: "text-[#f3f3f0]",
  hit:  "text-[#b6f0a5]",
  miss: "text-[#ffb454]",
};

const initialState = (): State => ({ cache: new Map() });
const sleep = (ms: number) => new Promise<void>((r) => setTimeout(r, ms));

export function MySimulator() {
  const [state, setState] = useState<State>(initialState);
  const [log, setLog] = useState<LogEntry[]>([]);
  const [busy, setBusy] = useState(false);
  const [speed, setSpeed] = useState(1);

  // Ref mirror so async handlers can read the latest state without racing
  // React's render cycle.
  const stateRef = useRef(state);
  const logIdRef = useRef(0);
  useEffect(() => { stateRef.current = state; }, [state]);

  const tick = useCallback(async (ms: number) => sleep(ms / speed), [speed]);

  const pushLog = useCallback((entry: Omit<LogEntry, "id">) => {
    logIdRef.current += 1;
    setLog((prev) => [{ id: logIdRef.current, ...entry }, ...prev].slice(0, 50));
  }, []);

  const reset = useCallback(() => {
    setState(initialState());
    setLog([]);
  }, []);

  const get = useCallback(async (key: string) => {
    if (busy) return;
    setBusy(true);
    try {
      pushLog({ tone: "info", text: `get ${key}` });
      await tick(400);
      const hit = stateRef.current.cache.has(key);
      if (hit) {
        pushLog({ tone: "hit", text: `hit ${key}` });
      } else {
        pushLog({ tone: "miss", text: `miss ${key} → compute` });
        await tick(800);
        setState((s) => ({
          ...s,
          cache: new Map(s.cache).set(key, `v_${key}_${Date.now() % 1000}`),
        }));
      }
    } finally {
      setBusy(false);
    }
  }, [busy, pushLog, tick]);

  return (
    <section
      className="not-prose my-12 rounded-3xl border border-black/10 bg-white p-7 pb-6 font-mono text-[12.5px] text-[color:var(--ink)] shadow-[0_22px_60px_-45px_rgba(20,20,19,0.4)]"
      aria-label="Simulator"
    >
      {/* header */}
      <header className="mb-6 flex items-start justify-between gap-6 border-b border-black/10 pb-5">
        <div className="flex flex-col gap-1.5">
          <span className="font-mono text-[10px] uppercase tracking-[0.28em] text-[color:var(--ink-muted)]">
            simulator
          </span>
          <h3 className="m-0 font-sans text-[19px] font-semibold tracking-[-0.01em]">
            My state machine
          </h3>
        </div>
      </header>

      {/* controls */}
      <div className="mb-6 flex flex-wrap items-center gap-3 border-b border-black/10 pb-5">
        <span className="font-mono text-[10px] uppercase tracking-[0.22em] text-[color:var(--ink-muted)]">
          speed
        </span>
        {[0.5, 1, 2].map((s) => (
          <button
            key={s}
            type="button"
            disabled={busy}
            onClick={() => setSpeed(s)}
            className={`rounded-full border border-black/20 bg-white px-2.5 py-[3px] font-mono text-[11px] disabled:opacity-45 ${
              speed === s ? "bg-[color:var(--ink)]! text-white! border-[color:var(--ink)]!" : ""
            }`}
          >
            {s}x
          </button>
        ))}
        <div className="ml-auto flex gap-2">
          <button
            type="button"
            disabled={busy}
            onClick={() => get("apple")}
            className="rounded-lg border border-black/20 bg-white px-3 py-1.5 font-mono text-[11px] disabled:opacity-45"
          >
            get apple
          </button>
          <button
            type="button"
            disabled={busy}
            onClick={() => get("banana")}
            className="rounded-lg border border-black/20 bg-white px-3 py-1.5 font-mono text-[11px] disabled:opacity-45"
          >
            get banana
          </button>
          <button
            type="button"
            disabled={busy}
            onClick={reset}
            className="rounded-lg border border-dashed border-black/20 bg-white px-3 py-1.5 font-mono text-[11px] text-[color:var(--ink-muted)] disabled:opacity-45"
          >
            reset
          </button>
        </div>
      </div>

      {/* board — your domain visualisation goes here */}
      <div className="mb-5 rounded-[0.85rem] border border-black/10 bg-black/[0.02] p-4">
        {state.cache.size === 0 ? (
          <div className="text-[11px] text-[color:var(--ink-muted)] opacity-60">empty</div>
        ) : (
          <ul className="list-none m-0 p-0 grid gap-1">
            {Array.from(state.cache.entries()).map(([k, v]) => (
              <li
                key={k}
                className="flex items-baseline justify-between border-b border-dashed border-black/10 py-1.5 text-[12px] last:border-b-0"
              >
                <code className="bg-transparent! p-0! font-mono text-[12px] text-[color:var(--ink)]!">
                  {k}
                </code>
                <span className="text-[11.5px] text-[#2e7d32]">{v}</span>
              </li>
            ))}
          </ul>
        )}
      </div>

      {/* event log */}
      <div className="mt-5 max-h-[260px] overflow-y-auto rounded-[0.85rem] bg-[#0e0f11] px-4 py-3.5 text-[11.5px] leading-[1.6] text-[#f3f3f0]" aria-live="polite">
        <div className="mb-2 flex items-center justify-between border-b border-dashed border-white/10 pb-2 font-mono text-[10px] uppercase tracking-[0.22em] text-white/55">
          <span>event stream</span>
          <span className="text-[9.5px] opacity-60">latest first</span>
        </div>
        {log.length === 0 ? (
          <p className="m-0 text-[11px] italic opacity-40">streaming output appears here</p>
        ) : (
          <ul className="list-none m-0 p-0 grid gap-[5px]">
            {log.map((entry) => (
              <li
                key={entry.id}
                className={`grid grid-cols-[1.85rem_1fr] items-baseline gap-2.5 ${LOG_TONE[entry.tone]}`}
              >
                <span className="rounded-[0.25rem] border border-white/10 px-1 py-[2px] text-center font-mono text-[9px] uppercase tracking-[0.16em] text-white/60">
                  {entry.tone}
                </span>
                <span>{entry.text}</span>
              </li>
            ))}
          </ul>
        )}
      </div>
    </section>
  );
}
```

Patterns worth understanding before you scale this up:

- **`useRef` mirror of state.** Async handlers that read state need a ref because the closure over `state` is stale between renders. Update the ref in a `useEffect` keyed on the state.
- **`busy` lock.** Prevents double-fires when the user mashes a button mid-animation. Disable every action button on `busy`.
- **`tick` with speed multiplier.** Lets users replay your animation at 0.5x / 1x / 2x without rewriting the timing math.
- **Bounded log.** Slice to 50 entries on every push. Without this, a long-running interaction will accumulate state forever and React re-renders will get slow.
- **Event tone palette.** Four standard tones (info / hit / miss + one for coordination) keep the visual language consistent across multiple simulators. Full palette in [REFERENCE.md](REFERENCE.md#tonal-palette-for-event-logs).
- **Co-located helpers stay at the bottom of the file.** Big simulators benefit from small `Stat`, `LegendDot`, `Lane`-type sub-components defined in the same file as the main component. Move them to bundle-local files only when one of them grows past ~50 lines.
