# Copilot Instructions for Penguin Book Website

## Project Overview
This is a one-page microsite for the satirical children's book "Why Daddy's Law Firm Works with the Nice Oil Men". The site should be clean, simple, and professional - like a product page or book microsite.

## Design Principles
- **Minimalist approach**: Only include what's necessary to convey the message
- **No satire markers**: The content speaks for itself; don't add meta-commentary
- **No emojis**: Ever
- **Responsible design**: Mobile-first, accessible
- **Clean typography**: Use Powell font throughout

## Color Palette
- Blue: `#8ED3E6`
- Teal: `#237F8D`
- Red: `#D94717`
- Yellow: `#FFC529`
- Grey: `#656D70`
- Off-white: `#EDE9E8`
- White: `#FFFFFF`

Primary color scheme is blue/white with teal accents.

## Typography
- Font: "powell" from Adobe Typekit
- Font weights: 400 (normal) and 400 italic
- Loaded via: `https://use.typekit.net/izw6lkb.css`

## Tech Stack
- **Astro 5.x**: Primary framework
- **Tailwind CSS 4**: Styling (the new version)
- **Svelte**: For interactive components (carousel, quote rotation)
- **Astro Image**: For optimized images
- No React unless absolutely necessary

## Key Components
1. **RotatingQuote.svelte**: Cycles between testimonial quotes
2. **BookCarousel.svelte**: Displays book pages with navigation (first page solo, then double spreads, last page solo if odd)

## Content Guidelines
- Draw copy from the provided book text
- Maintain the satirical tone without explaining it
- Keep messaging minimal and direct
- No overwriting or overspecifying

## Images
- Book pages: `src/assets/book/why-daddys-law-firm-works-with-the-nice-oil-men-01.png` through `32.png`
- Logo: `src/assets/SP-icon-logo.png`
- Background: `src/assets/illos/plain-blue-background.jpg`

## Deployment
- Target: GitHub Pages at `seriouspeople.co/penguin`
- Base path: `/penguin`
- Automated via GitHub Actions

## Analytics
- Umami tracking with ID: `d0e3b86b-b601-4b5f-8dcd-c6d5d7d47558`

## Future Enhancements
When developing new features:
- Keep the same minimal aesthetic
- Ensure mobile responsiveness
- Test carousel/reader functionality thoroughly
- Maintain performance (optimize images, lazy load)
