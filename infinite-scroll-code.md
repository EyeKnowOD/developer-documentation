# Infinite Scroll - Code Blocks for Copy/Paste

Quick reference with all code blocks separated for easy implementation.

---

## 1️⃣ CSS CODE

**Where to add:** WordPress → Appearance → Customize → Additional CSS

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

## 2️⃣ HTML CODE

**Where to add:** WordPress Block Editor → Add "Custom HTML" block after pagination

```html
<!-- Infinite Scroll Loading Spinner -->
<div id="infinite-scroll-loader" aria-live="polite" aria-busy="false">
    <div class="infinite-scroll-spinner"></div>
    <span class="infinite-scroll-text">Loading more contact lenses...</span>
</div>
```

---

## 3️⃣ JAVASCRIPT CODE - Option A (functions.php)

**Where to add:** Child theme's `functions.php` file

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

            this.log(\`Distance from bottom: \${distanceFromBottom}px\`);

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
            const $loadMoreButton = $(\`.jet-smart-filters-pagination[data-query-id=\"\${INFINITE_SCROLL_CONFIG.queryId}\"] .jet-filters-pagination__load-more\`);

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
            const $loadMoreButton = $(\`.jet-smart-filters-pagination[data-query-id=\"\${INFINITE_SCROLL_CONFIG.queryId}\"] .jet-filters-pagination__load-more\`);

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
            const $loader = $(\`#\${INFINITE_SCROLL_CONFIG.loadingSpinnerId}\`);
            $loader.addClass('is-loading').attr('aria-busy', 'true');
            this.log('Showing loader');
        },

        /**
         * Hide loading spinner
         */
        hideLoader: function() {
            const $loader = $(\`#\${INFINITE_SCROLL_CONFIG.loadingSpinnerId}\`);
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

---

## 3️⃣ JAVASCRIPT CODE - Option B (Code Snippets Plugin / Footer)

**Where to add:** Code Snippets plugin OR Insert Headers & Footers plugin (footer section)

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

## 🎨 CUSTOMIZATION QUICK REFERENCE

### Change spinner color
In the CSS, find this line:
```css
border-top-color: #2a95cf; /* Change this color */
```

### Change trigger distance
In the JavaScript, find this line:
```javascript
triggerDistance: 300,  // Change to 500 for earlier loading
```

### Enable debug logging
In the JavaScript, find this line:
```javascript
debug: false  // Change to true
```

### Change loading text
In the HTML, find this line:
```html
<span class="infinite-scroll-text">Loading more contact lenses...</span>
```

---

## ⚙️ IMPLEMENTATION ORDER

1. ✅ Copy/paste **CSS** → Save
2. ✅ Copy/paste **HTML** → Update page
3. ✅ Copy/paste **JavaScript** (choose Option A OR B) → Save
4. ✅ Test by scrolling down

---

## 🐛 QUICK TROUBLESHOOTING

**Not working?** Set `debug: true` and check browser console (F12)

**Button still showing?** Clear site cache and browser cache

**Spinner not appearing?** Verify HTML is placed AFTER the pagination block

---

For full documentation with explanations, see `infinite-scroll-implementation.md`
