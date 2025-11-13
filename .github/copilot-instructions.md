# Didara Sales Page - Copilot Instructions

## Project Overview
Didara is a high-conversion e-commerce sales funnel for Nigerian market products (currently featuring Dell Latitude 3189 laptops). It's a **three-page, single-file HTML/CSS/JS application** with no build system—all code is self-contained in HTML files.

### Architecture
- **index.html**: Main product sales page with sticky headers, product showcase, carousel, multi-form checkout flow
- **managers-office.html**: Secondary dashboard/hub for related products and customer support  
- **thankyou-page.html**: Post-purchase confirmation with redirect options

## Critical Patterns & Conventions

### 1. **Design System (CSS Custom Properties)**
All pages use **CSS variables** for theming and styling consistency:

```css
:root {
  --primary: 12 88% 59%;      /* Orange/Red - Main CTA color */
  --secondary: 174 72% 46%;   /* Teal - Secondary accent */
  --accent: 45 100% 51%;      /* Yellow/Gold - Highlights */
  --background: 30 40% 97%;   /* Light cream background */
  --card: 0 0% 100%;          /* White cards */
}
```

- Colors use **HSL format with CSS variables** (not Tailwind)—`hsl(var(--primary))` for all components
- **Dark mode** duplicates the CSS variables in a `.dark` class applied to `<body>`
- Theme toggle stores preference in `localStorage.setItem('theme', isDark ? 'dark' : 'light')`

### 2. **Form Handling (Formspree Integration)**
Two primary forms handle customer data:

| Form | Location | Purpose | Submission |
|------|----------|---------|-----------|
| **Form 1** | Modal (opens from CTAs) | Quick order with quantity selection | Formspree → `thankyou-page.html` |
| **Form 2** | Final checkout section (B7) | Detailed delivery address & confirmation | Formspree → `thankyou-page.html` |

**Key behavior:**
- Both forms POST to `https://formspree.io/f/meovoqaq`
- Hidden fields auto-populate: `Quantity_Selected_ID`, `Price_Total`, `Source` (tracks which button opened the form)
- Form 1 includes inline quantity selection; Form 2 requires pre-selection from main page
- Validation: if no quantity is selected before Form 2 submission, scroll user to quantity section (B7) and highlight

### 3. **State Management (Global Variables)**
- `selectedQuantityId`: Tracks which quantity pack the user selected (indices `qty1`, `qty2`, `qty3`)
- `OFFERS` array: Defines pricing tiers with savings calculations
- `ANNOUNCEMENTS` array: Rotates social proof messages every 10 seconds
- No external state library—all managed in `<script>` with window-scoped variables

### 4. **Sticky Header Behavior**
**index.html** features a sophisticated two-layer sticky header:

1. **Announcement Bar** (top-most, z-index 50): Rotates testimonials/alerts every 10 seconds
2. **CTA Bar** (below announcement): 
   - Countdown timer (always visible, resets hourly)
   - Order Now button **animates in/out on scroll**—hides on down-scroll, pops in on up-scroll
   - Quick Order button (secondary)
   - Theme toggle

**Scroll logic** in `handleScroll()`:
- Down-scroll: CTA bar slides out (`translateY(-100%)`)
- Up-scroll + threshold check: CTA button gains `cta-btn-visible` class (restores opacity/transform)

### 5. **Interactive Components (In-Page Logic)**

#### Flip Card (B1)
- Manually clickable to flip 360° view image
- Auto-flips every 5 seconds if not manually interacted
- Uses CSS `perspective: 1000px` and `transform-style: preserve-3d`

#### Carousel (B5)
- 7 product angle images auto-advance every 7 seconds
- Manual prev/next buttons + dot navigation (dots are clickable)
- Carousel index resets timer on manual interaction

#### Quantity Selection (B7)
- 3 preset packs: Single (₦155k), Couples (₦310k), Family (₦465k)
- Visual feedback: selected pack gets `quantity-highlight` border + red shadow
- Selection persists across modal opens

#### 3D Tilt Button (thankyou-page.html)
- Manager's Office CTA button responds to mouse position
- On mousemove: `rotateX/rotateY` based on cursor proximity + dynamic shadow
- Resets on mouseleave

### 6. **Tailwind CSS vs. Custom CSS**
- **index.html**: Uses **Tailwind CDN** + custom `<style>` for overrides
- **managers-office.html & thankyou-page.html**: Pure **custom CSS** (no Tailwind)
- When adding styles, check the **existing pattern in the file**—don't mix Tailwind and custom CSS in new sections

### 7. **Image Assets**
All product images are **local PNG/JPG files** in the root directory:
- `LP1.png`, `LP3.png`: Flip card front/back
- `CR1.png` through `CR8.jpg`: Carousel images  
- `BONUS1.png`, `BONUS2.webp`: Promotional imagery

**Path format:** Relative to HTML file root (e.g., `src="LP1.png"`)

### 8. **Localization (Nigerian Market)**
- Currency: **Nigerian Naira (₦)** with `toLocaleString('en-NG')` formatting
- Testimonials feature **Nigerian names & locations** (Lagos, Abuja, etc.)
- Delivery guarantees: "**2-3 Business Days**" + "**Pay On Delivery (POD)**"
- WhatsApp is primary contact method (not email)

## Common Workflows

### Adding a New Offer/Price Tier
1. Add entry to `OFFERS` array in `<script>`
2. Update `renderQuantityOptions()` to re-render cards
3. Add a new `BONUS*.png` image if needed

### Modifying Announcement Messages
Edit the `ANNOUNCEMENTS` array—rotation is automatic every 10 seconds

### Changing CTA Button Colors
Update CSS variables at `:root` scope; changes propagate site-wide via `hsl(var(--primary))`

### Adding Form Validation
- Form 1: Check `#modal-quantity-options input[name="modal_quantity"]:checked`  
- Form 2: Validate in `handleForm2Submission()` before preventing default

## Key Files to Reference

| File | Purpose |
|------|---------|
| `index.html` | Core sales funnel—study for interactive patterns |
| `managers-office.html` | Simpler page structure; reference for CSS-only styling |
| `thankyou-page.html` | Post-purchase UX; study 3D tilt effect implementation |

## Testing Tips
- **Forms**: Open browser DevTools → Network tab → POST requests to Formspree
- **Scroll behavior**: Test on mobile (viewport < 768px) to verify CTA bar responsiveness
- **Theme**: Check localStorage in DevTools → Application → Local Storage
- **Countdown**: Timer resets at UTC hour boundary (set in `startCountdown()`)

## Common Gotchas
- ⚠️ Forms redirect **manually** (not via Formspree)—check `window.location.href = 'thankyou-page.html'`
- ⚠️ Carousel auto-advance uses `setInterval()`—clear with `clearInterval(carouselTimer)` before resetting
- ⚠️ Dark mode only applies to `<body>` class; ensure all selectors account for `.dark` prefix
- ⚠️ Quantity selection in Form 2 is **required**—validation scrolls to B7 section if missing
