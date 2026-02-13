# Navbar Refactoring Plan

## Current State Analysis
- Bootstrap 4 alpha navbar with fixed-top positioning
- Inline styles on toggle button
- Logo not clickable
- Menu items use hash anchors (#about, #cc, etc.)
- Active state manually set
- Social media icons in navbar

## Refactoring Plan

### 1. HTML Structure Changes
- Make logo clickable (link to index.html)
- Update menu items to new structure:
  - Home → index.html
  - Team → index.html#cc (or team.html if separate page)
  - Committees → committees.html
  - Recent Editions → index.html#yra (or recent-editions.html)
  - Other Events → index.html#yra (or other-events.html)
  - Contact Us → index.html#contact (or contact.html)
- Remove inline styles
- Add data attributes for active page detection
- Keep social media icons

### 2. CSS Enhancements
- Enhanced sticky behavior with smooth background transition
- Professional UN-style design (dark, elegant, minimal)
- Subtle hover effects (underline animation, color transition)
- Active state styling
- Smooth transitions for all interactions
- Mobile-responsive improvements

### 3. JavaScript Functionality
- Active page detection based on current URL
- Smooth scroll for hash links
- Navbar background change on scroll (transparent to solid)
- Active state management

### 4. Design Philosophy
- Professional and formal
- United Nations aesthetic (dark blue/black, gold accents)
- Subtle animations (no flashy effects)
- Clean typography
- Premium feel

## Implementation Steps
1. Update HTML navbar markup
2. Enhance CSS for sticky behavior and styling
3. Add JavaScript for active page detection and scroll behavior
4. Test on all breakpoints
5. Create change log report

