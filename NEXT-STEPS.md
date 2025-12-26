# ✅ INFINITE SCROLL - NEXT STEPS

Your modified block code is ready! Follow these steps in order to complete the implementation.

---

## 🔄 STEP 1: Update Your WordPress Page

### Option A: Replace in Code Editor (Recommended)
1. Go to your Contact Lens page in WordPress
2. Click the **⋮ (three dots)** menu in the top right
3. Select **"Code editor"**
4. **Select ALL existing code** (Ctrl+A or Cmd+A)
5. **Copy the modified code** from `modified-contact-lens-block.html`
6. **Paste it** to replace everything
7. Click **"Update"** to save

### Option B: Add Just the Spinner (If you prefer)
1. In the **Visual Editor**, find your **JetSmartFilters Pagination** block
2. Click the **⊕ (Add block)** button right after it
3. Search for **"HTML"** or **"Custom HTML"**
4. Paste this code:

```html
<div id="infinite-scroll-loader" aria-live="polite" aria-busy="false">
    <div class="infinite-scroll-spinner"></div>
    <span class="infinite-scroll-text">Loading more contact lenses...</span>
</div>
```

5. Click **"Update"**

---

## 🎨 STEP 2: Add CSS Styling

Go to **WordPress Dashboard → Appearance → Customize → Additional CSS**

Paste this complete CSS code:

```css
/* ============================================
   INFINITE SCROLL: HIDE LOAD MORE BUTTON
   ============================================ */

/* Hide the JetSmartFilters Load More button */
.jet-smart-filters-pagination[data-query-id="gspb_filterid_gsbp-cl-grid"] .jet-filters-pagination__load-more {
    display: none !important;
}

/* Alternative: Target by class if the above doesn't work */
.jet-filters-pagination__load-more {
    display: none !important;
}

/* ============================================
   INFINITE SCROLL: LOADING SPINNER
   ============================================ */

/* Loading spinner container */
#infinite-scroll-loader {
    display: none;
    text-align: center;
    padding: 40px 20px;
    margin: 20px auto;
    width: 100%;
}

/* Show when loading */
#infinite-scroll-loader.is-loading {
    display: block;
}

/* Spinner animation */
.infinite-scroll-spinner {
    display: inline-block;
    width: 50px;
    height: 50px;
    border: 4px solid #e2e8f0;
    border-top-color: #2a95cf;
    border-radius: 50%;
    animation: infinite-scroll-spin 0.8s linear infinite;
}

/* Spinner rotation animation */
@keyframes infinite-scroll-spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}

/* Loading text below spinner */
.infinite-scroll-text {
    display: block;
    margin-top: 16px;
    font-size: 14px;
    color: #64748b;
    font-weight: 500;
}

/* Responsive adjustments */
@media (max-width: 768px) {
    #infinite-scroll-loader {
        padding: 30px 15px;
        margin: 15px auto;
    }

    .infinite-scroll-spinner {
        width: 40px;
        height: 40px;
        border-width: 3px;
    }

    .infinite-scroll-text {
        font-size: 13px;
    }
}
```

Click **"Publish"** to save.

---

## 💻 STEP 3: Add JavaScript Functionality

### ⚠️ IMPORTANT: Choose ONE method only (A or B, not both!)

### Option A: Add to functions.php (Recommended)

1. Go to **Appearance → Theme File Editor**
2. Select your **Child Theme** (if you don't have one, create it first!)
3. Click on **functions.php**
4. **Scroll to the bottom** of the file
5. Paste this code:

```php
<?php
/**
 * Infinite Scroll for Contact Lens Grid
 */
function gsbp_enqueue_infinite_scroll_script() {
    if ( is_page() || is_singular( 'contact-lenses' ) ) {
        wp_add_inline_script( 'jquery', "
(function($) {
    'use strict';
    const INFINITE_SCROLL_CONFIG = {
        queryId: 'gspb_filterid_gsbp-cl-grid',
        triggerDistance: 300,
        loadingSpinnerId: 'infinite-scroll-loader',
        debounceDelay: 150,
        debug: false
    };

    const InfiniteScroll = {
        isLoading: false,
        isComplete: false,
        scrollTimer: null,

        init: function() {
            document.addEventListener('jet-smart-filters/inited', () => {
                this.setupEventListeners();
            });
            if (window.JetSmartFilters && window.JetSmartFilters.events) {
                this.setupEventListeners();
            }
        },

        setupEventListeners: function() {
            $(window).on('scroll', () => this.handleScroll());
            if (window.JetSmartFilters && window.JetSmartFilters.events) {
                window.JetSmartFilters.events.subscribe('ajaxFilters/start-loading', (provider, queryId) => {
                    if (queryId === INFINITE_SCROLL_CONFIG.queryId) this.showLoader();
                });
                window.JetSmartFilters.events.subscribe('ajaxFilters/end-loading', (provider, queryId) => {
                    if (queryId === INFINITE_SCROLL_CONFIG.queryId) {
                        this.hideLoader();
                        this.isLoading = false;
                        this.checkIfComplete();
                    }
                });
            }
        },

        handleScroll: function() {
            clearTimeout(this.scrollTimer);
            this.scrollTimer = setTimeout(() => this.checkScrollPosition(), INFINITE_SCROLL_CONFIG.debounceDelay);
        },

        checkScrollPosition: function() {
            if (this.isLoading || this.isComplete) return;
            const scrollTop = $(window).scrollTop();
            const windowHeight = $(window).height();
            const documentHeight = $(document).height();
            const distanceFromBottom = documentHeight - (scrollTop + windowHeight);

            if (distanceFromBottom < INFINITE_SCROLL_CONFIG.triggerDistance) {
                this.loadMore();
            }
        },

        loadMore: function() {
            const \$btn = $(\`.jet-smart-filters-pagination[data-query-id=\"\${INFINITE_SCROLL_CONFIG.queryId}\"] .jet-filters-pagination__load-more\`);
            if (\$btn.length === 0) return;
            if (\$btn.hasClass('jet-filters-pagination__load-more--disabled') || \$btn.prop('disabled')) {
                this.isComplete = true;
                return;
            }
            this.isLoading = true;
            \$btn.trigger('click');
        },

        checkIfComplete: function() {
            const \$btn = $(\`.jet-smart-filters-pagination[data-query-id=\"\${INFINITE_SCROLL_CONFIG.queryId}\"] .jet-filters-pagination__load-more\`);
            if (\$btn.length === 0 || \$btn.hasClass('jet-filters-pagination__load-more--disabled') || \$btn.prop('disabled')) {
                this.isComplete = true;
            }
        },

        showLoader: function() {
            $(\`#\${INFINITE_SCROLL_CONFIG.loadingSpinnerId}\`).addClass('is-loading').attr('aria-busy', 'true');
        },

        hideLoader: function() {
            $(\`#\${INFINITE_SCROLL_CONFIG.loadingSpinnerId}\`).removeClass('is-loading').attr('aria-busy', 'false');
        }
    };

    $(document).ready(function() {
        InfiniteScroll.init();
    });
})(jQuery);
        ", 'after');
    }
}
add_action( 'wp_enqueue_scripts', 'gsbp_enqueue_infinite_scroll_script', 20 );
```

6. Click **"Update File"**

### Option B: Use Code Snippets Plugin

1. Install **Code Snippets** plugin from WordPress repository
2. Go to **Snippets → Add New**
3. Give it a title: "Infinite Scroll for Contact Lenses"
4. Paste the JavaScript code from `infinite-scroll-code.md` (Option B section)
5. Set **Run snippet**: Everywhere (or Front-end only)
6. Click **"Save Changes and Activate"**

---

## 🧪 STEP 4: Test the Implementation

### Clear Cache First
1. **Clear WordPress cache** (if using caching plugin)
2. **Clear browser cache** (Ctrl+Shift+R or Cmd+Shift+R)
3. **Clear any CDN cache** (Cloudflare, etc.)

### Test Checklist

✅ **Load More button should be hidden**
- Visit your Contact Lens page
- The "Load More" button should NOT be visible

✅ **Infinite scroll should trigger automatically**
- Scroll down to near the bottom
- More items should load automatically at ~300px from bottom
- Loading spinner should appear during loading

✅ **Spinner should appear/disappear**
- Watch for the spinner when scrolling triggers load
- It should disappear when new items finish loading

✅ **Filters should still work**
- Try searching for a contact lens
- Try checking filter boxes
- Infinite scroll should work with filtered results

✅ **All items should eventually load**
- Keep scrolling until no more items
- Spinner should stop appearing
- No console errors (check F12 developer tools)

### Enable Debug Mode (If issues occur)

In the JavaScript code, change:
```javascript
debug: false  // Change to true
```

Then:
1. Save the file
2. Open browser console (F12)
3. Scroll and watch for log messages
4. Look for errors or issues

---

## 🎨 OPTIONAL: Customizations

### Change Spinner Color
In the CSS, find:
```css
border-top-color: #2a95cf;  /* Change this */
```

### Change Loading Text
In the HTML, find:
```html
<span class="infinite-scroll-text">Loading more contact lenses...</span>
```

### Load Earlier (500px from bottom)
In the JavaScript, change:
```javascript
triggerDistance: 300,  // Change to 500
```

---

## 🐛 Troubleshooting

### Load More button still showing?
- Clear all caches
- Check CSS is added correctly
- Try the alternative CSS selector

### Infinite scroll not triggering?
- Enable `debug: true` and check console
- Verify query ID matches: `gspb_filterid_gsbp-cl-grid`
- Check JavaScript is loaded (View Source → search for "InfiniteScroll")

### Spinner not appearing?
- Verify HTML is placed AFTER pagination block
- Check CSS is added
- Inspect element (F12) to see if `is-loading` class is added

### Multiple loads at once?
- This is prevented by the code
- If still happening, increase `debounceDelay` to 250

### Works on desktop but not mobile?
- Clear mobile cache
- Test in incognito mode
- Check responsive CSS loaded correctly

---

## 📝 Summary of Changes

✅ **HTML**: Added loading spinner after pagination block
✅ **CSS**: Hide Load More button + Style loading spinner
✅ **JavaScript**: Auto-trigger loads when scrolling near bottom

---

## 📞 Need Help?

See the full documentation files:
- **`infinite-scroll-code.md`** - All code blocks separated
- **`infinite-scroll-implementation.md`** - Full guide with explanations

---

## ✨ You're Done!

Once all 4 steps are complete, your infinite scroll should be working perfectly. Users will scroll down, and more contact lenses will load automatically without clicking any button!
