# GameHub Static API

Privacy-respecting static JSON API for the GameHub Android app. This repository hosts all the configuration files, component manifests, and mock responses that were previously served by Chinese servers.

**GitHub Repository:** `https://github.com/gamehublite/gamehub_api`
**Raw URL Base:** `https://raw.githubusercontent.com/gamehublite/gamehub_api/main/`

---

## What This Repository Contains

This is a **static file repository** that stores JSON configuration files and data manifests. It works together with the [Cloudflare Worker](https://github.com/gamehublite/gamehub-api) to provide a complete privacy-respecting API infrastructure.

All files in this repository are served via GitHub's raw content API and consumed by the Cloudflare Worker proxy.

---

## Directory Structure

```
gamehub_api/
├── agreement/              # User agreement files
├── base/
│   └── getBaseInfo         # App base configuration
├── card/                   # Game card/detail endpoints (mostly empty)
├── cloud/
│   └── game/
│       └── check_user_timer  # Cloud sync timer (empty)
├── components/             # Component download manifests
│   ├── box64_manifest      # Box64 emulator versions (13 items)
│   ├── drivers_manifest    # GPU drivers (50+ items)
│   ├── dxvk_manifest       # DirectX to Vulkan layers
│   ├── vkd3d_manifest      # Direct3D 12 to Vulkan
│   ├── games_manifest      # Pre-configured game profiles
│   ├── libraries_manifest  # Windows libraries for Wine
│   └── steam_manifest      # Steam integration files
├── devices/                # Device compatibility info
├── email/                  # Email endpoints (empty)
├── ems/                    # EMS endpoints (empty)
├── game/
│   ├── getDnsIpPool        # DNS pool (empty - allows direct Steam connections)
│   ├── getSteamHost/
│   │   └── index           # Steam CDN IPs (hosts file format)
│   └── ...                 # Other game-related endpoints
├── simulator/
│   └── v2/
│       └── getComponentList  # Component list endpoint
├── upgrade/                # App update info
└── user/                   # User endpoints (empty)
```

---

## Component Manifests

The most important files in this repository are the **component manifests**. These JSON files list all downloadable components for the GameHub app.

### Component Types

#### 1. Box64 Manifest (`components/box64_manifest`)
**Type:** 1
**Purpose:** Box64 and FEX emulator builds for x86_64 game emulation on ARM64 devices
**Total Items:** 13

Contains versions like:
- Box64-0.38 (4.1 MB)
- Box64-0.37-b2 (4.3 MB)
- Box64-0.37-b1 (4.2 MB)
- FEX-20251025 (11.0 MB)
- FEX-20250910 (11.6 MB)
- FEX-20250823 (11.7 MB)
- And older versions...

#### 2. Drivers Manifest (`components/drivers_manifest`)
**Type:** 2
**Purpose:** GPU-specific drivers and Mesa builds
**Categories:**
- Mali drivers (Panfrost, Bifrost)
- Adreno drivers (Freedreno, Turnip)
- PowerVR drivers

#### 3. DXVK Manifest (`components/dxvk_manifest`)
**Type:** 3
**Purpose:** DirectX 9/10/11 to Vulkan translation layers

#### 4. VKD3D Manifest (`components/vkd3d_manifest`)
**Type:** 4
**Purpose:** Direct3D 12 to Vulkan translation layers

#### 5. Games Manifest (`components/games_manifest`)
**Type:** 5
**Purpose:** Pre-configured Wine prefixes and game-specific configurations

#### 6. Libraries Manifest (`components/libraries_manifest`)
**Type:** 6
**Purpose:** Windows DLLs and libraries for Wine compatibility

#### 7. Steam Manifest (`components/steam_manifest`)
**Type:** 7
**Purpose:** Steam client integration files

---

## Manifest File Format

All component manifests follow this JSON structure:

```json
{
  "code": 200,
  "msg": "Success",
  "data": {
    "type": 1,
    "type_name": "box64",
    "display_name": "Box64 Emulators",
    "total": 13,
    "components": [
      {
        "id": 327,
        "name": "Box64-0.37-b2",
        "display_name": "",
        "version": "1.0.0",
        "version_code": 1,
        "download_url": "https://zlyer-cdn-comps-en.bigeyes.com/ux-landscape/pc_zst/bfb6/b3/91/bfb6b3914b8f7abb792d8acafa676861.tzst",
        "file_md5": "bfb6b3914b8f7abb792d8acafa676861",
        "file_size": "4272182",
        "file_name": "Box64-0.37-b2.tzst",
        "logo": "https://zlyer-cdn-comps-en.bigeyes.com/ux-landscape/game-image/45e6/0d/21/45e60d211d35955bd045aabfded4e64b.png",
        "is_ui": 1,
        "type": 1
      }
      // ... more components
    ]
  }
}
```

**Key Fields:**
- `id` - Unique component identifier
- `name` - Component name
- `download_url` - Direct CDN link (downloads bypass the worker)
- `file_md5` - MD5 checksum for verification
- `file_size` - Size in bytes
- `file_name` - Filename with extension (.tzst typically)
- `type` - Component type (1-7)

---

## Configuration Files

### base/getBaseInfo

App base configuration returned on startup:

```json
{
  "code": 200,
  "msg": "Success",
  "data": {
    "cloud_game_switch": 2,
    "guide_info_img": "https://zlyer-cdn-comps-en.bigeyes.com/...",
    "guide_storage_img": "https://zlyer-cdn-comps-en.bigeyes.com/..."
  },
  "time": "1759483449"
}
```

### game/getSteamHost/index

Steam CDN optimization hosts file (plain text):

```
#steam Start
23.47.27.74         steamcommunity.com
104.94.121.98       www.steamcommunity.com
23.45.149.185       store.steampowered.com
23.47.27.74         api.steampowered.com
23.53.35.201        store.akamai.steamstatic.com
#steam End
# Last Update Time : 2025-06-30 09:55:04
```

### cloud/game/check_user_timer

Empty response (cloud sync feature removed):

```json
{
  "code": 200,
  "msg": "",
  "data": []
}
```

### game/getDnsIpPool

Empty DNS pool (allows direct Steam connections):

```json
{
  "code": 200,
  "msg": "",
  "data": []
}
```

---

## How This Works With The Cloudflare Worker

```
[GameHub App]
     ↓
[Cloudflare Worker] ← Routes requests
     ↓
     ├─→ GitHub (this repository) ← Static data
     ├─→ Chinese Server ← Game metadata (IP hidden)
     └─→ Worker-generated ← Empty responses
```

**Request Flow Example:**

1. App requests: `POST /simulator/v2/getComponentList` (type=1, page=1)
2. Worker fetches: `https://raw.githubusercontent.com/gamehublite/gamehub_api/main/components/box64_manifest`
3. GitHub returns: Full manifest JSON
4. Worker transforms: Paginates and renames "components" → "list"
5. App receives: Page 1 of Box64 components
6. App downloads: Direct from CDN (worker not involved)

---

## Maintenance

### Adding New Components

1. Edit the appropriate manifest file:
   ```bash
   vim components/box64_manifest
   ```

2. Add new component entry to the `components` array:
   ```json
   {
     "id": 346,
     "name": "Box64-0.39-b1",
     "download_url": "https://github.com/gamehublite/gamehub_api/releases/download/Components/Box64-0.39-b1.tzst",
     "file_md5": "abc123...",
     "file_size": "4300000",
     "file_name": "Box64-0.39-b1.tzst",
     "type": 1,
     "version": "1.0.0",
     "version_code": 1,
     "is_ui": 1,
     "logo": "https://github.com/gamehublite/gamehub_api/releases/download/Components/45e60d211d35955bd045aabfded4e64b.png"
   }
   ```

3. Update the `total` count in `data` object

4. Update all API endpoint files that reference this component type:
   - `components/downloads` - Add to downloads array, update total
   - `simulator/v2/getAllComponentList` - Add to list array, update total
   - `simulator/v2/getComponentList` - Add to list array, update total
   - `simulator/executeScript/qualcomm` - Add to components array (if Fex/Box64)
   - `simulator/executeScript/generic` - Add to components array (if Fex/Box64)

5. Upload the component file (.tzst) to GitHub releases:
   ```bash
   # Go to: https://github.com/gamehublite/gamehub_api/releases/tag/Components
   # Upload the .tzst file via the web interface
   ```

6. Commit and push:
   ```bash
   git add .
   git commit -m "Add Box64 0.38-b1"
   git push origin main
   ```

5. Changes propagate:
   - GitHub: < 10 seconds
   - Cloudflare Worker cache: 5 minutes
   - App receives: Next request after cache expiry

### Updating Configuration

Edit files in `base/`, `game/`, `cloud/`, etc. and push changes. Same propagation timing applies.

---

## File Formats

**JSON Files:**
- All manifests and configs
- Standard JSON with pretty printing
- Use UTF-8 encoding

**Plain Text:**
- `game/getSteamHost/index` (hosts file format)
- Line-separated, no JSON wrapper

---

## Privacy Features

### What This Repository Does NOT Contain

- ❌ User data
- ❌ Analytics tracking
- ❌ Personal information
- ❌ Device fingerprints
- ❌ IP addresses

### What It DOES Contain

- ✅ Public component manifests
- ✅ Open source configuration data
- ✅ CDN download links (publicly accessible)
- ✅ Steam optimization data
- ✅ Empty responses for removed features

---

## CDN Downloads

All component downloads are hosted on external CDN:
```
https://zlyer-cdn-comps-en.bigeyes.com/ux-landscape/pc_zst/...
```

**Important:**
- This repository only provides download URLs
- Actual file downloads are direct from CDN
- Worker and GitHub never see download traffic
- Your IP is not logged by this infrastructure

---

## Response Format

All API responses follow this structure:

```json
{
  "code": 200,           // HTTP-like status code
  "msg": "Success",      // Status message
  "data": { ... },       // Actual response data
  "time": "1759483449"   // Unix timestamp (optional)
}
```

**Status Codes:**
- `200` - Success
- `500` - Error (rare, only for malformed files)

---

## Repository Stats

- **Total Files:** ~50+ JSON files
- **Total Size:** < 1 MB (all text-based)
- **Update Frequency:** As needed (new component releases)
- **Public Access:** Yes (GitHub raw URLs)

---

## Usage

**Direct Access (Not Recommended):**
```bash
curl https://raw.githubusercontent.com/gamehublite/gamehub_api/main/components/box64_manifest
```

**Via Cloudflare Worker (Recommended):**
```bash
curl -X POST https://gamehub-api.secureflex.workers.dev/simulator/v2/getComponentList \
  -H "Content-Type: application/json" \
  -d '{"type": 1, "page": 1, "page_size": 10}'
```

The worker adds:
- Pagination
- CORS headers
- Caching
- Field transformations
- Error handling

---

## Related Repositories

- **Cloudflare Worker:** [gamehub-api](https://github.com/gamehublite/gamehub-api) - API proxy and router
- **APK Modifications:** See main GameHub analysis documentation

---

## Technical Details

**Hosting:** GitHub Pages (raw content API)
**Format:** JSON (UTF-8)
**Caching:** 5 minutes (Cloudflare Worker)
**Access:** Public (no authentication)
**Rate Limits:** GitHub standard limits apply

---

## Changelog

### 2025-10-07
- Initial repository setup
- Added all component manifests
- Added configuration files
- Added empty responses for removed features

---

**For questions about the Cloudflare Worker integration, see the [gamehub-api repository](https://github.com/gamehublite/gamehub-api).**
