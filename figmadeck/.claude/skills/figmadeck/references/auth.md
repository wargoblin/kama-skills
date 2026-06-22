# Figma Authentication (Playwright)

Figmadeck uses Playwright to authenticate with Figma for export operations. Credentials are stored locally, never in the repository.

## Credential Storage

File: `~/.claude/figmadeck-auth.json`

This file is stored in the user's home directory (outside any git repository). It contains Playwright browser cookies/session tokens — NOT plain-text passwords.

## First Run Flow

1. Check if `~/.claude/figmadeck-auth.json` exists
2. If NO → ask the user:
   ```
   Figma authentication required for export.
   Email: [wait for input]
   Password: [wait for input]
   ```
3. Launch Playwright (chromium):
   ```
   Navigate to https://www.figma.com/login
   Fill email field → submit
   Fill password field → submit
   ```
4. If 2FA form appears:
   ```
   Two-factor authentication detected.
   Enter your 2FA code: [wait for input]
   ```
   Fill 2FA field → submit
5. If login succeeds (redirected to figma.com dashboard or file):
   - Extract cookies: `context.cookies()`
   - Save to `~/.claude/figmadeck-auth.json`:
     ```json
     {
       "cookies": [...],
       "savedAt": "ISO-8601 timestamp"
     }
     ```
   - Print: `✓ Figma authentication successful. Session saved.`
6. If login fails (still on login page, error message visible):
   - Print error: `✗ Login failed: <error text from page>. Please try again.`
   - Return to step 2

## Subsequent Runs

1. Read `~/.claude/figmadeck-auth.json`
2. Launch Playwright, add saved cookies: `context.addCookies(saved.cookies)`
3. Navigate to `https://www.figma.com` (not login page)
4. Check: is the page the dashboard or a file? (Not redirected to /login?)
5. If YES → session is alive → return success
6. If NO (redirected to /login) → session expired:
   - Print: `⚠ Figma session expired. Re-authentication required.`
   - Delete `~/.claude/figmadeck-auth.json`
   - Run First Run Flow

## Timeouts and Error Handling

**Navigation timeout:** All Playwright navigation calls use `{ timeout: 30000 }` (30 seconds).

**Distinguish error types:**
- **Network error**: page didn't load (timeout, DNS failure, connection refused) → NOT an auth problem
- **Auth failure**: page loaded, login form shows error message → wrong credentials
- **2FA failure**: 2FA form shows error after code submission → wrong code

**Retry policy:**
- Network errors: retry up to 3 times with 10-second pause between attempts
- Auth failures: ask for new credentials, max 3 attempts total
- 2FA failures: ask for new code, max 3 attempts

**Hard stop:** If 3 attempts all fail → error: `"Unable to authenticate after 3 attempts. Check your credentials and network connection."` Do NOT loop forever.

## Usage in Pipeline

Auth check is the FIRST step of any operation that needs Playwright:
- `/figmadeck --export` — always needs auth
- `/figmadeck <url> <outline>` — needs auth only if export is requested after generation

Auth is NOT needed for `use_figma` MCP calls — those use the Figma MCP server's own authentication, not Playwright.

## Security Notes

- NEVER commit `~/.claude/figmadeck-auth.json` — it's outside the repo by design
- NEVER store plain-text passwords — only browser cookies
- NEVER log cookie values — only log success/failure status
- Cookies may expire (typically 30 days) — the subsequent run flow handles re-auth automatically
