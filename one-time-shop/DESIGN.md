---
name: Ethos Sustainable Digital
colors:
  surface: '#f8faf6'
  surface-dim: '#d9dad7'
  surface-bright: '#f8faf6'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f2f4f1'
  surface-container: '#edeeeb'
  surface-container-high: '#e7e9e5'
  surface-container-highest: '#e1e3e0'
  on-surface: '#191c1b'
  on-surface-variant: '#404944'
  inverse-surface: '#2e312f'
  inverse-on-surface: '#f0f1ee'
  outline: '#707974'
  outline-variant: '#bfc9c3'
  surface-tint: '#306855'
  primary: '#003426'
  on-primary: '#ffffff'
  primary-container: '#0f4c3a'
  on-primary-container: '#82bba4'
  inverse-primary: '#99d3ba'
  secondary: '#084edb'
  on-secondary: '#ffffff'
  secondary-container: '#376af4'
  on-secondary-container: '#fffbff'
  tertiary: '#4d1f1b'
  on-tertiary: '#ffffff'
  tertiary-container: '#68342f'
  on-tertiary-container: '#e69e96'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#b4efd6'
  primary-fixed-dim: '#99d3ba'
  on-primary-fixed: '#002117'
  on-primary-fixed-variant: '#15503e'
  secondary-fixed: '#dce1ff'
  secondary-fixed-dim: '#b5c4ff'
  on-secondary-fixed: '#00164e'
  on-secondary-fixed-variant: '#003cae'
  tertiary-fixed: '#ffdad6'
  tertiary-fixed-dim: '#ffb4ab'
  on-tertiary-fixed: '#370e0b'
  on-tertiary-fixed-variant: '#6c3832'
  background: '#f8faf6'
  on-background: '#191c1b'
  surface-variant: '#e1e3e0'
  eco-green-light: '#E8F5E9'
  trust-blue-dark: '#0A2540'
  sustainable-gold: '#C5A059'
  surface-gray: '#F8F9FA'
typography:
  display-lg:
    fontFamily: Manrope
    fontSize: 48px
    fontWeight: '800'
    lineHeight: 56px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Manrope
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
  headline-lg-mobile:
    fontFamily: Manrope
    fontSize: 28px
    fontWeight: '700'
    lineHeight: 36px
  title-md:
    fontFamily: Manrope
    fontSize: 20px
    fontWeight: '600'
    lineHeight: 28px
  body-lg:
    fontFamily: Inter
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 28px
  body-md:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  label-sm:
    fontFamily: JetBrains Mono
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
    letterSpacing: 0.05em
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  base: 8px
  container-max: 1280px
  gutter: 24px
  margin-mobile: 16px
  margin-desktop: 40px
---

## Brand & Style

The design system is built on the principles of **digital sustainability** and **radical transparency**. It positions software not as a temporary service, but as a durable asset. The visual language balances the high-tech nature of software with the organic, grounded values of environmental stewardship.

The chosen style is **Corporate Modern with a Minimalist infusion**. It avoids the "disposable" feel of consumer apps by using structured layouts, generous whitespace, and a high-fidelity finish. The interface is intentionally calm to reduce cognitive load, reflecting an ethical approach to user experience that respects the user's time and attention.

**Key Brand Pillars:**
- **Longevity:** UI elements feel stable and permanent rather than trendy or fleeting.
- **Clarity:** Information architecture prioritizes pricing and ownership rights over marketing fluff.
- **Trust:** A professional, polished execution that reassures users about their long-term investment.

## Colors

The palette is anchored by **Deep Forest Green** (`primary`), symbolizing sustainability and growth, and **Strategic Blue** (`secondary`), which reinforces the marketplace's professional reliability.

- **Primary:** Used for the most critical actions and the "Sustainable Software Badge." It represents the "One-Time" commitment.
- **Secondary:** Used for interactive elements, navigation, and links to distinguish utility from brand identity.
- **Neutral:** A range of cool grays and "Paper White" backgrounds to ensure content remains the hero.
- **Sustainable Gold:** Reserved exclusively for high-tier sustainability scores and "verified durable" status badges.

The color mode is primarily **Light**, utilizing high-contrast text to maximize legibility and minimize the energy required for reading.

## Typography

This design system employs a tri-font strategy to balance modernity with technical precision:

1.  **Manrope (Headlines):** A geometric sans-serif that feels contemporary yet approachable. It is used for all display and heading levels to provide a strong brand voice.
2.  **Inter (Body):** Chosen for its exceptional legibility at all sizes. It handles the bulk of the marketplace's data and descriptions.
3.  **JetBrains Mono (Labels/Technical):** Used for pricing, "Sustainable Scores," and versioning data. The monospaced nature evokes the "software" aspect of the marketplace and suggests transparency and "under-the-hood" honesty.

**Scaling:** On mobile devices, display and headline sizes should reduce by approximately 15-20% to maintain comfortable reading gravity.

## Layout & Spacing

The layout follows a **Fixed-Fluid Hybrid Grid**. On desktop, content is centered within a 1280px container to ensure readability isn't compromised on ultra-wide monitors.

- **Grid:** A 12-column system for desktop, 8-column for tablet, and 4-column for mobile.
- **Rhythm:** An 8px linear scale is used for all padding and margins (`base`). 
- **Reflow:** On mobile, side-by-side comparison tables must reflow into vertical "attribute cards" to maintain the integrity of the "Subscription Cost Calculator" data.
- **Whitespace:** Emphasize vertical "breathing room" between sections (64px+) to prevent the marketplace from feeling cluttered, reinforcing the "quality over quantity" brand ethos.

## Elevation & Depth

This design system uses **Tonal Layering** instead of heavy shadows to communicate depth, reflecting a "lightweight" and eco-conscious digital footprint.

- **Level 0 (Background):** The base canvas (`#FFFFFF` or `#F8F9FA`).
- **Level 1 (Cards):** Subtle 1px borders in a soft neutral gray. No shadow.
- **Level 2 (Interactive/Hover):** A very soft, tinted ambient shadow (Primary Color at 5% opacity, 12px blur) to indicate lift without visual noise.
- **Depth Cues:** Background blurs are used sparingly behind sticky navigation bars to maintain context while scrolling without requiring high-contrast separators.

## Shapes

The shape language is **Refined and Structured**.

- **Standard Elements:** Buttons, input fields, and cards use a **0.5rem (8px)** radius. This strikes a balance between the "friendliness" of rounded corners and the "professionalism" of sharp ones.
- **Badges:** The "Sustainable Software Badge" uses a **Pill-shape** to distinguish it from interactive buttons.
- **Icons:** Use a consistent 2px stroke weight with slightly rounded terminals to match the typography.

## Components

### Buttons
Primary buttons use the Forest Green background with white text. Secondary buttons use a ghost style with the Strategic Blue border. Buttons should never use "loud" gradients; solid fills only to reflect transparency.

### Sustainable Software Badge
A signature component. It features a JetBrains Mono label and a small leaf or "infinite" icon. The color shifts from Gray (neutral) to Green (high score) to Gold (exemplary).

### Pricing Transparency Card
A specialized card component that displays the "One-Time Price" in a large display font, immediately followed by a "Yearly Savings" label in the secondary color. It must include a "No Hidden Fees" checklist.

### Subscription Cost Calculator
A slider-based component with a dual-axis chart. As users slide the "Years of Use," the UI should visually demonstrate the widening gap between the flat cost of a one-time purchase and the compounding cost of a subscription.

### Input Fields
Clean, 1px bordered boxes. On focus, the border transitions to Strategic Blue with a soft 2px outer glow. Labels always remain visible (no floating labels that disappear) to maintain accessibility.

### List Items
Used for software features. Instead of standard bullets, use a "check-circle" in the primary green to reinforce the positive, ethical nature of the software choice.