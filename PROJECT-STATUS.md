# Durrell Masks - Project Status

## Overview
A mobile web app for creating and sharing face masks using real-time face detection. Users capture textures, draw masks, and share them through group-based collections.

**Live URL:** https://dbgh.uk/m

## Architecture

```
Frontend (index.html)     →     Cloudflare Worker     →     GitHub (storage)
   ~5300 lines                  mask-worker.dbgh.workers.dev      + jsDelivr CDN
   MediaPipe face detection
```

## Recent Session Changes (Jan 27, 2026)

### Security Fix: Admin Authentication
- **Problem:** Admin code `durrell310863` was hardcoded in public GitHub repo
- **Solution:** Moved to Cloudflare Worker environment variable
- Admin code now stored in `ADMIN_SECRET` env variable (encrypted)
- Frontend verifies via `/auth/verify` endpoint
- Code stored in user's localStorage after verification (not in public code)

### iPad Support Added
- iPadOS 13+ reports as "Macintosh" - now detected via touch support
- `navigator.maxTouchPoints > 1` check added

### Title Image Caching Fix
- jsDelivr CDN was aggressively caching old images
- Switched title images to use `raw.githubusercontent.com` instead

### Admin Panel Improvements
- Increased group list height (200px → 400px for picker, 300px → 450px for admin)
- Added **Delete** button for groups (red, with double confirmation)
- Shows "Unrevoke" when group is already revoked
- Shortened "Copy Link" to "Copy" to fit better

## URLs

| URL | Purpose |
|-----|---------|
| `https://dbgh.uk/m` | Main app |
| `https://dbgh.uk/m?admin=YOURCODE` | Admin login |
| `https://dbgh.uk/m?title=admin` | Title screen admin (need admin login first) |
| `https://dbgh.uk/m?group=g_XXX` | View public group |
| `https://dbgh.uk/m?friend=f_XXX` | View friend group |

## Cloudflare Worker

**Location:** https://dash.cloudflare.com → Workers & Pages → mask-worker

### Environment Variables
| Name | Type | Purpose |
|------|------|---------|
| `ADMIN_SECRET` | Secret | Admin password |
| `GITHUB_TOKEN` | Secret | GitHub API access |
| `GITHUB_REPO` | Plaintext | `DurrellBishop-iCloud/m` |
| `GITHUB_BRANCH` | Plaintext | `main` |

### API Endpoints
| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| POST | `/auth/verify` | None | Verify admin code |
| GET | `/groups` | Admin | List all groups |
| GET | `/groups/public` | None | List public groups |
| POST | `/group/create` | Admin | Create new group |
| POST | `/group/revoke` | Admin | Revoke/unrevoke group |
| POST | `/group/delete` | Admin | Delete group permanently |
| GET | `/masks?group=X` | Token/Admin | Get group masks |
| POST | `/upload` | Token/Admin | Upload mask |
| DELETE | `/mask` | Admin | Delete single mask |
| GET | `/title?group=X` | None | Get title config |
| POST | `/title` | Admin | Upload title image |
| DELETE | `/title` | Admin | Remove title |

## Group Types

- **Local** (`local`) - Browser localStorage only, private to device
- **Public** (`g_XXXX`) - Anyone with link can view, admin can edit
- **Friend** (`f_XXXX`) - Token holders can view and edit, admin can manage

## Title System

| Button | Path | Shows for |
|--------|------|-----------|
| Default | `title/title-image.png` | Public-masks + fallback for all groups |
| Local | `title/local-title-image.png` | Local-masks only |
| Group | `groups/{id}/title-image.png` | That specific group |

## Key Files

```
/m
├── index.html              # Main app (all frontend code)
├── PROJECT-STATUS.md       # This file
├── NOTES.md               # Development gotchas
├── FUTURE-ONE-TIME-LINKS.md # Planned feature
├── title/
│   ├── title.json         # Default title config
│   ├── title-image.png    # Default title image
│   ├── local-title.json   # Local title config
│   └── local-title-image.png
├── groups/
│   ├── g_*/               # Public groups
│   ├── f_*/               # Friend groups
│   └── public/            # Legacy public group
└── durrell-masks/         # Owner's personal masks
```

## Common Tasks

### Change Admin Password
1. Go to Cloudflare dashboard → Workers → mask-worker → Settings
2. Edit `ADMIN_SECRET` variable
3. Use new password in URL: `?admin=NEWPASSWORD`

### Edit Worker Code
1. Cloudflare dashboard → Workers → mask-worker
2. Click "Edit code" (top right)
3. Make changes
4. Click "Deploy"

### Delete a Group
1. Login as admin: `?admin=YOURCODE`
2. Tap group name → Admin Panel
3. Find group → tap red "Delete" button
4. Confirm twice

## Known Issues

- jsDelivr caching can delay mask image updates (not title images - those use raw GitHub now)
- "Public-masks" is both a legacy group folder AND the default title target (can be confusing)

## Contributors
- DurrellBishop-iCloud (owner)
- claude (Worker commits via GitHub API)
