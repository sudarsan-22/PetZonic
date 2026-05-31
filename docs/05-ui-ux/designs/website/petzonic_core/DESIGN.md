---
name: PetZonic Core
colors:
  surface: '#f9f9fc'
  surface-dim: '#dadadc'
  surface-bright: '#f9f9fc'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f3f3f6'
  surface-container: '#eeeef0'
  surface-container-high: '#e8e8ea'
  surface-container-highest: '#e2e2e5'
  on-surface: '#1a1c1e'
  on-surface-variant: '#564334'
  inverse-surface: '#2f3133'
  inverse-on-surface: '#f0f0f3'
  outline: '#897362'
  outline-variant: '#ddc1ae'
  surface-tint: '#904d00'
  primary: '#904d00'
  on-primary: '#ffffff'
  primary-container: '#ff8c00'
  on-primary-container: '#623200'
  inverse-primary: '#ffb77d'
  secondary: '#006a6a'
  on-secondary: '#ffffff'
  secondary-container: '#90efef'
  on-secondary-container: '#006e6e'
  tertiary: '#645d53'
  on-tertiary: '#ffffff'
  tertiary-container: '#b2a99c'
  on-tertiary-container: '#443e34'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#ffdcc3'
  primary-fixed-dim: '#ffb77d'
  on-primary-fixed: '#2f1500'
  on-primary-fixed-variant: '#6e3900'
  secondary-fixed: '#93f2f2'
  secondary-fixed-dim: '#76d6d5'
  on-secondary-fixed: '#002020'
  on-secondary-fixed-variant: '#004f4f'
  tertiary-fixed: '#ece1d3'
  tertiary-fixed-dim: '#cfc5b8'
  on-tertiary-fixed: '#201b13'
  on-tertiary-fixed-variant: '#4c463c'
  background: '#f9f9fc'
  on-background: '#1a1c1e'
  surface-variant: '#e2e2e5'
typography:
  headline-xl:
    fontFamily: Montserrat
    fontSize: 40px
    fontWeight: '700'
    lineHeight: 48px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Montserrat
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Montserrat
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
  body-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 28px
  body-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  label-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 14px
    fontWeight: '600'
    lineHeight: 20px
    letterSpacing: 0.01em
  label-sm:
    fontFamily: Plus Jakarta Sans
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
  headline-xl-mobile:
    fontFamily: Montserrat
    fontSize: 30px
    fontWeight: '700'
    lineHeight: 36px
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
  margin-desktop: 48px
---

## Brand & Style

The design system is engineered to evoke a sense of joyful reliability. It targets pet parents in the Indian market, balancing the vibrant energy of a growing pet culture with the professional trust required for high-value transactions and animal welfare.

The aesthetic follows a **Modern / Tactile** approach. It utilizes soft, organic depth and high-saturation accents to create a space that feels approachable yet high-end. The UI should feel "plump" and friendly—avoiding sharp edges or aggressive minimalism in favor of a welcoming, community-driven atmosphere. Every interaction should reinforce a sense of warmth and vitality.

## Colors

The palette is anchored by **Vibrant Orange**, used strategically for primary actions and energetic brand moments. **Teal** serves as the stabilizing force, applied to utility elements, trust indicators (badges, verifications), and secondary navigation to ensure a professional grounding.

- **Primary (Orange):** High-visibility CTA, price points, and active states.
- **Secondary (Teal):** Verification icons, links, and success states.
- **Tertiary (Peach/Cream):** Subtitle backgrounds, light containers, and soft separation.
- **Surface:** A crisp White (#FFFFFF) base with ultra-light Neutral Gray (#F8F9FA) for subtle background layering.
- **Text:** Deep charcoal (#1A1C1E) for high legibility, avoiding pure black to maintain the soft brand character.

## Typography

This design system uses a dual-font strategy to balance character with readability. **Montserrat** provides a bold, geometric presence for headlines, echoing the circularity of the "PetZonic" spirit. **Plus Jakarta Sans** is used for body text and labels; its soft curves and high x-height ensure exceptional legibility on mobile screens during long browsing sessions.

Maintain generous line heights to prevent text-heavy descriptions from feeling cramped. Titles should always use a tighter letter spacing to maintain a "punchy" editorial look.

## Layout & Spacing

The layout employs a **Fluid-Fixed Hybrid** model. On desktop, content is constrained to a 1280px central container using a 12-column grid. On mobile, the system transitions to a single-column flow with generous 16px side margins to ensure touch targets are easily accessible.

The spacing rhythm is based on an **8px linear scale**. Use larger gaps (48px+) between distinct sections to allow the UI to "breathe," emphasizing the premium nature of the marketplace. Horizontal scrolling patterns are encouraged for "Category" or "Trending" chips on mobile to maximize vertical real estate.

## Elevation & Depth

Hierarchy is established through **Ambient Shadows**. This design system avoids harsh borders in favor of soft, multi-layered shadows that make components feel as though they are floating slightly above the canvas.

- **Level 1 (Surface):** Default white background.
- **Level 2 (Cards/Tiles):** 12% opacity shadow, 16px blur, 4px vertical offset, tinted with a hint of Teal (#008080) to maintain color harmony.
- **Level 3 (Modals/Popovers):** 18% opacity shadow, 32px blur, 12px vertical offset.

Interactive elements should "lift" (increase shadow depth) on hover to provide clear tactile feedback.

## Shapes

The shape language is defined by extreme **2XL Roundedness**. 

- **Cards & Primary Containers:** Use `rounded-xl` (1.5rem/24px) to create a friendly, safe appearance.
- **Buttons & Inputs:** Use `rounded-lg` (1rem/16px) for a substantial, "squishy" feel that invites interaction.
- **Badges & Tags:** Use fully pill-shaped (3rem/48px) styling to differentiate them from functional UI buttons.

Iconography should follow a "Line-and-Fill" style with rounded terminals and thick strokes (2px) to match the weight of the typography.

## Components

### Modern E-commerce Cards
Cards feature a top-aligned image with a subtle zoom effect on hover. The bottom section contains the title in `headline-md`, price in bold Primary Orange, and a Teal "Verified" badge. The card container must have a soft `level 2` elevation.

### Clear CTAs
Primary buttons are solid Orange with white text. They should be tall (min-height: 48px) with significant horizontal padding. Secondary buttons use a Teal outline with a 2px stroke.

### Wide Search Bar
The search bar is a signature element: a wide, `rounded-lg` container with a light gray stroke and a `level 2` shadow. It features a Primary Orange "Search" icon or button on the right edge to draw the eye immediately.

### Soft-Shadow Tiles
Used for category navigation (e.g., "Dogs", "Cats", "Supplies"). These are square-aspect tiles with a centered icon and label, utilizing the Tertiary background color to differentiate them from product cards.

### Input Fields
Inputs use a light background (#F8F9FA) rather than a white one, making the `rounded-lg` container clearly visible against the white page surface. Focus states use a 2px Teal glow.