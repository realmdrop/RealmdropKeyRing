# KeyRing by Realmdrop — Project Summary

## What is KeyRing?

KeyRing is a **local-first secrets manager and brand kit vault** built for developers and designers. It stores passwords and brand kits entirely on the user's device — nothing is sent to a server. All data is encrypted with the user's master password.

## Who is it for?

- **Developers** who need a lightweight, local password manager they can trust (no cloud dependency, no subscription lock-in)
- **Designers** who want to store and organize brand kits (colors, fonts, logos, assets) alongside their credentials in one secure place

## Core Features

- **Local-only storage** — all data stays on the user's device, encrypted at rest with their master password
- **Password vault** — store, organize, and retrieve credentials securely
- **Brand kit vault** — store brand kits (color palettes, typography, logos, design tokens) alongside secrets
- **Import & export** — bring in existing credentials or brand kits; export them for portability
- **Local backups** — create encrypted backup files stored on the user's machine
- **Sharing** — share individual entries or brand kits with collaborators
- **PWA (Progressive Web App)** — installable on desktop and mobile, works offline
- **No account required** — no sign-up, no cloud sync, no third-party dependency

## Brand Identity

- **Product name:** KeyRing
- **Parent brand:** Realmdrop (tagline: "by Realmdrop")
- **Logo:** A key-ring icon — a white circle (the ring) with a mint-green center dot, and two key shafts extending from the ring (one straight down, one at 45°). Set on a deep violet (#2E1065) rounded-rectangle background.
- **Primary colors:**
  - Deep Violet: `#2E1065` (backgrounds, logo tile)
  - Dark Violet: `#12062E` (dark-mode background)
  - Mint / Accent: `#00FFB2` (highlights, the "Ring" in the dark-mode wordmark)
  - White: `#FFFFFF` (text on dark, logo strokes)
- **Typography:** Inter (medium weight, -0.5 letter spacing for the wordmark)
- **Dark mode wordmark:** "Key" in white, "Ring" in mint green, "BY REALMDROP" subtitle in white at reduced opacity
- **Light mode wordmark:** Both "Key" and "Ring" in deep violet, "BY REALMDROP" subtitle in deep violet at reduced opacity

## Positioning / What Makes It Different

- **Zero-cloud architecture** — unlike 1Password, Bitwarden, or LastPass, nothing ever leaves the device. No servers to breach.
- **Built for creatives, not just IT** — the brand kit vault is a unique feature that makes it useful beyond passwords, specifically targeting the design workflow.
- **No subscription** — local tool, no recurring cost tied to cloud infrastructure.
- **PWA-based** — runs in the browser, installable, works offline. No native app install or Electron overhead.

## Technical Notes

- Built as a single-page PWA (HTML/JS/CSS)
- Service worker for offline support (cache versioned as `keyring-v3`)
- Manifest configured for standalone display
- Export filenames follow the pattern `keyring-export-[timestamp]`

## Branding Assets Available

- `keyring-logo.svg` — logo mark on violet background
- `keyring-logo-transparent.svg` — logo mark, transparent background
- `keyring-wordmark-light.svg` — full wordmark for light backgrounds
- `keyring-wordmark-dark.svg` — full wordmark for dark backgrounds

---

*Use this summary to design a landing page for KeyRing. The branding SVGs listed above are available as source files.*
