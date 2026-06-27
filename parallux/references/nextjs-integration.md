# Next.js / React integration

Use this only when the user is already working inside a Next.js (or other React) codebase and explicitly wants real components, not a standalone HTML file. The default for this skill is still the single-file HTML templates — reach for this reference instead of them when that's clearly the context.

For a 3D/WebGL hero specifically (React Three Fiber), see `references/3d-hero.md` instead of building one from scratch here — it covers the `Canvas`/`Model`/lighting setup in full.

**Contents:** Dependencies · `useGSAP` hook · `Text` (split-text reveal) · `Card` (3D flip shell) · `Marquee` (velocity-aware infinite loop) · `MagneticPreviewList` (cursor-follow project list) · `FullScreenNav` (slide-in overlay + animated burger) · Sticky stacking section · Asset-gated preloader · Lenis smooth scroll.

## Dependencies

```bash
npm install gsap @gsap/react
# optional, for buttery smooth-scroll instead of native scroll:
npm install lenis
```

Tailwind is assumed for the class names below, but the GSAP logic is framework-CSS-agnostic — swap in whatever styling approach the project already uses.

## The `useGSAP` hook

`@gsap/react`'s `useGSAP` is `useEffect` purpose-built for GSAP: it auto-reverts tweens on unmount/dependency change, so timelines don't leak or double-fire across re-renders or fast refresh.

```tsx
"use client"
import { useRef } from "react"
import gsap from "gsap"
import ScrollTrigger from "gsap/ScrollTrigger"
import { useGSAP } from "@gsap/react"

const PinnedSection = () => {
  const sectionRef = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: sectionRef.current,
        start: "top top",
        end: "+=400%",
        pin: true,
        scrub: 0.5,
      },
    })
    // ...tl.to(...) calls — see references/patterns.md for the actual effects
  }, { scope: sectionRef }) // scope auto-qualifies selector-based tweens to this subtree

  return <div ref={sectionRef}>...</div>
}
```

Register `ScrollTrigger` (and `SplitText`) once near the top of a client component before using them — in the npm package these come from `gsap/ScrollTrigger` and `gsap/SplitText`, not a CDN script tag.

## Reusable `Text` component (split-text reveal)

```tsx
"use client"
import { useRef } from "react"
import SplitText from "gsap/SplitText"
import gsap from "gsap"
import { useGSAP } from "@gsap/react"

type TextProps = { children: React.ReactNode; className?: string; delay?: number }

const Text = ({ children, className = "", delay = 0 }: TextProps) => {
  const textRef = useRef<HTMLDivElement | null>(null)

  useGSAP(() => {
    const split = SplitText.create(textRef.current, { type: "words", mask: "words" })
    gsap.set(split.words, { opacity: 0, yPercent: 110 })
    gsap.to(split.words, { yPercent: 0, opacity: 1, stagger: 0.05, ease: "power2.inOut", duration: 1, delay })
  }, { dependencies: [children, delay] })

  return <div ref={textRef} className={`text-4xl ${className}`}>{children}</div>
}

export default Text
```

## Reusable `Card` component (3D flip shell)

```tsx
"use client"
import Image from "next/image"
import type { ReactNode } from "react"

type CardProps = { src: string; alt: string; children?: ReactNode; backfaceClassName?: string }

const Card = ({ src, alt, children, backfaceClassName = "" }: CardProps) => (
  <div data-card-shell className="relative aspect-2/3 w-[clamp(220px,32vw,400px)] perspective-[1000px]">
    <div data-flip-inner className="relative h-full w-full transform-3d will-change-transform">
      <div data-card-face="front" className="absolute inset-0 overflow-hidden backface-hidden transform-[translateZ(4px)]">
        <Image src={src} alt={alt} fill className="object-cover" />
      </div>
      <div data-card-face="back" className={`absolute inset-0 flex flex-col justify-center overflow-hidden p-8 backface-hidden transform-[rotateY(180deg)_translateZ(4px)] ${backfaceClassName}`}>
        {children}
      </div>
    </div>
  </div>
)

export default Card
```

The `data-card-shell` / `data-flip-inner` / `data-card-face` attributes are query hooks the pinned-section effect (in `references/patterns.md`, "3D card flip") uses via `querySelectorAll` — keep them if reusing that logic verbatim.

## Reusable `Marquee` component (velocity-aware infinite loop)

A React wrapper around the `horizontalLoop` + `Observer` technique from `references/patterns.md` ("Velocity marquee") — the loop math doesn't change, it's just driven from `useEffect` instead of a plain `<script>` block, and items are rendered from props instead of hardcoded markup:

```tsx
"use client"
import { useEffect, useRef } from "react"
import gsap from "gsap"
import { Observer } from "gsap/all"
gsap.registerPlugin(Observer)

type MarqueeProps = { items: string[]; reverse?: boolean; className?: string }

const Marquee = ({ items, reverse = false, className = "" }: MarqueeProps) => {
  const itemsRef = useRef<HTMLSpanElement[]>([])

  useEffect(() => {
    // horizontalLoop(...) — copy verbatim from references/patterns.md "Velocity marquee"
    const tl = horizontalLoop(itemsRef.current, { repeat: -1, paddingRight: 30 })
    if (reverse) tl.reverse()

    Observer.create({
      onChangeY(self) {
        const factor = ((!reverse && self.deltaY < 0) || (reverse && self.deltaY > 0) ? 1 : -1) * 2.5
        gsap.timeline({ defaults: { ease: "none" } })
          .to(tl, { timeScale: factor * 2.5, duration: 0.2, overwrite: true })
          .to(tl, { timeScale: factor / 2.5, duration: 1 }, "+=0.3")
      },
    })
    return () => tl.kill()
  }, [items, reverse])

  return (
    <div className={`overflow-hidden whitespace-nowrap flex items-center h-20 ${className}`}>
      <div className="flex">
        {items.map((text, i) => (
          <span key={i} ref={(el) => { if (el) itemsRef.current[i] = el }} className="px-16 text-4xl uppercase">
            {text}
          </span>
        ))}
      </div>
    </div>
  )
}

export default Marquee
```

Render the `items` array duplicated 2–3x in the parent (`[...words, ...words, ...words]`) the same way the vanilla pattern needs repeated markup — the loop math assumes there's enough content width to fill several screens before it needs to wrap.

## Reusable `MagneticPreviewList` component (cursor-follow project list)

React version of "Magnetic cursor preview" from `references/patterns.md` — `gsap.quickTo` for the follow, `useState` for which row is active instead of manually toggling classes:

```tsx
"use client"
import { useRef, useState } from "react"
import { useGSAP } from "@gsap/react"
import gsap from "gsap"
import Image from "next/image"

type Item = { id: string; title: string; image: string }

const MagneticPreviewList = ({ items }: { items: Item[] }) => {
  const previewRef = useRef<HTMLDivElement>(null)
  const moveX = useRef<ReturnType<typeof gsap.quickTo>>()
  const moveY = useRef<ReturnType<typeof gsap.quickTo>>()
  const [active, setActive] = useState<string | null>(null)

  useGSAP(() => {
    moveX.current = gsap.quickTo(previewRef.current, "x", { duration: 0.6, ease: "power3.out" })
    moveY.current = gsap.quickTo(previewRef.current, "y", { duration: 0.8, ease: "power3.out" })
  }, [])

  const handleMouseMove = (e: React.MouseEvent) => {
    if (window.innerWidth < 768) return // no cursor on touch — rows fall back to inline images instead
    moveX.current?.(e.clientX + 24)
    moveY.current?.(e.clientY + 24)
  }

  const activeItem = items.find((it) => it.id === active)

  return (
    <div onMouseMove={handleMouseMove} className="relative">
      {items.map((item) => (
        <div
          key={item.id}
          className="group relative flex justify-between border-t py-6 cursor-pointer"
          onMouseEnter={() => setActive(item.id)}
          onMouseLeave={() => setActive(null)}
        >
          <h3 className="text-3xl">{item.title}</h3>
        </div>
      ))}
      <div
        ref={previewRef}
        className="fixed top-0 left-0 w-80 pointer-events-none z-50 overflow-hidden transition-opacity duration-300"
        style={{ opacity: activeItem ? 1 : 0 }}
      >
        {activeItem && <Image src={activeItem.image} alt={activeItem.title} width={320} height={400} className="object-cover" />}
      </div>
    </div>
  )
}

export default MagneticPreviewList
```

## Reusable `FullScreenNav` component (slide-in overlay + animated burger)

React version of "Slide-in nav overlay" — the two-timeline (panel + icon) structure stays identical, just driven by a `useGSAP` + `useState` toggle instead of a raw click listener:

```tsx
"use client"
import { useRef, useState } from "react"
import { useGSAP } from "@gsap/react"
import gsap from "gsap"

const links = ["home", "work", "about", "contact"]

const FullScreenNav = () => {
  const panelRef = useRef<HTMLDivElement>(null)
  const linksRef = useRef<HTMLAnchorElement[]>([])
  const topBar = useRef<HTMLSpanElement>(null)
  const bottomBar = useRef<HTMLSpanElement>(null)
  const panelTl = useRef<gsap.core.Timeline>()
  const iconTl = useRef<gsap.core.Timeline>()
  const [open, setOpen] = useState(false)

  useGSAP(() => {
    gsap.set(panelRef.current, { xPercent: 100 })
    gsap.set(linksRef.current, { autoAlpha: 0, x: -20 })

    panelTl.current = gsap.timeline({ paused: true })
      .to(panelRef.current, { xPercent: 0, duration: 0.8, ease: "power3.out" })
      .to(linksRef.current, { autoAlpha: 1, x: 0, stagger: 0.08, duration: 0.5, ease: "power2.out" }, "<")

    iconTl.current = gsap.timeline({ paused: true })
      .to(topBar.current, { rotate: 45, y: 4, duration: 0.3, ease: "power2.inOut" })
      .to(bottomBar.current, { rotate: -45, y: -4, duration: 0.3, ease: "power2.inOut" }, "<")
  }, [])

  const toggle = () => {
    open ? (panelTl.current?.reverse(), iconTl.current?.reverse()) : (panelTl.current?.play(), iconTl.current?.play())
    setOpen(!open)
  }

  return (
    <>
      <div ref={panelRef} className="fixed top-0 right-0 h-full w-full max-w-[480px] bg-black text-white z-40 flex flex-col justify-center gap-8 px-12">
        {links.map((link, i) => (
          <a key={link} ref={(el) => { if (el) linksRef.current[i] = el }} href={`#${link}`} className="text-6xl uppercase">
            {link}
          </a>
        ))}
      </div>
      <button onClick={toggle} className="fixed top-6 right-6 z-50 w-11 h-11 flex flex-col items-center justify-center gap-1.5">
        <span ref={topBar} className="block w-6 h-0.5 bg-white" />
        <span ref={bottomBar} className="block w-6 h-0.5 bg-white" />
      </button>
    </>
  )
}

export default FullScreenNav
```

## Sticky stacking section

The "Sticky stacking cards" pattern from `references/patterns.md` needs no GSAP at all for the stacking itself — just `position: sticky` with a per-index `top` offset, plus an entrance tween:

```tsx
"use client"
import { useGSAP } from "@gsap/react"
import gsap from "gsap"

type Card = { title: string; description: string }

const StackingCards = ({ cards }: { cards: Card[] }) => {
  useGSAP(() => {
    gsap.utils.toArray<HTMLElement>(".stack-card").forEach((card) => {
      gsap.from(card, { y: 120, opacity: 0, duration: 1, ease: "circ.out", scrollTrigger: { trigger: card, start: "top 85%" } })
    })
  }, [cards])

  return (
    <>
      {cards.map((card, i) => (
        <div key={i} className="stack-card sticky border-t border-white/20 p-12" style={{ top: `calc(8vh + ${i * 2.5}rem)` }}>
          <h2 className="text-4xl">{card.title}</h2>
          <p className="mt-4 text-white/60">{card.description}</p>
        </div>
      ))}
    </>
  )
}

export default StackingCards
```

## Asset-gated preloader (React)

Gate the whole app behind real load progress instead of a fixed timer — for a 3D scene, `@react-three/drei`'s `useProgress` already tracks this (see `references/3d-hero.md`); for an image/video-heavy page without 3D, track `<img>`/`<video>` elements the same way the vanilla pattern in `references/patterns.md` ("Asset-gated preloader") does, just wired through `useEffect` + `useState` instead of plain DOM listeners.

## Optional: Lenis smooth scroll

Wrap the app root so native scroll feels smoother, driven by GSAP's own ticker so it stays in sync with ScrollTrigger:

```tsx
"use client"
import gsap from "gsap"
import { ReactLenis, type LenisRef } from "lenis/react"
import { useEffect, useRef, type ReactNode } from "react"

export default function SmoothScroller({ children }: { children: ReactNode }) {
  const lenisRef = useRef<LenisRef | null>(null)

  useEffect(() => {
    function update(time: number) { lenisRef.current?.lenis?.raf(time * 1000) }
    gsap.ticker.add(update)
    return () => gsap.ticker.remove(update)
  }, [])

  return <ReactLenis ref={lenisRef} root options={{ autoRaf: false }}>{children}</ReactLenis>
}
```

Mount this once in the root layout, wrapping `children`. Skip it entirely if the project doesn't already want a dependency on a smooth-scroll library — native scroll + ScrollTrigger works fine on its own, this is purely a feel upgrade.
