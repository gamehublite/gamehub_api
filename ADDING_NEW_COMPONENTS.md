# Adding New Components to GameHub Static API

This guide explains how to add new components (drivers, libraries, etc.) to the GameHub static API hosted on GitHub.

## Overview

The GameHub app uses a static API hosted on GitHub to serve component information. When new components are released, they need to be added to multiple JSON files to ensure they work properly in the app.

## Prerequisites

- HAR file from HTTP Toolkit capturing GameHub API traffic
- Access to `/Users/ghlite/Downloads/gamehub_api/` repository
- `jq` command-line tool for JSON processing

## Step 1: Extract New Components from HAR File

### Find the HAR File
HAR files are typically stored in:
```
/Users/ghlite/Desktop/dec/HAR/
```

### Extract Components by Type

```bash
# Extract all drivers (type=2) from HAR file
jq -r '.log.entries[].response.content.text' /path/to/file.har | \
  grep -v "^null$" | head -1 | jq -r '.' | \
  jq -r '.data.list[] | select(.type == 2) | "\(.id)|\(.name)|\(.file_md5)|\(.file_size)"' | \
  sort -t'|' -k1,1n > /tmp/har_drivers.txt
```

**Component Types:**
- Type 1: Box64/FEX Emulators
- Type 2: GPU Drivers
- Type 3: DXVK Layers
- Type 4: VKD3D Proton
- Type 5: Game Patches
- Type 6: System Libraries
- Type 7: Steam Components

### Compare with Existing Components

```bash
# Extract existing drivers from manifest
jq -r '.data.components[] | "\(.id)|\(.name)|\(.file_md5)|\(.file_size)"' \
  /Users/ghlite/Downloads/gamehub_api/components/drivers_manifest | \
  sort -t'|' -k1,1n > /tmp/manifest_drivers.txt

# Find differences (new drivers)
diff /tmp/manifest_drivers.txt /tmp/har_drivers.txt
```

## Step 2: Files That Need to be Updated

When adding new components, you MUST update these files:

### 1. Component-Specific Manifest
**Path:** `/Users/ghlite/Downloads/gamehub_api/components/{type}_manifest`

Example for drivers: `components/drivers_manifest`

**What to update:**
- Update `"total"` count (e.g., 61 → 64)
- Add new component entries with all fields

**Component Entry Structure:**
```json
{
  "id": 338,
  "name": "8eGen5-842.8",
  "type": 2,
  "version": "1.0.0",
  "version_code": 1,
  "file_md5": "f69bfda11b5ada8e6542e94896570193",
  "file_size": "12568832",
  "download_url": "https://zlyer-cdn-comps-en.bigeyes.com/ux-landscape/pc_zst/f69b/fd/a1/f69bfda11b5ada8e6542e94896570193.tzst",
  "file_name": "8eGen5-842.8.tzst",
  "display_name": "",
  "is_ui": 1,
  "logo": "https://zlyer-cdn-comps-en.bigeyes.com/ux-landscape/game-image/45e6/0d/21/45e60d211d35955bd045aabfded4e64b.png"
}
```

**Important:** Group similar components together (e.g., turnip with turnip, 8Elite with 8Elite)

### 2. Components Index
**Path:** `/Users/ghlite/Downloads/gamehub_api/components/index`

**What to update:**
- Update `"total_components"` (e.g., 256 → 259)
- Update category `"count"` for the specific type (e.g., drivers: 61 → 64)

```json
{
  "data": {
    "total_components": 259,
    "categories": [
      {
        "type": 2,
        "name": "drivers",
        "count": 64  // Update this
      }
    ]
  }
}
```

### 3. Downloads File
**Path:** `/Users/ghlite/Downloads/gamehub_api/components/downloads`

**What to update:**
- Update `"total"` count (e.g., 256 → 259)
- Add new entries to the `"list"` array

**Important:** This file has a DIFFERENT structure - NO `id` field!

```json
{
  "name": "8eGen5-842.8",
  "type": 2,
  "version": "1.0.0",
  "download_url": "https://zlyer-cdn-comps-en.bigeyes.com/ux-landscape/pc_zst/f69b/fd/a1/f69bfda11b5ada8e6542e94896570193.tzst",
  "file_name": "8eGen5-842.8.tzst",
  "file_size": "12568832",
  "file_md5": "f69bfda11b5ada8e6542e94896570193"
}
```

### 4. Get All Component List (CRITICAL!)
**Path:** `/Users/ghlite/Downloads/gamehub_api/simulator/v2/getAllComponentList`

**What to update:**
- Update `"total"` count in TWO places (top and bottom of file)
- Add new component entries to the `"list"` array
- Group with similar components

**Why Critical:** This file is used by the app to validate installed components. If components aren't in this file, they will be automatically removed when the app closes!

```json
{
  "data": {
    "total": 259,  // Update here (top)
    "list": [
      // Add new components here
    ]
    "total": 259   // Update here too (bottom)
  }
}
```

## Step 3: Verification Commands

### Verify Component Counts
```bash
# Check total matches count
cd /Users/ghlite/Downloads/gamehub_api

# For drivers_manifest
jq '.data.total, (.data.components | length)' components/drivers_manifest

# For getAllComponentList
jq '.data.total, (.data.list | length), (.data.total == (.data.list | length))' simulator/v2/getAllComponentList
```

### Verify New Components Exist
```bash
# Check if specific component IDs exist
jq '.data.list[] | select(.id == 338 or .id == 337 or .id == 336) | {id: .id, name: .name}' \
  simulator/v2/getAllComponentList
```

### Count Components by Type
```bash
# Count drivers (type=2)
jq '.data.list[] | select(.type == 2)' simulator/v2/getAllComponentList | jq -s length

# For downloads file (no ID field)
jq '.data.list[] | select(.type == 2)' components/downloads | jq -s length
```

## Step 4: Push Changes to GitHub

```bash
cd /Users/ghlite/Downloads/gamehub_api

git add .
git commit -m "Add new components: [list component names]

Added X new drivers (IDs: 336, 337, 338)
- Updated drivers_manifest: 61 → 64
- Updated components/index: 256 → 259
- Updated components/downloads: 256 → 259
- Updated simulator/v2/getAllComponentList: 256 → 259

"

git push
```

## Step 5: Worker (Optional)

The Cloudflare Worker at `/Users/ghlite/Desktop/dec/worker/src/index.ts` proxies requests to the GitHub static API.

**Cache Settings:**
- 5-minute cache (`cacheTtl: 300`)
- Automatically fetches from GitHub raw CDN

**No changes needed** - Worker automatically serves updated GitHub files after cache expires (5 minutes max).

## Common Issues

### Issue: Components disappear after closing app
**Cause:** Missing from `simulator/v2/getAllComponentList`
**Fix:** Add components to this file and update both total counts

### Issue: Components show in list but clicking goes back
**Cause:** Component detail endpoint not working (shouldn't happen with static files)
**Fix:** Verify component exists in the specific manifest file (e.g., `drivers_manifest`)

### Issue: Wrong component count
**Cause:** Forgot to update total in one of the files
**Fix:** Search for old total and update all occurrences:
```bash
cd /Users/ghlite/Downloads/gamehub_api
grep -r "\"total\": 256" .
# Or for specific category:
grep -r "\"count\": 61" .
```

### Issue: File corruption during edit
**Cause:** Automated reconstruction failed
**Fix:** Use `git restore` to recover, then edit manually:
```bash
git restore simulator/v2/getAllComponentList
# Then edit manually with proper tool
```

## Quick Reference: File Checklist

When adding 3 new drivers (example):

- [ ] `components/drivers_manifest` - Update total, add 3 entries
- [ ] `components/index` - Update total_components and drivers count
- [ ] `components/downloads` - Update total, add 3 entries (no ID field!)
- [ ] `simulator/v2/getAllComponentList` - Update TWO totals, add 3 entries
- [ ] Verify all counts match with `jq` commands
- [ ] Git commit and push
- [ ] Wait 5 minutes for worker cache to expire (or test directly on GitHub raw CDN)

## GitHub Raw CDN URLs

Components are served from:
```
https://raw.githubusercontent.com/[owner]/[repo]/main/components/drivers_manifest
https://raw.githubusercontent.com/[owner]/[repo]/main/simulator/v2/getAllComponentList
```

The worker proxies these URLs with caching.

## Notes

- Always use exact values from HAR file (MD5, file size, URLs)
- Group similar components together for better organization
- The `downloads` file has different structure (no ID field)
- `getAllComponentList` has TWO total fields that must match
- Worker caches for 5 minutes - changes may not be immediate
- Test with direct GitHub raw URLs if worker seems cached

## Example: Adding 3 New Drivers

1. Extract from HAR: IDs 336, 337, 338
2. Update `drivers_manifest`: 61 → 64 drivers
3. Update `components/index`: 256 → 259 total, 61 → 64 drivers
4. Update `components/downloads`: 256 → 259 total
5. Update `simulator/v2/getAllComponentList`: 256 → 259 total (twice!)
6. Verify with jq
7. Git commit and push
8. Wait 5 minutes or refresh cache

---

**Last Updated:** October 2025
