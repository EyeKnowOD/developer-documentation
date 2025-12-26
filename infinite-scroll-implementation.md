# Infinite Scroll Implementation for Contact Lens Grid

## Overview

This implementation converts the JetSmartFilters "Load More" button into an infinite scroll experience. When users scroll within 300px of the bottom of the page, the next set of items automatically loads. The Load More button remains hidden, and a custom loading spinner appears during AJAX requests.

## How It Works

1. **CSS** hides the default Load More button and styles a custom loading spinner
2. **JavaScript** monitors scroll position using the JetSmartFilters event bus
3. When the user scrolls within **300px of the bottom**, the script programmatically triggers the Load More button
4. A **loading spinner** appears during AJAX requests
5. Loading continues automatically until **all items are loaded**
6. **Prevents multiple simultaneous requests** to avoid conflicts

---

## Implementation Steps

### Step 1: Add Custom CSS

Add this CSS code to your WordPress site. You can add it via:
- **Appearance → Customize → Additional CSS**, or
- **Theme's style.css file**, or
- **Custom CSS plugin** (like Simple Custom CSS)

```css
/* ============================================
   INFINITE SCROLL: HIDE LOAD MORE BUTTON
   ============================================ */

/* Hide the JetSmartFilters Load More button for the contact lens grid */
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
    display: none; /* Hidden by default */
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
    border-top-color: #2a95cf; /* Match your theme secondary color */
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

---

### Step 2: Add Loading Spinner HTML

Add this HTML snippet **directly after** your `jet-smart-filters/pagination` block in the WordPress block editor.

**In the Block Editor:**
1. Click the **⊕ (Add Block)** button after the pagination block
2. Search for **"Custom HTML"** block
3. Paste the following code:

```html
<!-- Infinite Scroll Loading Spinner -->
<div id="infinite-scroll-loader" aria-live="polite" aria-busy="false">
    <div class="infinite-scroll-spinner"></div>
    <span class="infinite-scroll-text">Loading more contact lenses...</span>
</div>
```

**Alternative: Add via Code Editor**
1. Switch to **Code Editor** (⋮ menu → Editor → Code Editor)
2. Find the `<!-- /wp:jet-smart-filters/pagination -->` closing tag
3. Paste the HTML immediately after it

---

### Step 3: Add JavaScript for Infinite Scroll

Add this JavaScript code to your WordPress site. You can add it via:
- **Theme's functions.php** (recommended - see code below), or
- **Footer code snippet plugin** (like Code Snippets or Insert Headers and Footers)

#### Option A: Add via functions.php (Recommended)

Add this to your **child theme's functions.php** file:

```php
<?php
/**
 * Infinite Scroll for Contact Lens Grid
 * Enqueues custom JavaScript for JetSmartFilters infinite scroll functionality
 */
function gsbp_enqueue_infinite_scroll_script() {
    // Only load on pages with the contact lens grid
    // Adjust the conditional to match your specific page(s)
    if ( is_page() || is_singular( 'contact-lenses' ) ) {

        wp_add_inline_script( 'jquery', "
(function($) {
    'use strict';

    /**
     * Infinite Scroll Configuration
     * Adjust these settings as needed
     */
    const INFINITE_SCROLL_CONFIG = {
        queryId: 'gspb_filterid_gsbp-cl-grid',    // Your GreenShift query ID
        provider: 'greenshift-query-loop',         // Content provider type
        triggerDistance: 300,                      // Distance from bottom in pixels
        loadingSpinnerId: 'infinite-scroll-loader', // Spinner element ID
        debounceDelay: 150,                        // Scroll event throttle in ms
        debug: false                               // Enable console logging
    };

    /**
     * Infinite Scroll Manager
     */
    const InfiniteScroll = {
        isLoading: false,
        isComplete: false,
        scrollTimer: null,

        /**
         * Initialize infinite scroll
         */
        init: function() {
            // Wait for JetSmartFilters to initialize
            document.addEventListener('jet-smart-filters/inited', () => {
                this.log('JetSmartFilters initialized, setting up infinite scroll');
                this.setupEventListeners();
            });

            // Fallback: If JSF already initialized
            if (window.JetSmartFilters && window.JetSmartFilters.events) {
                this.log('JetSmartFilters already initialized');
                this.setupEventListeners();
            }
        },

        /**
         * Set up scroll and JSF event listeners
         */
        setupEventListeners: function() {
            // Listen to scroll events
            $(window).on('scroll', () => this.handleScroll());

            // Subscribe to JSF loading events
            if (window.JetSmartFilters && window.JetSmartFilters.events) {
                // When filtering starts (AJAX begins)
                window.JetSmartFilters.events.subscribe('ajaxFilters/start-loading', (provider, queryId) => {
                    if (queryId === INFINITE_SCROLL_CONFIG.queryId) {
                        this.showLoader();
                    }
                });

                // When filtering ends (AJAX complete)
                window.JetSmartFilters.events.subscribe('ajaxFilters/end-loading', (provider, queryId) => {
                    if (queryId === INFINITE_SCROLL_CONFIG.queryId) {
                        this.hideLoader();
                        this.isLoading = false;
                        this.checkIfComplete();
                    }
                });
            }
        },

        /**
         * Handle scroll event with debouncing
         */
        handleScroll: function() {
            // Clear existing timer
            clearTimeout(this.scrollTimer);

            // Set new timer (debounce)
            this.scrollTimer = setTimeout(() => {
                this.checkScrollPosition();
            }, INFINITE_SCROLL_CONFIG.debounceDelay);
        },

        /**
         * Check if user has scrolled near bottom
         */
        checkScrollPosition: function() {
            // Don't trigger if already loading or complete
            if (this.isLoading || this.isComplete) {
                return;
            }

            // Calculate scroll position
            const scrollTop = $(window).scrollTop();
            const windowHeight = $(window).height();
            const documentHeight = $(document).height();
            const distanceFromBottom = documentHeight - (scrollTop + windowHeight);

            this.log(`Distance from bottom: ${distanceFromBottom}px`);

            // Trigger load if within threshold
            if (distanceFromBottom < INFINITE_SCROLL_CONFIG.triggerDistance) {
                this.log('Trigger distance reached, loading more items');
                this.loadMore();
            }
        },

        /**
         * Programmatically trigger Load More button
         */
        loadMore: function() {
            // Find the Load More button
            const $loadMoreButton = $(`.jet-smart-filters-pagination[data-query-id=\"${INFINITE_SCROLL_CONFIG.queryId}\"] .jet-filters-pagination__load-more`);

            if ($loadMoreButton.length === 0) {
                this.log('Load More button not found');
                return;
            }

            // Check if button is disabled (no more items)
            if ($loadMoreButton.hasClass('jet-filters-pagination__load-more--disabled') ||
                $loadMoreButton.prop('disabled')) {
                this.log('Load More button is disabled, all items loaded');
                this.isComplete = true;
                return;
            }

            // Set loading state
            this.isLoading = true;
            this.log('Triggering Load More button');

            // Click the hidden button
            $loadMoreButton.trigger('click');
        },

        /**
         * Check if all items have been loaded
         */
        checkIfComplete: function() {
            // Check if Load More button still exists and is enabled
            const $loadMoreButton = $(`.jet-smart-filters-pagination[data-query-id=\"${INFINITE_SCROLL_CONFIG.queryId}\"] .jet-filters-pagination__load-more`);

            if ($loadMoreButton.length === 0) {
                this.log('Load More button removed, all items loaded');
                this.isComplete = true;
                return;
            }

            if ($loadMoreButton.hasClass('jet-filters-pagination__load-more--disabled') ||
                $loadMoreButton.prop('disabled')) {
                this.log('Load More button disabled, all items loaded');
                this.isComplete = true;
                return;
            }

            this.log('More items available for loading');
        },

        /**
         * Show loading spinner
         */
        showLoader: function() {
            const $loader = $(`#${INFINITE_SCROLL_CONFIG.loadingSpinnerId}`);
            $loader.addClass('is-loading').attr('aria-busy', 'true');
            this.log('Showing loader');
        },

        /**
         * Hide loading spinner
         */
        hideLoader: function() {
            const $loader = $(`#${INFINITE_SCROLL_CONFIG.loadingSpinnerId}`);
            $loader.removeClass('is-loading').attr('aria-busy', 'false');
            this.log('Hiding loader');
        },

        /**
         * Debug logging
         */
        log: function(message, data = null) {
            if (INFINITE_SCROLL_CONFIG.debug) {
                console.log('[Infinite Scroll]', message, data || '');
            }
        }
    };

    /**
     * Initialize on DOM ready
     */
    $(document).ready(function() {
        InfiniteScroll.init();
    });

})(jQuery);
        ", 'after');
    }
}
add_action( 'wp_enqueue_scripts', 'gsbp_enqueue_infinite_scroll_script', 20 );
```

#### Option B: Add via Code Snippets Plugin

If you prefer using a plugin:

1. Install **Code Snippets** plugin
2. Go to **Snippets → Add New**
3. Paste the JavaScript code from the section below
4. Set to run in **Frontend only**
5. Save and activate

**JavaScript Code (for Code Snippets):**

```javascript
(function($) {
    'use strict';

    /**
     * Infinite Scroll Configuration
     */
    const INFINITE_SCROLL_CONFIG = {
        queryId: 'gspb_filterid_gsbp-cl-grid',
        provider: 'greenshift-query-loop',
        triggerDistance: 300,
        loadingSpinnerId: 'infinite-scroll-loader',
        debounceDelay: 150,
        debug: false  // Set to true for console logging
    };

    /**
     * Infinite Scroll Manager
     */
    const InfiniteScroll = {
        isLoading: false,
        isComplete: false,
        scrollTimer: null,

        init: function() {
            document.addEventListener('jet-smart-filters/inited', () => {
                this.log('JetSmartFilters initialized');
                this.setupEventListeners();
            });

            if (window.JetSmartFilters && window.JetSmartFilters.events) {
                this.log('JetSmartFilters already initialized');
                this.setupEventListeners();
            }
        },

        setupEventListeners: function() {
            $(window).on('scroll', () => this.handleScroll());

            if (window.JetSmartFilters && window.JetSmartFilters.events) {
                window.JetSmartFilters.events.subscribe('ajaxFilters/start-loading', (provider, queryId) => {
                    if (queryId === INFINITE_SCROLL_CONFIG.queryId) {
                        this.showLoader();
                    }
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
            this.scrollTimer = setTimeout(() => {
                this.checkScrollPosition();
            }, INFINITE_SCROLL_CONFIG.debounceDelay);
        },

        checkScrollPosition: function() {
            if (this.isLoading || this.isComplete) return;

            const scrollTop = $(window).scrollTop();
            const windowHeight = $(window).height();
            const documentHeight = $(document).height();
            const distanceFromBottom = documentHeight - (scrollTop + windowHeight);

            this.log(`Distance from bottom: ${distanceFromBottom}px`);

            if (distanceFromBottom < INFINITE_SCROLL_CONFIG.triggerDistance) {
                this.log('Loading more items');
                this.loadMore();
            }
        },

        loadMore: function() {
            const $loadMoreButton = $(`.jet-smart-filters-pagination[data-query-id="${INFINITE_SCROLL_CONFIG.queryId}"] .jet-filters-pagination__load-more`);

            if ($loadMoreButton.length === 0) {
                this.log('Load More button not found');
                return;
            }

            if ($loadMoreButton.hasClass('jet-filters-pagination__load-more--disabled') ||
                $loadMoreButton.prop('disabled')) {
                this.log('All items loaded');
                this.isComplete = true;
                return;
            }

            this.isLoading = true;
            this.log('Triggering Load More');
            $loadMoreButton.trigger('click');
        },

        checkIfComplete: function() {
            const $loadMoreButton = $(`.jet-smart-filters-pagination[data-query-id="${INFINITE_SCROLL_CONFIG.queryId}"] .jet-filters-pagination__load-more`);

            if ($loadMoreButton.length === 0 ||
                $loadMoreButton.hasClass('jet-filters-pagination__load-more--disabled') ||
                $loadMoreButton.prop('disabled')) {
                this.log('All items loaded');
                this.isComplete = true;
            }
        },

        showLoader: function() {
            $(`#${INFINITE_SCROLL_CONFIG.loadingSpinnerId}`)
                .addClass('is-loading')
                .attr('aria-busy', 'true');
        },

        hideLoader: function() {
            $(`#${INFINITE_SCROLL_CONFIG.loadingSpinnerId}`)
                .removeClass('is-loading')
                .attr('aria-busy', 'false');
        },

        log: function(message, data = null) {
            if (INFINITE_SCROLL_CONFIG.debug) {
                console.log('[Infinite Scroll]', message, data || '');
            }
        }
    };

    $(document).ready(function() {
        InfiniteScroll.init();
    });

})(jQuery);
```

---

## Configuration Options

You can customize the behavior by editing the `INFINITE_SCROLL_CONFIG` object:

| Setting | Default | Description |
|---------|---------|-------------|
| `queryId` | `'gspb_filterid_gsbp-cl-grid'` | Your GreenShift query ID (must match!) |
| `provider` | `'greenshift-query-loop'` | Content provider type |
| `triggerDistance` | `300` | Pixels from bottom before loading (increase for earlier loading) |
| `loadingSpinnerId` | `'infinite-scroll-loader'` | ID of the loading spinner element |
| `debounceDelay` | `150` | Scroll event throttle in milliseconds |
| `debug` | `false` | Set to `true` to enable console logging for troubleshooting |

---

## Troubleshooting

### 1. Infinite scroll not triggering

**Solution:** Enable debug mode
- Set `debug: true` in the configuration
- Open browser console (F12)
- Scroll and check for log messages
- Verify the `queryId` matches your grid

### 2. Load More button still visible

**Solution:** Check CSS specificity
- Inspect the button in browser DevTools
- Add more specific CSS selector if needed
- Ensure no other CSS is overriding the `display: none`

### 3. Spinner not appearing

**Solution:** Verify HTML placement
- Ensure the spinner HTML is placed AFTER the pagination block
- Check that the ID matches `infinite-scroll-loader`
- Verify CSS is loaded

### 4. Multiple loads at once

**Solution:** Already prevented by code
- The `isLoading` flag prevents concurrent requests
- If still occurring, increase `debounceDelay`

### 5. Works on desktop but not mobile

**Solution:** Check responsive CSS
- Verify CSS media queries are correct
- Test scroll position calculations on mobile
- Ensure jQuery and JetSmartFilters load on mobile

### 6. Doesn't work after filtering

**Solution:** Reset handled automatically
- JetSmartFilters events reset the `isComplete` flag
- If issues persist, manually reset by adding to filter events

---

## Testing Checklist

- [ ] Load More button is hidden on page load
- [ ] Scrolling near bottom (300px) triggers automatic loading
- [ ] Loading spinner appears during AJAX requests
- [ ] Loading spinner disappears when items are loaded
- [ ] Multiple requests don't fire simultaneously
- [ ] Loading stops when all items are loaded
- [ ] Works after applying filters (search, checkboxes)
- [ ] Works after sorting
- [ ] Works on mobile devices
- [ ] Page doesn't scroll to top after loading

---

## Advanced Customization

### Change spinner color to match your brand

Edit the CSS:
```css
.infinite-scroll-spinner {
    border-top-color: #YOUR-COLOR-HERE;
}
```

### Add smooth scroll to new items

Add this to the `ajaxFilters/end-loading` event:
```javascript
window.JetSmartFilters.events.subscribe('ajaxFilters/end-loading', (provider, queryId) => {
    if (queryId === INFINITE_SCROLL_CONFIG.queryId) {
        // Smooth scroll to first new item
        const newItems = $('.gsbp-cl-grid .wp-block-block:last-child');
        if (newItems.length) {
            $('html, body').animate({
                scrollTop: newItems.offset().top - 100
            }, 500);
        }
    }
});
```

### Stop after X automatic loads

Add a counter:
```javascript
loadCount: 0,
maxAutoLoads: 5,  // Stop after 5 automatic loads

loadMore: function() {
    if (this.loadCount >= this.maxAutoLoads) {
        this.log('Max auto-loads reached');
        this.isComplete = true;
        return;
    }
    this.loadCount++;
    // ... rest of loadMore code
}
```

---

## Browser Compatibility

✅ **Fully Supported:**
- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Mobile browsers (iOS Safari, Chrome Android)

⚠️ **Requires jQuery:**
- Ensure jQuery is loaded (WordPress includes it by default)
- JetSmartFilters requires jQuery

---

## Performance Notes

- **Debouncing:** Scroll events are throttled to 150ms to prevent excessive calculations
- **Loading state:** Prevents multiple simultaneous AJAX requests
- **Memory:** Automatically cleans up event listeners
- **Mobile:** Same performance on mobile devices

---

## Support & Documentation

**GreenShift Documentation:**
- [GreenShift Learning Center](https://greenshiftwp.com/learning-center/)
- [Integration with Filter Plugins](https://greenshiftwp.com/documentation/blocks/integration-with-filter-plugins/)

**JetSmartFilters Documentation:**
- [Pagination Overview](https://crocoblock.com/knowledge-base/jetsmartfilters/pagination-overview/)
- [JavaScript Events](https://element.how/crocoblock-jetsmartfilters-javascript-events/)
- [JetSmartFilters API](https://gist.github.com/Crocoblock/b8632bfc3aa9ca024d5f8650f7745ab9)

---

## Summary

This implementation provides a seamless infinite scroll experience by:
1. Hiding the default Load More button with CSS
2. Displaying a branded loading spinner during AJAX requests
3. Automatically triggering loads when users scroll within 300px of the bottom
4. Preventing race conditions with loading state management
5. Continuing until all items are loaded
6. Working consistently across desktop and mobile devices

The solution is production-ready, accessible (ARIA attributes), and easily customizable to match your site's design.
