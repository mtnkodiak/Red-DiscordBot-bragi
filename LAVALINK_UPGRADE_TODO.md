# Lavalink Upgrade TODO

## Critical: API v3 Deprecation Warning

**Date Identified:** 2026-08-18

### Issue
Lavalink is warning that websocket commands are deprecated and will be removed in API v4. API v3 will be removed in Lavalink 5.

```
Sending websocket commands to Lavalink has been deprecated and will be removed in API version 4.
API version 3 will be removed in Lavalink 5.
Please use the new REST endpoints instead.
```

### Current Setup
- **Red-DiscordBot Version:** 3.5.25.dev62+g81c4198f.dirty
- **Lavalink Version:** 3.7.13 (red=5)
- **Red-Lavalink (Python client):** 0.11.1
- **YouTube Plugin:** 1.18.2

### Action Required
Red-DiscordBot's audio cog is using the deprecated websocket API to communicate with Lavalink. This needs to be upgraded to use REST endpoints before Lavalink v5.

### Tasks
- [ ] Check if a newer version of Red-Lavalink (Python library) supports Lavalink v4 REST API
- [ ] Upgrade Red-DiscordBot to a version that supports Lavalink v4
- [ ] Test audio functionality after upgrade
- [ ] Update Lavalink JAR to v4.x when Red-DiscordBot supports it

### References
- Red-Lavalink library: https://github.com/Cog-Creators/Red-Lavalink
- Lavalink v4 changelog: https://lavalink.dev/changelog/v4
- Red-DiscordBot audio cog: `/home/cj/projects/cj-red/Red-DiscordBot/redbot/cogs/audio/`

### Priority
**Medium** - Not immediately breaking, but should be addressed before Lavalink v5 is released.

---

## YouTube Playback Issues ⚠️ UNRESOLVED

### Current Status
OAuth2 has been configured with a valid refresh token, but YouTube playback still fails with multiple errors.

### Root Cause
YouTube is actively blocking/rate-limiting requests even with OAuth2. All client types are failing:
- **WEB_EMBEDDED_PLAYER**: "Video player configuration error"
- **ANDROID_VR/ANDROID_MUSIC**: "This video requires login"
- **IOS**: "Invalid status code 400"
- **WEB**: "No supported audio streams available"
- **MWEB**: Signature cipher extraction failure

### Current Configuration
- **YouTube Plugin**: 1.18.2 (latest available as of 2026-08-18)
- **OAuth2**: ✅ Enabled (refresh token must be added manually - see setup below)
- **Config Location**: `redbot/cogs/audio/managed_node/ll_server_config.py`

### OAuth2 Setup (For Fresh Clones)

**IMPORTANT**: The refresh token is NOT committed to the repo for security reasons.

After cloning this repo, you need to set up OAuth2:

1. **Start the bot once** - it will prompt for OAuth
   ```bash
   sudo systemctl restart bragibot
   tail -f /var/log/syslog | grep OAuth
   ```

2. **Look for the OAuth prompt** in logs:
   ```
   OAUTH INTEGRATION: To give youtube-source access to your account,
   go to https://www.google.com/device and enter code XXXX-XXXX-XXXX
   ```

3. **Complete OAuth flow**:
   - Visit https://www.google.com/device
   - Enter the code from logs
   - Sign in with a **burner Google account** (not your main account!)
   - Grant permissions

4. **Get the refresh token** from logs:
   ```
   OAUTH INTEGRATION: Token retrieved successfully.
   Store your refresh token: 1//04xxxxxxxxxxxxx
   ```

5. **Add token to config**:
   Edit `redbot/cogs/audio/managed_node/ll_server_config.py` line 101:
   ```python
   "yaml__plugins__youtube__oauth__refreshToken": "YOUR_TOKEN_HERE",
   ```

6. **Restart bot**:
   ```bash
   sudo systemctl restart bragibot
   ```

### Attempted Fixes
1. ✅ Updated YouTube plugin from 1.18.1 → 1.18.2
2. ✅ Configured OAuth2 with valid Google account
3. ✅ Added refresh token to persist across restarts
4. ❌ **Still failing** - OAuth doesn't resolve YouTube's blocking

### Why OAuth Didn't Fix It
- OAuth helps with rate limiting but doesn't bypass YouTube's anti-bot measures
- YouTube actively detects and blocks automated access patterns
- The signature cipher changes frequently, breaking extraction
- Some clients don't properly support OAuth in plugin v1.18.2

### Possible Solutions (Priority Order)

#### 1. Monitor for Plugin Updates (Recommended)
- Watch: https://github.com/lavalink-devs/youtube-source/releases
- YouTube breaks plugins regularly; fixes come within days/weeks
- Check Maven for newer versions: https://maven.lavalink.dev/releases/dev/lavalink/youtube/youtube-plugin/

#### 2. Use Alternative Music Sources
- **SoundCloud**: Working fine in current setup
- **Spotify**: Requires Spotify API credentials (uses YouTube as backend)
- **Local files**: Upload MP3s to `localtracks` folder
- **Bandcamp, Vimeo**: Also configured and working

#### 3. Try YouTube Music URLs Instead
- `!play https://music.youtube.com/...` instead of regular YouTube
- May use different API endpoints

#### 4. Wait It Out
- YouTube playback breakage is cyclical
- Usually fixed within 1-2 weeks as plugin devs respond

#### 5. Use External Lavalink Service (Advanced)
- Host Lavalink on residential IP (not datacenter)
- Use paid Lavalink hosting service
- May have better success avoiding YouTube blocks

### Monitoring
Check for new plugin versions weekly:
```bash
curl -s "https://maven.lavalink.dev/releases/dev/lavalink/youtube/youtube-plugin/" | grep -oP '1\.[0-9]+\.[0-9]+' | sort -V | tail -5
```

### Last Successful OAuth Setup
- Date: 2026-08-18 19:30:29
- Token retrieved successfully
- Token is being used by plugin (no re-authentication prompts)
