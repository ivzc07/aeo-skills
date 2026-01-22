---
name: aeo-failure-patterns
description: Recognize common errors and apply known fixes automatically. Hybrid: core patterns + project-specific learning.
---

# AEO Failure Patterns

**Purpose:** Recognize common errors and apply known fixes automatically. Uses hybrid architecture: core curated patterns + project-specific learned patterns.

## Architecture

**Core Patterns (in this SKILL.md):**
- 20-30 curated patterns with high-confidence fixes
- Portable across projects
- Confidence ≥ 0.85 → Auto-fix

**Project-Specific Patterns (in PAI memory):**
- Learned from human resolutions
- `$PAI_DIR/MEMORY/aeo-failure-patterns.json`
- Confidence ≥ 0.80 → Auto-fix

## When to Use

Invoke when:
1. Error occurs during task execution
2. Test fails
3. Build fails
4. Runtime exception

## Core Patterns

### 1. Module Not Found

**Error Signature:**
```
Error: Cannot find module 'module-name'
or
SyntaxError: Named export 'ExportName' not found
```

**Fix:**
```bash
# Check if module installed
ls node_modules | grep module-name

# If missing, install
npm install module-name

# If wrong import, check package.json exports
cat node_modules/module-name/package.json | grep exports
```

**Confidence:** 0.90

---

### 2. Type Error: Property Missing

**Error Signature:**
```typescript
TypeError: Cannot read properties of undefined (reading 'propertyName')
or
Property 'propertyName' does not exist on type 'Type'
```

**Fix:**
```typescript
// Add optional chaining
object?.propertyName

// Or add null check
if (object && object.propertyName) { ... }

// Or add type guard
if ('propertyName' in object) { ... }
```

**Confidence:** 0.85

---

### 3. Dependency Version Conflicts

**Error Signature:**
```
npm ERR! peer dep missing: module@version required by package@version
or
npm ERR! Conflicting peer dependency
```

**Fix:**
```bash
# Use --legacy-peer-deps flag
npm install --legacy-peer-deps

# Or use --force
npm install --force

# Or resolve manually
npm install module@required-version
```

**Confidence:** 0.80

---

### 4. Async Callback Timeout

**Error Signature:**
```jest
Error: Timeout - Async callback was not invoked within the 5000ms timeout
```

**Fix:**
```javascript
// Add done callback
test('async test', (done) => {
  asyncFunction()
    .then(result => {
      expect(result).toBe(value)
      done()  // ← Required
    })
    .catch(err => {
      done(err)
    })
})

// Or use async/await
test('async test', async () => {
  const result = await asyncFunction()
  expect(result).toBe(value)
})
```

**Confidence:** 0.90

---

### 5. Stack Overflow (Maximum Call Stack)

**Error Signature:**
```
RangeError: Maximum call stack size exceeded
```

**Fix:**
```javascript
// Likely infinite recursion - check base case

function recursive(data) {
  // ❌ Missing base case
  return recursive(data.processed())

  // ✅ Add base case
  if (data.isComplete()) return data
  return recursive(data.processed())
}
```

**Confidence:** 0.85

---

### 6. Permission Denied (File System)

**Error Signature:**
```bash
Error: EACCES: permission denied, open 'path/to/file'
```

**Fix:**
```bash
# Check file permissions
ls -la path/to/file

# If owned by different user, use sudo
sudo chown $USER:$USER path/to/file

# Or fix directory permissions
chmod 755 path/to/directory
```

**Confidence:** 0.95

---

### 7. Configuration File Not Found

**Error Signature:**
```bash
Error: ENOENT: no such file or directory, open '.env'
or
Config file not found: config.json
```

**Fix:**
```bash
# Copy example config
cp .env.example .env

# Or create from template
cat > .env << EOF
API_KEY=your-key-here
DATABASE_URL=your-database-url
EOF
```

**Confidence:** 0.90

---

### 8. Git Merge Conflicts

**Error Signature:**
```bash
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and commit
```

**Fix:**
```bash
# Find conflicted files
git status | grep "both modified"

# Edit file, resolve markers:
# <<<<<<< HEAD
# your changes
# =======
# their changes
# >>>>>>> branch-name

# After resolving:
git add file.txt
git commit
```

**Confidence:** 0.85

---

### 9. Unrelated Histories (Git)

**Error Signature:**
```bash
fatal: refusing to merge unrelated histories
```

**Fix:**
```bash
git merge branch-name --allow-unrelated-histories
```

**Confidence:** 0.95

---

### 10. Port Already in Use

**Error Signature:**
```bash
Error: listen EADDRINUSE: address already in use :::3000
```

**Fix:**
```bash
# Find process using port
lsof -i :3000

# Kill it
kill -9 <PID>

# Or use different port
PORT=3001 npm start
```

**Confidence:** 0.95

---

### 11. CORS Error

**Error Signature:**
```
Access to fetch at 'http://api.example.com' from origin 'http://localhost:3000' has been blocked by CORS policy
```

**Fix:**
```javascript
// Server-side: Add CORS headers
import cors from 'cors'
app.use(cors())

// Or specific origin
app.use(cors({
  origin: 'http://localhost:3000'
}))
```

**Confidence:** 0.90

---

### 12. JSON Parse Error

**Error Signature:**
```javascript
SyntaxError: Unexpected token < in JSON at position 0
```

**Fix:**
```javascript
// Check response before parsing
const response = await fetch(url)
const text = await response.text()

// Check if HTML (error page) or JSON
if (text.startsWith('<')) {
  throw new Error('Received HTML instead of JSON')
}

const data = JSON.parse(text)
```

**Confidence:** 0.85

---

### 13. React Hydration Mismatch

**Error Signature:**
```
Warning: Text content did not match. Server: "X" Client: "Y"
```

**Fix:**
```react
// Ensure server and client render same content
// Check for date(), Math.random(), etc.

// ❌ Wrong - different on server vs client
<div>{Date.now()}</div>

// ✅ Correct - use useEffect for client-only
<div>{typeof window !== 'undefined' && Date.now()}</div>

// Or use useEffect
useEffect(() => {
  setTimestamp(Date.now())
}, [])
```

**Confidence:** 0.85

---

### 14. WebSocket Connection Failed

**Error Signature:**
```javascript
WebSocket connection to 'ws://localhost:3000' failed
```

**Fix:**
```javascript
// Check if WebSocket server is running
// Add connection timeout
const ws = new WebSocket('ws://localhost:3000')

ws.onerror = (error) => {
  console.error('WebSocket error:', error)
}

ws.onclose = (event) => {
  if (event.code === 1006) {
    console.log('WebSocket closed abnormally - server may be down')
  }
}
```

**Confidence:** 0.80

---

### 15. Database Connection Pool Exhausted

**Error Signature:**
```
Error: Connection pool exhausted
```

**Fix:**
```javascript
// Increase pool size
const pool = new Pool({
  max: 20,  // Increase from default
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
})

// Or ensure connections are released
const client = await pool.connect()
try {
  await client.query('SELECT * FROM users')
} finally {
  client.release()  // ← Always release
}
```

**Confidence:** 0.90

---

### 16. YAML Parse Error

**Error Signature:**
```
YAMLException: bad indentation of a mapping entry
```

**Fix:**
```yaml
# ❌ Wrong - inconsistent indentation
key:
  subkey: value
    subsubkey: value

# ✅ Correct - consistent indentation
key:
  subkey: value
  subsubkey: value
```

**Confidence:** 0.95

---

### 17. Cannot Read Headers After Sent

**Error Signature:**
```javascript
Error [ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client
```

**Fix:**
```javascript
// ❌ Wrong - sending response twice
app.get('/api/users', (req, res) => {
  res.json({ users: [] })
  res.json({ count: 0 })  // ← Error
})

// ✅ Correct - return after sending
app.get('/api/users', (req, res) => {
  res.json({ users: [], count: 0 })
  return  // ← Prevent further execution
})

// Or use if/else
app.get('/api/users', (req, res) => {
  if (error) {
    res.status(500).json({ error })
  } else {
    res.json({ users })
  }
})
```

**Confidence:** 0.90

---

### 18. NPM Install Hangs

**Error Signature:**
```bash
npm install hangs forever
```

**Fix:**
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and package-lock.json
rm -rf node_modules package-lock.json

# Reinstall
npm install
```

**Confidence:** 0.85

---

### 19. Docker Container Exits Immediately

**Error Signature:**
```bash
docker run <image> # exits immediately
```

**Fix:**
```dockerfile
# ❌ Wrong - no process keeping container alive
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install

# ✅ Correct - use CMD or ENTRYPOINT
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "start"]  # ← Keeps container alive
```

**Confidence:** 0.90

---

### 20. Test Environment Variables Missing

**Error Signature:**
```javascript
ReferenceError: process.env is not defined
```

**Fix:**
```javascript
// vite.config.js / jest.config.js
export default {
  test: {
    env: {
      process: { env: {} }  // ← Add process.env polyfill
    }
  }
}

// Or set in test setup
process.env.API_KEY = 'test-key'
```

**Confidence:** 0.90

---

## Pattern Matching Flow

1. **Error occurs** → Extract message + stack trace + context
2. **Match core patterns** → If confidence ≥ 0.85 → Auto-fix
3. **Match project patterns** → Read `$PAI_DIR/MEMORY/aeo-failure-patterns.json`
4. **Fuzzy match** → If confidence ≥ 0.70 → Propose, ask confirmation
5. **No match** → Escalate to human

## Max Retry Limit

**Prevent infinite loops:**
- Same error + same fix → Max 2 attempts
- After 2 failures → Escalate
- Reset counter on different error

## Learning New Patterns

When human resolves an error:

```bash
# Extract pattern from resolution
pattern = {
  error_signature: "Error message substring",
  context: { language, framework },
  fix: "Steps taken to fix",
  confidence: 0.50,  # Start low
  success_count: 0,
  failure_count: 0
}

# Write to memory
echo $pattern | jq -c '.' >> ~/.claude/MEMORY/aeo-failure-patterns.json
```

**Promote to auto-fix:**
- 3+ successes → Increase confidence to ≥ 0.80
- Failures → Decrease confidence by 0.20
- Below 0.50 → Remove from active patterns

## Project Pattern Format

```json
{
  "error_signature": "specific error message",
  "context": {
    "language": "javascript",
    "framework": "react"
  },
  "fix": "step-by-step fix instructions",
  "confidence": 0.85,
  "success_count": 5,
  "failure_count": 0,
  "last_seen": "2026-01-22T10:00:00Z"
}
```

## Escalation

If no pattern matches or max retries reached:

```
❌ CANNOT AUTO-FIX - PATTERN NOT RECOGNIZED

Error: [error message]

Searched:
• 20 core patterns - no match
• 15 project patterns - no match

Action: Escalating to human for guidance.
```

Invoke aeo-escalation skill.
