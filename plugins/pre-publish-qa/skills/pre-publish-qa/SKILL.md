---
name: pre-publish-qa
description: Run pre-publish QA for open-source Node.js apps, including secrets, PII, security, dependency, and documentation checks before release.
---

# Pre-Publish QA

Comprehensive pre-publish quality assurance for open-source Node.js apps. Scans for secrets, PII, security issues, dependency problems, and documentation gaps.

## Triggers
- `/pre-publish-qa` - Main command
- "pre-publish check", "ready to publish?", "open source check"

## Workflow

Run ALL checks in parallel using agents, then compile a unified report.

### Check Categories

#### 1. SECRETS SCAN (Critical)
```bash
# Scan current code for API keys, tokens, passwords
grep -rn --include="*.mjs" --include="*.js" --include="*.json" --include="*.html" \
  -E "(sk-ant-|sk-[a-zA-Z0-9]{20,}|api_key\s*=\s*['\"][^'\"]+|password\s*=\s*['\"][^'\"]+|Bearer [A-Za-z0-9]{10,}|ANTHROPIC_API_KEY\s*=\s*['\"])" \
  src/ public/ knowledge/ 2>/dev/null

# Scan git history for .env files
git log --all --full-history -- ".env" "*.env" ".env.*" 2>/dev/null

# Verify .env is gitignored
grep -q "\.env" .gitignore && echo "PASS: .env in .gitignore" || echo "FAIL: .env NOT in .gitignore"

# Check if .env is tracked
git ls-files .env && echo "FAIL: .env IS tracked" || echo "PASS: .env not tracked"
```

#### 2. PII & PRIVACY SCAN
```bash
# Email addresses in source
grep -rn --include="*.mjs" --include="*.js" --include="*.json" \
  -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" \
  src/ knowledge/ public/ 2>/dev/null

# Phone numbers
grep -rn --include="*.mjs" --include="*.js" --include="*.json" \
  -E "(\+972|05[0-9])[- ]?[0-9]{7,}" \
  src/ knowledge/ public/ 2>/dev/null

# Check committed data files for real client data
git ls-files "knowledge/*.json" "data/*.json"
# Inspect each for personal names, addresses, real business details

# Check if reference documents are committed
git ls-files "document refrences*"

# Check git commit messages for PII
git log --oneline --all | grep -iE "email|phone|client name|password|key"
```

#### 3. DEPENDENCY AUDIT
```bash
# NPM vulnerability audit
npm audit --audit-level=high 2>&1

# Check for outdated packages
npm outdated 2>&1

# License compatibility scan
npx license-checker-rseidelsohn --summary 2>&1

# Verify license field in package.json
grep '"license"' package.json || echo "FAIL: No license in package.json"
```

#### 4. SECURITY SURFACE
```bash
# Express security headers
grep -rn "x-powered-by\|helmet\|Content-Security-Policy" src/

# CORS configuration
grep -rn "cors\|Access-Control-Allow-Origin" src/

# Server binding (should be localhost for desktop app)
grep -rn "listen\|hostname\|host.*:" src/server.mjs

# Input validation — endpoints accepting user input
grep -rn "req\.body\.\|req\.query\.\|req\.params\." src/server.mjs | head -20

# File upload safety
grep -rn "multer\|upload\|formidable" src/ | head -10

# JSON body size limits
grep -rn "json(\|urlencoded(" src/server.mjs

# Path traversal risk in file-serving routes
grep -rn "path\.join.*req\.\|path\.resolve.*req\." src/

# AI max_tokens bounded
grep -rn "max_tokens" src/
```

#### 5. BUILD & PACKAGING
```bash
# .gitignore completeness
for f in ".env" "node_modules" "dist/" "output/"; do
  grep -q "$f" .gitignore && echo "PASS: $f in .gitignore" || echo "FAIL: $f NOT in .gitignore"
done

# No large binaries tracked
git ls-files | grep -E "\.exe$|\.dmg$|\.pkg$|\.zip$|\.tar\.gz$" | head -5

# No node_modules tracked
git ls-files node_modules/ | head -3

# No output/ tracked
git ls-files output/ | head -3

# Test clean build
npm run build 2>&1 | tail -5

# Test app startup without API key
timeout 5 bash -c 'ANTHROPIC_API_KEY="" node src/server.mjs 2>&1' || true
```

#### 5b. EXECUTABLE BUNDLE AUDIT (Critical for packaged apps)
```bash
# Check what gets bundled into the executable (pkg, electron-builder, etc.)
# For @yao-pkg/pkg projects:
if grep -q '"pkg"' package.json 2>/dev/null; then
  echo "=== PKG ASSETS BUNDLED INTO EXE ==="
  node -e "const p=require('./package.json'); console.log((p.pkg?.assets||[]).join('\n'))"

  # CRITICAL: Ensure NO private data files are in pkg assets
  # Only sample/template files should be bundled
  node -e "
    const p=require('./package.json');
    const assets = p.pkg?.assets || [];
    const risky = assets.filter(a =>
      !a.includes('sample') && !a.includes('example') && !a.includes('public') &&
      (a.includes('.json') && !a.includes('package')) ||
      a.match(/data\/|private|secret|\.env/)
    );
    if (risky.length) { console.log('FAIL: Private data in pkg assets:'); risky.forEach(r => console.log('  - ' + r)); }
    else console.log('PASS: No private data in pkg assets');
  "

  # Check logo/branding — is it a placeholder or personal?
  if [ -f "assets/logo.png" ]; then
    SIZE=$(wc -c < assets/logo.png)
    if [ "$SIZE" -lt 500 ]; then
      echo "PASS: logo.png is placeholder ($SIZE bytes)"
    else
      echo "WARN: logo.png is $SIZE bytes — verify it's not personal branding"
    fi
  fi

  # Check sample templates for personal pricing (non-zero = personal)
  if [ -f "knowledge/clauses-db.sample.json" ]; then
    PRICES=$(grep -o '"price":[^,}]*' knowledge/clauses-db.sample.json | grep -v '"price": *0' | grep -v '"price":0')
    if [ -n "$PRICES" ]; then
      echo "WARN: Sample templates have non-zero prices — may contain personal rates:"
      echo "$PRICES"
    else
      echo "PASS: Sample template prices are zeroed out"
    fi
  fi
fi
```

#### 6. DOCS & LICENSING
```bash
# LICENSE file exists
ls LICENSE* 2>/dev/null && echo "PASS" || echo "FAIL: No LICENSE file"

# README.md sections
grep -c "^##" README.md 2>/dev/null || echo "FAIL: No README.md"

# .env.example exists
ls .env.example 2>/dev/null && echo "PASS" || echo "FAIL: No .env.example"

# Check README has key sections
for section in "Install" "Usage" "License" "Configuration"; do
  grep -qi "$section" README.md 2>/dev/null && echo "PASS: README has $section" || echo "WARN: README missing $section section"
done
```

#### 7. CODE QUALITY
```bash
# Debug code
grep -rn --include="*.mjs" "debugger;" src/
grep -rn --include="*.mjs" "console\.debug\|console\.trace" src/

# TODO/FIXME/HACK comments
grep -rn --include="*.mjs" "TODO\|FIXME\|HACK\|XXX\|REMOVEME" src/ | head -20

# Hardcoded dev URLs that shouldn't be there
grep -rn --include="*.mjs" "localhost" src/ | grep -v "//.*localhost" | head -10
```

### Report Format

After all checks complete, compile results into:

```
## Pre-Publish QA Report

### Critical (must fix before publishing)
- [list of blocking issues]

### Warnings (should fix)
- [list of recommended fixes]

### Passed
- [list of checks that passed]

### Summary
X critical / Y warnings / Z passed
```

#### 8. BUILT BINARY SCAN (Critical — runs against the actual exe)
```bash
# Scan the built binary with `strings` for embedded personal data
# This catches things that source-level grep CANNOT (bundled assets, snapshots)
if ls dist/executables/*linux* 2>/dev/null; then
  BINARY=$(ls dist/executables/*linux* | head -1)
  echo "Scanning binary: $BINARY"

  echo "=== Personal names ===" 
  strings -n 6 "$BINARY" | grep -iE "your-name-here|your-company" | head -5
  # Replace "your-name-here" with actual developer name patterns

  echo "=== Secrets ===" 
  strings -n 10 "$BINARY" | grep -iE "sk-ant-|api_key\s*=\s*['\"]|password\s*=" | head -5

  echo "=== Personal emails ===" 
  strings -n 8 "$BINARY" | grep -E "^[a-zA-Z0-9._%+-]+@(gmail|yahoo|hotmail)" | head -5

  echo "=== Personal URLs ===" 
  strings -n 10 "$BINARY" | grep -iE "linkedin\.com/in/|x\.com/[A-Z]" | head -5
fi
```

#### 9. SMOKE TEST (Critical — tests the actual user experience)
```bash
# Run the smoke test against the built binary
# This simulates a FRESH USER in an isolated data directory
if [ -f "e2e/smoke-test-binary.mjs" ]; then
  node e2e/smoke-test-binary.mjs
else
  echo "SKIP: No smoke test found at e2e/smoke-test-binary.mjs"
fi

# The smoke test verifies:
# - Binary starts and serves HTTP
# - No personal data in served HTML or API responses
# - Fresh user gets empty profile (setupComplete: false)
# - Sample clauses loaded (not personal clauses)
# - Template prices are zero (not personal rates)
# - Profile can be updated and persists
# - Chat handles missing API key gracefully
# - No x-powered-by header
# - All key endpoints respond
```

## Rules

1. **Run ALL checks** — don't skip any category
2. **Critical = blocks publish** — secrets, PII, tracked .env, license issues, binary scan failures
3. **Warning = should fix** — debug code, missing docs, outdated deps
4. **Pass = good to go** — clean checks
5. **Always show the full report** — don't summarize away important details
6. **Test the BINARY, not just source** — source-level checks miss bundled assets
7. **Test in isolation** — use CONTRACTOR_DATA_DIR env var to avoid polluting real data
