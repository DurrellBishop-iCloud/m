# Future Implementation: One-Time Sharing Links

## The Problem

The current mask sharing approach has privacy/safety concerns:

1. **Guessable group names** - URLs like `?group=kids` are enumerable
2. **Permanent public URLs** - jsDelivr CDN serves images permanently
3. **No access control** - Anyone with the URL pattern can access images
4. **No expiry** - Links live forever
5. **Abuse potential** - Could be used to share photos of faces (including children)

For a kids-safe app promoted on Instagram, this is problematic.

---

## Options Considered

### Option A: Local Pack Sharing (No Server)
- Export masks as files, share via AirDrop/iMessage
- Most private, zero liability
- **Downside**: UX friction - users must save file, then manually import

### Option B: R2 + Signed URLs (Proper Server)
- Private storage with expiring signed URLs
- Good UX but still "hosting content"

### Option C: One-Time Links (Recommended Hybrid)
- Temporary server storage
- Links expire after first use OR 10 minutes
- Auto-delete after download
- Best balance of UX and privacy

---

## One-Time Link Architecture

### The Flow

1. **Sender creates share**
   - App uploads a "package" (JSON with base64 masks) to temporary storage
   - Worker returns a link: `dbgh.uk/m?get=<token>`

2. **Recipient taps link**
   - App calls `GET /m?get=<token>`
   - Worker returns the package
   - Worker immediately invalidates token and deletes blob

3. **Recipient imports**
   - App stores masks locally
   - Done - nothing persists on server

### API Endpoints

```
POST /share
- Body: { masks: [...], groupName: "..." }
- Creates package in R2
- Stores token metadata in KV
- Returns: { url: "dbgh.uk/m?get=<token>", code: "7X2K" }

GET /get?token=<token>
- Validates token exists and not expired
- Streams package to client
- Immediately marks token as consumed
- Deletes blob from R2
- Returns 410 Gone if already used
```

### Token Properties

- 128-bit+ random (22+ chars base64url)
- Not derived from group names
- Not guessable or enumerable
- Atomic one-time use (prevent race conditions)

### Expiry Rules

- Expire after **10 minutes** from creation
- Also expire after **first successful download**
- Cleanup cron to delete orphaned blobs

### Critical Headers (No Caching)

```
Cache-Control: no-store, max-age=0
Pragma: no-cache
Expires: 0
```

### Storage

- **R2**: Blob storage for mask packages
- **KV**: Token metadata (already exists for rate limiting)
- **D1** (optional): If need more complex token bookkeeping

### Security Considerations

1. Don't log query strings (token leakage)
2. No edge caching for share routes
3. Redact tokens in error reporting
4. Rate limit share creation

---

## Mask Pack Format

```json
{
  "version": 1,
  "name": "Ben's Masks",
  "created": "2025-01-26T12:00:00Z",
  "expires": "2025-01-26T12:10:00Z",
  "masks": [
    {
      "id": "mask-123",
      "name": "Monster",
      "image": "data:image/png;base64,iVBOR...",
      "anchors": {
        "leftEye": { "x": 0.4, "y": 0.5 },
        "rightEye": { "x": 0.6, "y": 0.5 }
      }
    }
  ]
}
```

### Size Estimates

- Each mask (1024px max): ~50-200KB PNG
- Typical share (5-10 masks): ~500KB - 2MB
- Well within R2/KV limits

---

## What's Already In Place

- Cloudflare Worker (`mask-worker`)
- KV namespace (for rate limiting)
- Rate limiting logic (40/hour/IP)
- File size checking

## What Would Need Adding

- R2 bucket for temporary blob storage
- Token generation and validation
- Cleanup cron trigger
- Privacy policy page
- "Don't upload real faces" messaging

---

## Legal Position

This approach is closer to a "temporary transfer service" than a "publishing platform", which lowers risk. However, you still need:

- Privacy policy
- Deletion policy
- Abuse/report channel (even just an email)
- Minimal data collection
- Clear statement about not uploading photos of real people

---

## Alternative: Short Code Entry

Instead of (or in addition to) links, offer code entry:

- Sender sees: `MASK-7X2K`
- Recipient opens app → "Enter Code" → types `7X2K`
- Same backend, different UX
- Works better for verbal sharing

---

## References

- Cloudflare R2: https://developers.cloudflare.com/r2/
- Cloudflare KV: https://developers.cloudflare.com/kv/
- Web Share API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Share_API

---

*Document created: 2025-01-26*
*Status: Not implemented - saved for future consideration*
