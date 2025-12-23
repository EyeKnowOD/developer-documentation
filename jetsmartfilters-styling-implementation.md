# JetSmartFilters Styling Implementation Guide

## Overview
This guide shows how to style JetSmartFilters to match your Greenshift design system with rounded corners and proper flex layout.

## Implementation Methods

### Method 1: Using Code Snippets Plugin (Recommended)

1. **Go to** WordPress Admin → Snippets → Add New
2. **Name:** "JetSmartFilters Greenshift Styling"
3. **Type:** CSS Snippet
4. **Paste the CSS** from `jetsmartfilters-greenshift-styling.css`
5. **Save and Activate**

### Method 2: Theme Customizer

1. **Go to** Appearance → Customize → Additional CSS
2. **Paste the CSS** from `jetsmartfilters-greenshift-styling.css`
3. **Click Publish**

### Method 3: Child Theme (Most Permanent)

1. **Create/Edit** `style.css` in your child theme
2. **Add the CSS** from `jetsmartfilters-greenshift-styling.css`
3. **Save** and clear cache

---

## Alternative: Adjust Flex Layout in Block Editor

If you want the select dropdown to be even wider, you can adjust the flex properties:

### Current Layout:
- Search: `flex: 1` (grows to fill space)
- Select: `flex: 0 0 250px` (fixed at 250px)

### Option A: Make Select Wider (Fixed Width)
Change the select wrapper flex to:
```
flex: 0 0 350px (desktop)
flex: 0 0 300px (tablet)
flex: 1 1 100% (mobile)
```

### Option B: 60/40 Split
- Search: `flex: 1 1 60%`
- Select: `flex: 1 1 40%`

### Option C: Equal Width
- Search: `flex: 1 1 50%`
- Select: `flex: 1 1 50%`

### How to Change in Block Editor:

1. Select the **gsbp-select-wrap** element
2. In Greenshift panel → **Advanced** → **Flex**
3. Change **Flex** value from `0 0 250px` to your desired value
4. Do the same for **gsbp-search-wrap** if needed

---

## Testing Checklist

After applying the CSS:

- [ ] Search input has rounded corners (8px border-radius)
- [ ] Select dropdown has rounded corners (8px border-radius)
- [ ] Both fields have subtle shadow (matches Greenshift design)
- [ ] Focus states show purple border (#8c02e8)
- [ ] Hover states work on select dropdown
- [ ] Responsive layout works on mobile (filters stack vertically)
- [ ] Select dropdown takes up full width of its container
- [ ] Custom dropdown arrow appears (no browser default arrow)

---

## Customization Options

### Change Border Radius
Find this line in the CSS and adjust:
```css
border-radius: 8px !important;
```
Change to `6px`, `10px`, `12px`, etc.

### Change Primary Color (Focus States)
Find `#8c02e8` in the CSS and replace with your brand color:
```css
border-color: #8c02e8 !important; /* Change this */
box-shadow: 0 0 0 3px rgba(140, 2, 232, 0.1) !important; /* And this */
```

### Remove Shadows
Remove or comment out:
```css
box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05) !important;
```

### Adjust Padding
Change the padding values:
```css
padding: 12px 16px !important; /* vertical horizontal */
```

---

## Troubleshooting

### CSS Not Applying?

1. **Clear cache** (browser, WordPress, and any caching plugins)
2. **Check specificity** - add more `!important` if needed
3. **Inspect element** (F12) to see if styles are being overridden
4. **Check class names** - make sure `cb-zbrmv6eo` and `cb-hpjdnbq4` match your filters

### How to Find Your Filter's Unique Class:

1. Right-click on the filter on your live page
2. Select **Inspect** (F12)
3. Look for classes starting with `cb-` (e.g., `cb-zbrmv6eo`)
4. Update the CSS with your actual class names if different

### Select Dropdown Not Full Width?

Add this additional CSS:
```css
.gsbp-select-wrap {
    flex: 1 1 auto !important;
    max-width: 350px !important; /* Adjust as needed */
}
```

---

## Advanced: Custom Select Dropdown Style

If you want a more custom select dropdown (like a styled React Select), you can use JetSmartFilters' "Checkbox" filter styled to look like a dropdown, or use a custom select plugin.

However, the CSS provided gives you a clean, rounded, Greenshift-matching style that works with the native select element.
