# JetSmartFilters + Greenshift Debugging Checklist

## Issue: Filters showing "no results"

### ✅ Checklist:

1. **Query ID Match**
   - [ ] Greenshift Query Grid ID: `gsbp-drug-grid`
   - [ ] JetSmartFilters Search Query ID: Should be `gsbp-drug-grid` (not `gspb_filterid_gsbp-drug-grid`)
   - [ ] JetSmartFilters Select Query ID: Should be `gsbp-drug-grid` (not `gspb_filterid_gsbp-drug-grid`)

2. **Filter Configuration (Filter ID: 15110 - Select)**
   - [ ] What is it filtering? (Taxonomy/Meta field name): ______________
   - [ ] Does this field exist on "drugs" post type?
   - [ ] Data Source set to "drugs" post type?

3. **Filter Configuration (Filter ID: 15111 - Search)**
   - [ ] Search fields configured (post title, content, etc.)?
   - [ ] Query type matches?

4. **Indexer**
   - [ ] Go to JetSmartFilters > Settings > Indexer
   - [ ] Click "Update Filters Index"
   - [ ] Wait for completion

5. **Post Type**
   - [ ] Custom post type slug: "drugs"
   - [ ] At least one published post exists
   - [ ] Posts have the taxonomies/meta fields you're filtering by

6. **Content Provider**
   - [ ] Both filters set to "greenshift-query-loop" or "Custom Query"

7. **Test Without Filters**
   - [ ] Does the Query Loop show posts WITHOUT filters applied?
   - [ ] If yes → filter configuration issue
   - [ ] If no → query loop issue

## Common Fixes:

### Fix 1: Query ID Mismatch (Most Common)
Remove the `gspb_filterid_` prefix from JetSmartFilters Query ID field

### Fix 2: Missing Indexer
Run the indexer in JetSmartFilters settings

### Fix 3: Wrong Filter Field
Make sure the taxonomy/meta field you're filtering actually exists and has values

### Fix 4: Post Type Mismatch
Verify the custom post type slug is exactly "drugs"

## Test Commands:

1. Clear cache (if using caching plugin)
2. Test in incognito window
3. Check browser console for JavaScript errors (F12)
4. Check Network tab for AJAX requests when filtering
