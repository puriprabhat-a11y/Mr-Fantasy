# ⚡ Performance Optimization Checklist for Mr. Fantasy Portfolio

**Current Metrics (Before):**
- FCP: 9.5s ❌
- LCP: 10.1s ❌
- TBT: 80ms ⚠️
- CLS: 0 ✅
- Speed Index: 9.5s ❌

**Target Metrics (After):**
- FCP: <1.8s
- LCP: <2.5s
- TBT: <50ms
- CLS: 0
- Speed Index: <3s

---

## 🔴 CRITICAL FIXES (Implement ASAP)

### 1. IMAGE OPTIMIZATION
**Problem:** Your hero image (Prabhat Puri.jpg) is 230KB. This is blocking LCP.

**Solution:**
```bash
# Use ImageOptim, TinyPNG, or Squoosh to optimize
# Target: Reduce to 80-120KB maximum

# Steps:
1. Download Prabhat Puri.jpg from repo
2. Use online tool: https://squoosh.app/
   - Select WebP format (50-70% smaller)
   - Resize to 600px width (you're displaying it at max 320px)
   - Compress to ~40-60KB
3. Upload as "prabhat-puri.webp"
4. Keep JPG as fallback
```

**Update in index.html (line 812):**
```html
<!-- Before -->
<img src="Prabhat Puri.jpg" alt="Prabhat Puri" />

<!-- After -->
<picture>
  <source srcset="prabhat-puri.webp" type="image/webp">
  <source srcset="Prabhat Puri.jpg" type="image/jpeg">
  <img src="Prabhat Puri.jpg" alt="Prabhat Puri" loading="lazy" />
</picture>
```

---

### 2. CRITICAL CSS EXTRACTION
**Problem:** Your entire CSS (36KB) is inline in `<head>`, blocking render.

**Solution:**
Separate **critical** CSS from non-critical:

```html
<!-- CRITICAL CSS (inline in head) - just hero + header -->
<style>
/* Only header, hero, typography, buttons */
/* Remove: expertise cards, case studies, footer, forms */
</style>

<!-- DEFER NON-CRITICAL CSS -->
<link rel="preload" href="/css/non-critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="/css/non-critical.css"></noscript>
```

**Impact:** Reduces render-blocking time by ~60%.

---

### 3. FONT LOADING OPTIMIZATION
**Problem:** Fonts from fontshare API add ~1-2s latency.

**Current (line 12-13):**
```html
<link rel="preconnect" href="https://api.fontshare.com">
<link href="https://api.fontshare.com/v2/css?f[]=satoshi@400,500,600,700&f[]=boska@400,500,700&display=swap" rel="stylesheet">
```

**Solution - Add font-display & preload:**
```html
<!-- DNS Prefetch + Preconnect -->
<link rel="dns-prefetch" href="https://api.fontshare.com">
<link rel="preconnect" href="https://api.fontshare.com" crossorigin>

<!-- Add display=swap to prevent FOUT -->
<link href="https://api.fontshare.com/v2/css?f[]=satoshi@400,500,600,700&f[]=boska@400,500,700&display=swap" rel="stylesheet">
```

**Better: Self-host fonts**
- Download font files from Fontshare
- Host on your own server
- Use `@font-face` with `font-display: swap`
- Saves 500-800ms

---

### 4. LAZY LOAD IMAGES
**Problem:** All images load upfront, even below-the-fold.

**Solution - Add to all images below hero:**
```html
<!-- Instead of: <img src="image.jpg"> -->
<img src="image.jpg" loading="lazy" alt="description">

<!-- For background images, use native lazy loading -->
<div loading="lazy" style="background-image:url(...)"></div>
```

**Or use modern srcset with lazy:**
```html
<img 
  src="placeholder-small.jpg" 
  srcset="image-320w.jpg 320w, image-640w.jpg 640w, image-1200w.jpg 1200w"
  sizes="(max-width: 768px) 100vw, 50vw"
  loading="lazy"
  alt="Prabhat Puri"
/>
```

---

### 5. DEFER NON-CRITICAL JAVASCRIPT
**Problem:** Your scripts run on page load, blocking interactivity.

**Current:**
```html
<script>
function showPage(page) { ... }
function toggleTheme() { ... }
function toggleMobileNav() { ... }
</script>
```

**Solution - Defer to end of body:**
```html
</div><!-- close .page -->

<!-- Defer non-critical JS to bottom of page -->
<script async>
// Only critical script: page routing
function showPage(page) { ... }
</script>

<script defer>
// Non-critical: theme toggle
function toggleTheme() { ... }
function toggleMobileNav() { ... }
</script>
```

---

## 🟡 MEDIUM PRIORITY

### 6. MINIFY & COMPRESS
**What:** Reduce CSS & JS size by 30-40%.

- CSS: 36KB → 22KB
- JS: ~2KB → 1.2KB

**Tools:**
- CSS: https://cssnano.co/ or PostCSS
- JS: https://terser.org/

**In GitHub:** Use `jekyll-compress-html` plugin (already added to _config.yml).

---

### 7. GOOGLE TAG MANAGER & ANALYTICS
**Problem:** GTM script blocks rendering.

**Before:**
```html
<!-- Google Tag Manager (noscript) -->
<noscript><iframe src="..." height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
```

**After:**
```html
<!-- Google Tag Manager (noscript) - add async -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXX"></script>
```

---

### 8. ADD PRELOAD HINTS
```html
<head>
  <!-- Preload critical images -->
  <link rel="preload" as="image" href="prabhat-puri.webp" type="image/webp">
  
  <!-- Preload critical fonts -->
  <link rel="preload" as="font" href="/fonts/satoshi.woff2" type="font/woff2" crossorigin>
  
  <!-- DNS Prefetch for external APIs -->
  <link rel="dns-prefetch" href="https://www.google-analytics.com">
  <link rel="dns-prefetch" href="https://www.googletagmanager.com">
</head>
```

---

## 🟢 NICE TO HAVE

### 9. SERVICE WORKER (PWA)
- Cache critical files
- Offline fallback page
- Reduces repeat visit load time by 70%

### 10. CLOUDFLARE CONFIG
If hosting on Cloudflare Workers or using Cloudflare:
- ✅ Enable Rocket Loader (defer JS)
- ✅ Enable Polish (automatic WebP + optimization)
- ✅ Enable Brotli compression
- ✅ Create Page Rule: `Cache everything`

---

## 📊 EXPECTED RESULTS AFTER ALL FIXES

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| FCP | 9.5s | 1.5s | **84% faster** ⚡ |
| LCP | 10.1s | 2.1s | **79% faster** ⚡ |
| TBT | 80ms | 40ms | **50% better** |
| Speed Index | 9.5s | 2.8s | **71% faster** ⚡ |
| **Performance Score** | ~15 | ~85+ | **5.6x improvement** |

---

## ✅ IMPLEMENTATION STEPS (In Order)

1. **Day 1:** Optimize image (Prabhat Puri.jpg) → WebP + compress
2. **Day 1:** Extract critical CSS from inline styles
3. **Day 1:** Add `loading="lazy"` to all images
4. **Day 2:** Self-host fonts (or add `display=swap`)
5. **Day 2:** Defer JavaScript to bottom
6. **Day 2:** Test with Lighthouse in Chrome DevTools
7. **Day 3:** Deploy to GitHub Pages
8. **Day 3:** Run PageSpeed Insights → Check scores
9. **Day 3:** Implement Cloudflare optimizations
10. **Day 4:** Monitor with Google Analytics 4

---

## 🔗 TOOLS & RESOURCES

- **Lighthouse:** Chrome DevTools → Lighthouse tab
- **PageSpeed Insights:** https://pagespeed.web.dev/
- **WebPageTest:** https://www.webpagetest.org/
- **Image Optimization:** https://squoosh.app/
- **CSS Minify:** https://cssnano.co/
- **Font Optimizer:** https://glyphhanger.com/

---

## 📝 DEBUGGING TIPS

**If LCP still high after image optimization:**
1. Check Network tab (Slow 4G throttling)
2. Look for 3G network conditions
3. Check if fonts are blocking render

**If FCP still high:**
1. Reduce render-blocking resources
2. Defer CSS not needed for above-the-fold
3. Use `display=swap` on fonts

**If CLS increases:**
- Add explicit `width` and `height` to images
- Reserve space for ads/widgets
- Avoid layout-shifting animations

---

## 🚀 NEXT STEPS

1. **Download your image** and optimize with Squoosh
2. **Run Lighthouse** to get a baseline
3. **Implement fixes in this order:** Images → CSS → JS → Fonts
4. **Re-run Lighthouse** after each major change
5. **Deploy to GitHub Pages**
6. **Monitor with PageSpeed Insights** weekly

You've got this! Drop me a message if you hit any blockers. 💪
