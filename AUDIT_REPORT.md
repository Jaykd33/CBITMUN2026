# CBITMUN 2026 Website - Code Audit Report

## Executive Summary
This audit identifies outdated patterns, structural limitations, and provides a refactoring strategy for the CBITMUN 2026 website. The codebase is functional but requires modernization for scalability, maintainability, and professional standards.

---

## 1. OUTDATED PATTERNS & DEPENDENCIES

### 1.1 Bootstrap Version (CRITICAL)
**Issue:** Using Bootstrap 4.0.0-alpha.6 (2017 alpha release)
- **Location:** Line 11 in `index.html`
- **Problem:** Alpha version, outdated, security concerns, missing modern features
- **Impact:** Unstable, potential compatibility issues, large bundle size
- **Recommendation:** Upgrade to Bootstrap 5.3.x or remove if minimal usage

### 1.2 Font Awesome Version
**Issue:** Font Awesome 4.7.0 (2017 release)
- **Location:** Line 13 in `index.html`
- **Problem:** Outdated, missing modern icons, larger file size
- **Recommendation:** Upgrade to Font Awesome 6.x or consider icon alternatives

### 1.3 Inline Styles (HIGH PRIORITY)
**Issues Found:**
- Line 23: `style="color: rgb(255, 255, 255);"` in navbar toggle
- Line 131: `style="padding: 40px 0px;"` in about section
- Line 135: `width="200"` inline attribute
- Line 139: `style="text-align: justify;"` in paragraph
- Line 152: `style="padding: 0px 0px 80px 0px;"` in Core Committee
- Line 156: `style="margin: 35px 0px;"` in rows
- Line 158: `width="250"` inline attribute (multiple instances)
- Line 207: `style="background: #F6F6F6; padding: 40px 110px;"` in Secretariat
- Line 213-224: **Inline `<style>` block** in HTML (Secretariat section)
- Line 260: `style="padding: 60px 0px;"` in YRA section
- Line 266: `style="height: 170px;"` in card divs (multiple)
- Line 412: Syntax error `style="padding: 40px 0px;">` (missing quote)
- Line 420: `style="text-align: left;"` in paragraph
- Line 422: `style="color: #007bff; text-decoration: none;"` in link

**Impact:** 
- Hard to maintain
- Violates separation of concerns
- Difficult to theme consistently
- Poor scalability

### 1.4 Deprecated HTML Attributes
- Line 78: `align="left"` (deprecated, use CSS)
- Line 79: `align="left"` (deprecated)
- Line 85: `align="left"` (deprecated)
- Line 89: `onclick` inline handler (should use event listeners)

### 1.5 Commented-Out Code Blocks
**Large commented sections:**
- Lines 94-129: Entire committees section commented
- Lines 303-411: Entire partners section commented (includes nested HTML/head/body tags - invalid)
- Lines 486-526: jQuery/Bootstrap scripts commented but still referenced

**Impact:** 
- Code bloat
- Confusion about what's active
- Maintenance burden

### 1.6 JavaScript Dependencies
**Issues:**
- jQuery dependency (commented but referenced)
- Bootstrap JS dependency (commented)
- Mixed vanilla JS and jQuery patterns
- `js/script.js` exists but minimal functionality

**Current State:**
- Hero animations use vanilla JS (good)
- Smooth scroll functionality missing (commented jQuery code)
- Navbar scroll behavior missing (commented jQuery code)

---

## 2. STRUCTURAL LIMITATIONS

### 2.1 HTML Structure Issues

#### 2.1.1 Invalid HTML Structure
- **Line 148:** Duplicate closing `</section>` tag (no opening)
- **Line 412:** Syntax error in contact section: `id="contact"; padding: 40px 0px;">` (semicolon instead of closing quote)
- **Line 164:** Unclosed `<a>` tag (opening tag missing)
- **Line 173:** Unclosed `<a>` tag (opening tag missing)
- **Line 178:** Unclosed `<a>` tag (opening tag missing)
- **Line 186:** Unclosed `<a>` tag (opening tag missing)
- **Line 192:** Unclosed `<a>` tag (opening tag missing)
- **Line 198:** Unclosed `<a>` tag (opening tag missing)

#### 2.1.2 Semantic HTML Issues
- Missing `<main>` wrapper
- Missing proper heading hierarchy (h1 used in hero, then h1 again in sections)
- Table used for Secretariat (should use semantic list/grid)
- Missing ARIA labels for accessibility
- Missing alt text for some images

#### 2.1.3 Scalability Issues
- **Single-page architecture:** All content in one file
- **No component reusability:** Navbar, footer hardcoded
- **No template system:** Can't reuse across pages
- **Hard to maintain:** Changes require editing large file

### 2.2 CSS Architecture Issues

#### 2.2.1 Organization Problems
- **No CSS methodology:** BEM, OOCSS, or SMACSS
- **Mixed concerns:** Layout, typography, components all mixed
- **No CSS variables usage:** Hardcoded colors despite `:root` variables defined
- **Inconsistent naming:** `.hero-unit-primary-text` vs `.h1-heads` vs `.team-name`
- **No modular structure:** Single large file (516 lines)

#### 2.2.2 Specific CSS Issues
- **Line 24-27:** Unused `.padding-table-columns` class
- **Line 437-439:** `.article-font` defined but never used
- **Line 443-445:** `h4{ hyphens:auto }` missing semicolon
- **CSS Variables defined but underutilized:**
  - `--primary-color: #FFB300` defined but hardcoded `#FFB300` used in multiple places
  - `--secondry-color` typo (should be `--secondary-color`)
  - Variables not used consistently

#### 2.2.3 Responsive Design Issues
- **Inconsistent breakpoints:**
  - `@media(max-width: 746px)`
  - `@media(max-width: 575)` (missing `px`)
  - `@media(max-width: 946px)`
  - `@media(max-width: 748px)`
- **No mobile-first approach:** Desktop-first media queries
- **Hardcoded padding:** `padding: 40px 110px;` breaks on mobile
- **Fixed widths:** `width="250"`, `width="200"` don't scale

#### 2.2.4 Vendor Prefixes
- Excessive `-webkit-` prefixes (many no longer needed)
- `-moz-` and `-o-` prefixes outdated
- Should use autoprefixer or remove unnecessary ones

### 2.3 JavaScript Issues

#### 2.3.1 Code Organization
- Inline scripts in HTML (lines 444-484)
- Separate `js/script.js` file but minimal
- Commented jQuery code suggests incomplete migration
- No module system or organization

#### 2.3.2 Functionality Gaps
- Smooth scroll navigation missing (commented)
- Navbar background change on scroll missing (commented)
- No active state management for navigation
- No lazy loading for images
- No intersection observer for animations

---

## 3. REFACTOR STRATEGY

### 3.1 What to REFACTOR (High Priority)

#### 3.1.1 HTML Structure
**Priority: CRITICAL**
- ✅ Fix all HTML syntax errors (unclosed tags, duplicate sections)
- ✅ Remove all inline styles (move to CSS)
- ✅ Remove deprecated attributes (`align`, inline `onclick`)
- ✅ Fix semantic HTML (add `<main>`, proper heading hierarchy)
- ✅ Clean up commented code blocks
- ✅ Extract navbar and footer into reusable components (or separate files)
- ✅ Fix contact section syntax error

**Estimated Impact:** High - Improves maintainability, accessibility, SEO

#### 3.1.2 CSS Architecture
**Priority: HIGH**
- ✅ Organize CSS into logical sections (variables, base, components, utilities)
- ✅ Use CSS variables consistently (replace hardcoded colors)
- ✅ Implement mobile-first responsive design
- ✅ Standardize breakpoints
- ✅ Remove unused CSS (`.padding-table-columns`, `.article-font`)
- ✅ Fix CSS syntax errors (missing semicolons)
- ✅ Remove unnecessary vendor prefixes
- ✅ Create reusable component classes

**Estimated Impact:** High - Improves maintainability, reduces file size, better performance

#### 3.1.3 Dependencies
**Priority: MEDIUM-HIGH**
- ✅ Upgrade Bootstrap to 5.3.x OR remove if minimal usage
- ✅ Upgrade Font Awesome to 6.x OR replace with SVG icons
- ✅ Remove jQuery dependency (convert to vanilla JS)
- ✅ Implement smooth scroll with vanilla JS
- ✅ Implement navbar scroll behavior with vanilla JS

**Estimated Impact:** Medium - Security, performance, modern standards

#### 3.1.4 JavaScript
**Priority: MEDIUM**
- ✅ Consolidate all JS into organized modules
- ✅ Implement smooth scroll navigation
- ✅ Implement navbar scroll behavior
- ✅ Add active state management for navigation
- ✅ Consider lazy loading for images
- ✅ Add intersection observer for scroll animations

**Estimated Impact:** Medium - Better UX, performance, maintainability

### 3.2 What to REUSE (Keep & Enhance)

#### 3.2.1 Hero Section
**Status: GOOD - Keep & Polish**
- Modern hero structure is well-designed
- Good animation implementation
- Responsive design in place
- **Enhancement:** Minor polish, ensure all animations work

#### 3.2.2 Card Components
**Status: ACCEPTABLE - Keep & Enhance**
- `.card-primary` structure is functional
- Hover effects are good
- **Enhancement:** Improve responsive behavior, add loading states

#### 3.2.3 Typography System
**Status: GOOD - Keep & Standardize**
- Font choices are appropriate (Playfair Display, Open Sans)
- **Enhancement:** Create typography scale, ensure consistent usage

#### 3.2.4 Color Scheme
**Status: GOOD - Keep & Systematize**
- CSS variables defined (good foundation)
- **Enhancement:** Use variables consistently, expand palette if needed

#### 3.2.5 Section Structure
**Status: ACCEPTABLE - Keep & Refine**
- Section-based layout is clear
- **Enhancement:** Add consistent spacing system, improve semantic structure

---

## 4. SCALABILITY CONCERNS

### 4.1 Current Limitations
1. **Single-file architecture:** Adding new pages requires duplicating navbar/footer
2. **No component system:** Can't reuse sections across pages
3. **Hardcoded content:** Team members, secretariat hardcoded in HTML
4. **No build process:** No minification, optimization, or bundling
5. **No version control for assets:** Image paths hardcoded

### 4.2 Recommendations for Scalability
1. **Create shared components:**
   - `components/navbar.html` (or JS template)
   - `components/footer.html` (or JS template)
   - Load via JavaScript or server-side includes

2. **Modularize CSS:**
   - `css/variables.css` - CSS custom properties
   - `css/base.css` - Reset, typography
   - `css/components.css` - Reusable components
   - `css/layout.css` - Layout utilities
   - `css/pages/` - Page-specific styles

3. **Data-driven content:**
   - Consider JSON for team members, committees
   - Generate HTML from data (if needed)

4. **Build process (optional but recommended):**
   - Use a simple build tool (Parcel, Vite, or Webpack)
   - Minify CSS/JS
   - Optimize images
   - Bundle assets

---

## 5. PRIORITY MATRIX

### Phase 1: Critical Fixes (Do First)
1. Fix HTML syntax errors
2. Remove inline styles
3. Fix contact section syntax error
4. Remove commented code blocks
5. Fix CSS syntax errors

### Phase 2: Structure & Organization (Do Second)
1. Organize CSS into logical sections
2. Use CSS variables consistently
3. Extract navbar/footer to reusable components
4. Implement mobile-first responsive design
5. Standardize breakpoints

### Phase 3: Modernization (Do Third)
1. Upgrade Bootstrap/Font Awesome (or remove)
2. Remove jQuery dependency
3. Implement smooth scroll with vanilla JS
4. Add active navigation state management
5. Remove unnecessary vendor prefixes

### Phase 4: Enhancement (Do Fourth)
1. Add lazy loading for images
2. Implement intersection observer for animations
3. Add loading states
4. Improve accessibility (ARIA labels, alt text)
5. Optimize performance

---

## 6. ESTIMATED EFFORT

- **Phase 1 (Critical):** 2-3 hours
- **Phase 2 (Structure):** 4-6 hours
- **Phase 3 (Modernization):** 3-4 hours
- **Phase 4 (Enhancement):** 2-3 hours

**Total Estimated Time:** 11-16 hours

---

## 7. RISK ASSESSMENT

### Low Risk Changes
- CSS organization
- Removing commented code
- Using CSS variables
- Removing vendor prefixes

### Medium Risk Changes
- Removing inline styles (need to test all sections)
- Upgrading Bootstrap (may need layout adjustments)
- Converting jQuery to vanilla JS (need thorough testing)

### High Risk Changes
- Restructuring HTML (could break existing functionality)
- Changing navigation structure (could break links)

**Recommendation:** Implement changes incrementally with testing at each phase.

---

## 8. CONCLUSION

The codebase is **functional but requires modernization**. The hero section and some components are well-designed, but the overall structure needs refactoring for maintainability and scalability.

**Key Strengths:**
- Modern hero section with good animations
- Appropriate typography choices
- CSS variables foundation exists
- Responsive design partially implemented

**Key Weaknesses:**
- Outdated dependencies
- Inline styles throughout
- HTML syntax errors
- No component reusability
- Mixed code patterns

**Recommended Approach:**
1. Fix critical issues first (syntax errors, inline styles)
2. Organize and structure code
3. Modernize dependencies
4. Enhance with new features

This refactoring will result in a more maintainable, scalable, and professional codebase while preserving all existing functionality.

