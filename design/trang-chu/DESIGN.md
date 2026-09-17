---
name: Retail Shift Attendance
colors:
  surface: '#f8f9ff'
  surface-dim: '#cbdbf5'
  surface-bright: '#f8f9ff'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#eff4ff'
  surface-container: '#e5eeff'
  surface-container-high: '#dce9ff'
  surface-container-highest: '#d3e4fe'
  on-surface: '#0b1c30'
  on-surface-variant: '#3d4a3d'
  inverse-surface: '#213145'
  inverse-on-surface: '#eaf1ff'
  outline: '#6d7b6c'
  outline-variant: '#bccbb9'
  surface-tint: '#006e2f'
  primary: '#006e2f'
  on-primary: '#ffffff'
  primary-container: '#22c55e'
  on-primary-container: '#004b1e'
  inverse-primary: '#4ae176'
  secondary: '#9d4300'
  on-secondary: '#ffffff'
  secondary-container: '#fd761a'
  on-secondary-container: '#5c2400'
  tertiary: '#565e74'
  on-tertiary: '#ffffff'
  tertiary-container: '#a4abc4'
  on-tertiary-container: '#383f54'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#6bff8f'
  primary-fixed-dim: '#4ae176'
  on-primary-fixed: '#002109'
  on-primary-fixed-variant: '#005321'
  secondary-fixed: '#ffdbca'
  secondary-fixed-dim: '#ffb690'
  on-secondary-fixed: '#341100'
  on-secondary-fixed-variant: '#783200'
  tertiary-fixed: '#dae2fd'
  tertiary-fixed-dim: '#bec6e0'
  on-tertiary-fixed: '#131b2e'
  on-tertiary-fixed-variant: '#3f465c'
  background: '#f8f9ff'
  on-background: '#0b1c30'
  surface-variant: '#d3e4fe'
typography:
  headline-lg:
    fontFamily: Inter
    fontSize: 28px
    fontWeight: '700'
    lineHeight: 34px
  headline-md:
    fontFamily: Inter
    fontSize: 22px
    fontWeight: '600'
    lineHeight: 28px
  headline-sm:
    fontFamily: Inter
    fontSize: 18px
    fontWeight: '600'
    lineHeight: 24px
  body-lg:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  body-md:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  body-sm:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '400'
    lineHeight: 16px
  label-lg:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '600'
    lineHeight: 20px
  label-md:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
  label-sm:
    fontFamily: Inter
    fontSize: 11px
    fontWeight: '500'
    lineHeight: 14px
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  gutter: 1rem
  margin: 1rem
  space-xs: 0.25rem
  space-sm: 0.5rem
  space-md: 1rem
  space-lg: 1.5rem
  space-xl: 2rem
---

## Brand & Style

This design system is tailored for fast-paced retail and store environments where frontline employees need frictionless, immediate interactions on mobile devices. The visual language is rooted in **Modern Functional Minimalism**: calm, structured, high-clarity, and distraction-free.

The aesthetic reduces cognitive load during shift transitions, opening, and closing rushes. It prioritizes instantaneous confirmation of actions, high-contrast readability under bright store lighting or quick glances, and clean tactile affordances that inspire reliability and trust between workers and management.

## Colors

The palette establishes an unambiguous semantic hierarchy optimized for status recognition:

- **Primary Action (Check-in)**: `#22C55E` (Emerald Green) signals start of shift, successful verification, geofence alignment, and active duty.
- **Secondary Action (Check-out / Break)**: `#F97316` (Vibrant Orange) distinguishes departure, meal breaks, and shift closure without using error-connoting red.
- **Surface Hierarchy**:
  - App Canvas / Ground: `#F8FAFC` (Slate 50) and `#F1F5F9` (Slate 100) for inset containers.
  - Interactive Surfaces / Elevated Containers: `#FFFFFF` (Pure White).
  - Subtle Dividing Line: `#E2E8F0` (Slate 200).
- **Text & Content Hierarchy**:
  - Primary Text / Key Numbers: `#0F172A` (Slate 900) for uncompromised legibility.
  - Secondary Text / Timestamps / Labels: `#64748B` (Slate 500).
  - Tertiary / Placeholder / Muted Meta: `#94A3B8` (Slate 400).

## Typography

Inter serves across all typography levels for its neutral grotesque proportions, open counters, and high legibility on standard and high-density mobile viewports.

- Numeric figures for live shift clocks, working hours, and time logs must apply tabular numbers (`font-variant-numeric: tabular-nums`) to prevent horizontal jitter during second-by-second updates.
- Keep letter spacing tight (`-0.01em` to `-0.02em`) on bold headings and strictly standard (`0`) on body copy to retain rapid visual parsing for store staff.

## Layout & Spacing

The layout is built for fluid single-column mobile viewport usage with a strict 4px/8px incremental rhythm:

- **Mobile Viewports (< 640px)**: 16px (`1rem`) outer canvas margins and 16px grid gutters. Primary action zones are anchored within thumb-reach zones at the lower third of the display.
- **Tablet / In-store Terminal (640px - 1024px)**: Centers content within a max-width container of 540px for single-user check-in kiosk modes, or splits into a two-column balance for shift schedule and live timecard views.
- Vertical density is compact yet breathable: critical check-in triggers preserve a minimum 52px touch target height.

## Elevation & Depth

Visual hierarchy uses a dual-layer approach combining subtle border outlines with diffused low-intensity shadows to prevent visual noise:

- **Ground Level (Elevation 0)**: Background canvases sit on `#F8FAFC`.
- **Card Tier (Elevation 1)**: Pure white `#FFFFFF` surfaces with a crisp 1px stroke of `#E2E8F0` and an ultra-soft ambient shadow: `0 1px 3px 0 rgba(15, 23, 42, 0.04), 0 1px 2px -1px rgba(15, 23, 42, 0.03)`.
- **Floating Modals & Pinned Action Bars (Elevation 2)**: Persistent bottom navigation and swipe-up sheets use `0 10px 15px -3px rgba(15, 23, 42, 0.08), 0 4px 6px -4px rgba(15, 23, 42, 0.03)` with border-t of `#E2E8F0`.
- Dark or dramatic heavy dropshadows are forbidden; boundaries are defined primarily by surface contrast and fine outlines.

## Shapes

The interface embraces balanced, friendly radii:
- Standard cards, shift modules, and grouped list items: `12px` to `16px` (`rounded-lg` / `rounded-xl`).
- Interactive form controls and text inputs: `12px`.
- Status pills, shift state chips, and avatar rings: Full circular/pill geometry (`rounded-full` / `9999px`).
- Icons utilize a consistent 1.5px to 2px stroke width with rounded join and cap properties.

## Components

### Action Triggers (Check-in / Check-out Buttons)
- **Check-in CTA**: Full-width or large prominent button, solid `#22C55E` background, white label text, minimum 54px height, 14px border radius. Focus/active state scales down slightly (`0.98`) with subtle green glow.
- **Check-out CTA**: Solid `#F97316` background, white label text, identical geometry and state transitions.
- **Secondary / Auxiliary**: Outlined `#E2E8F0` surface with `#0F172A` text and `#FFFFFF` background.

### Cards
- Pure `#FFFFFF` background, `1px solid #E2E8F0`, `16px` radius, padded with 16px (`space-md`). Used for active shift overview, today's schedule, and store branch verification.

### Pill Badges & Status Chips
- Height 24px–28px, rounded-full.
- **On-time / Present**: Background `#DCFCE7` (Emerald 100) with `#15803D` (Emerald 700) text.
- **Late / Overtime**: Background `#FFEDD5` (Orange 100) with `#C2410C` (Orange 700) text.
- **Off-duty / Neutral**: Background `#F1F5F9` (Slate 100) with `#475569` (Slate 600) text.

### Attendance History & List Items
- Clean divided rows without full-width bounding boxes or enclosed inside Elevation 1 cards.
- Left-aligned icon/avatar indicating shift type, middle block containing role and branch name, right-aligned tabular timestamp and status badge.

### Inputs & Verification Selectors
- Branch selection dropdowns and note fields: Background `#FFFFFF`, border `1px solid #E2E8F0`, 48px height, 12px corner radius. Focused state shifts border to `#0F172A` with no harsh outline ring.

### Geofence / Location Status Indicator
- Compact header component containing GPS ping icon, store branch name, and micro-badge: Green pulsing dot when within branch boundaries, amber dot when searching for signal.