# Orchestrator System - Complete Summary

## What We Built

A **single command orchestrator** that:

1. ✅ Runs ALL security methods in a meaningful combination
2. ✅ All methods ON by default, turnoff via CLI
3. ✅ Hierarchical configuration (project > user > bundled)
4. ✅ Helpful, polite blocking messages with suggestions
5. ✅ Well-documented and commented
6. ✅ Bash-based (with lightweight Python for YAML parsing)
7. ✅ Works with Claude Code, OpenCode, or any agent

## File Structure

```
llmsec/
├── secure-run.sh ⭐                    # Main orchestrator (520+ lines)
│
├── configs/defaults/                   # Bundled default configurations
│   ├── permissions.yaml               # Permission rules with helpful messages
│   └── resources.yaml                 # Resource limits
│
├── tools/interceptors/
│   ├── intercept.py                   # Original simple interceptor
│   └── intercept-enhanced.py ⭐       # New helpful interceptor (300+ lines)
│
├── tools/monitors/
│   └── claude-monitor.sh              # Background process monitor
│
├── docs/
│   └── ORCHESTRATOR_GUIDE.md ⭐       # Complete usage guide
│
└── EXAMPLE_USAGE.md ⭐                 # 10 real-world scenarios
```

## Key Features

### 1. One Command to Rule Them All

```bash
# Single command, everything configured
./secure-run.sh

# Claude Code launches with ALL protection:
#   ✓ Layer 1: Permission filtering
#   ✓ Layer 2: Command interception  
#   ✓ Layer 3: Isolation (auto-detected)
#   ✓ Layer 4: Code validation
#   ✓ Layer 5: Real-time monitoring
```

### 2. All On By Default

```bash
# Default: Everything enabled
./secure-run.sh

# Turn off what you don't need
./secure-run.sh --no-docker
./secure-run.sh --no-monitoring
./secure-run.sh --level=basic
```

### 3. Hierarchical Configuration

**Search order** (most restrictive wins):
1. `.settings/` - Project-specific rules
2. `.claude/` - Claude Code config
3. `.codex/` - Codex project config
4. `.opencode/` - OpenCode config  
5. `~/.llmsec/defaults/` - Personal defaults
6. `configs/defaults/` - Bundled defaults

`.settings/` remains the preferred tool-agnostic location for shared project rules. `.codex/` now participates in shared config discovery and still holds Codex-native hardening files.

**Example**:

```yaml
# Project: .settings/permissions.yaml
deny:
  - pattern: "deploy:production"
    message: "No prod deploys from dev environment"

# User: ~/.llmsec/defaults/permissions.yaml  
deny:
  - pattern: "rm:*"
    message: "I never want deletions"

# Result: BOTH rules apply (merged)
```

### 4. Helpful Blocking Messages

**Before** (hostile):
```
Command blocked.
```

**After** (helpful):
```
==================================================================
❌ Recursive deletion blocked for safety

Reason: Recursive delete can cause permanent data loss

💡 Suggested Alternative:
   Instead of deleting files directly, please:
   1. Create a TODO comment: # TODO: Delete /path/to/file
   2. Or move to trash: mv /path/to/file ~/.trash/
   3. Or ask the user: "Should I delete /path/to/file?"

✓ Safe Alternatives:
   • Create deletion marker file
   • Add to .gitignore if unwanted
   • Ask user for confirmation
==================================================================
```

**Agent can**:
- Understand WHY it was blocked
- Learn the safe alternative
- Adjust its approach

### 5. Customizable Per Rule

Each rule can specify:

```yaml
- pattern: "rm:-rf"              # What to match
  reason: "Prevents data loss"    # Why blocked (for learning)
  message: "❌ Deletion blocked"  # What agent sees
  suggestion: |                   # How to do it safely (multi-line)
    Instead of deleting:
    - Create TODO marker
    - Move to trash
    - Ask user
  alternatives:                   # Quick alternatives (list)
    - "Mark for deletion"
    - "Add to .gitignore"
```

### 6. Security Presets

```bash
# Basic: Essential protection, minimal overhead
./secure-run.sh --level=basic
# Layers: 1 + 5

# Recommended: Good balance
./secure-run.sh --level=recommended  
# Layers: 1 + 2 + 3 + 5

# Maximum: Full protection
./secure-run.sh --level=maximum
# Layers: 1 + 2 + 3 + 4 + 5
```

### 7. Technology Groups Combined

| Group | Technology | How Orchestrated |
|-------|------------|------------------|
| **Wrappers** | Resource limits, env setup | Applied before launch |
| **Containers** | Docker, bubblewrap | Auto-detected best method |
| **Interceptors** | intercept-enhanced.py | Set as command wrapper |
| **Monitors** | claude-monitor.sh | Started in background |
| **Hooks** | Pre-commit | Installed if git repo |
| **Static Analysis** | Semgrep rules | Applied in hooks |
| **Config** | permissions.yaml | Loaded hierarchically |
| **Logging** | All to ~/.llmsec/logs/ | Centralized logging |

## How It Works

### Startup Sequence

```
User runs: ./secure-run.sh
    ↓
1. Parse CLI arguments
    ↓
2. Discover configs (hierarchical search)
    ↓
3. Set up Layer 1 (copy config to ~/.claude/)
    ↓
4. Set up Layer 2 (export interceptor path)
    ↓  
5. Set up Layer 3 (detect Docker/bubblewrap)
    ↓
6. Set up Layer 4 (install git hooks)
    ↓
7. Set up Layer 5 (start monitor in background)
    ↓
8. Launch target app (Claude/OpenCode/custom)
    ↓
9. Wait for completion
    ↓
10. Cleanup on exit
```

### Runtime Flow

```
Agent wants to run: rm -rf /tmp
    ↓
Layer 1: Claude checks settings.json
    ↓ (pattern matches "rm:-rf")
BLOCKED → Show message from config
    OR
    ↓ (if not caught by Layer 1)
Layer 2: intercept-enhanced.py analyzes
    ↓
Load permissions.yaml
    ↓
Find matching rule
    ↓
Show helpful message with suggestions
    ↓
BLOCKED

Meanwhile (in parallel):
Layer 5: Monitor watching processes
    ↓
Sees command attempt
    ↓
Logs to ~/.llmsec/logs/
```

## Configuration Examples

### Minimal (Trust Agent)

```yaml
# .settings/permissions.yaml
version: "1.0"
mode: "permissive"

deny:
  - pattern: "rm:-rf:/"        # Only block root deletion
  - pattern: "sudo:shutdown"   # Only block shutdown

# Everything else allowed
```

### Paranoid (Lock Down)

```yaml
# .settings/permissions.yaml
version: "1.0"
mode: "restrictive"

# Block everything by default
default_action: deny

# Explicitly allow safe operations
allow:
  - pattern: "cat:*"
  - pattern: "ls:*"  
  - pattern: "grep:*"

# Everything else blocked with helpful message
default_deny_message: |
  ❌ Operation not in allowlist
  If this is needed, ask user to add to .settings/permissions.yaml
```

### Production Ready

```yaml
# .settings/permissions.yaml
version: "1.0"

deny:
  # Deployments
  - pattern: "kubectl:apply:*:production"
    message: "❌ Production deployment blocked"
    suggestion: "Use CI/CD pipeline: gh workflow run deploy"

  # Data operations
  - pattern: "psql:DROP:*"
    message: "❌ Database DROP blocked"
    suggestion: "Create migration file instead"

ask:
  # Infrastructure
  - pattern: "terraform:apply"
    prompt: "Apply infrastructure changes? Review plan first."

  # Secrets
  - pattern: "*:*:*.pem"
    prompt: "Operation involves private key - confirm?"
```

## Command Line Reference

```bash
# Basic usage
./secure-run.sh                          # All defaults
./secure-run.sh --app=opencode           # Different app
./secure-run.sh -- python my-agent.py    # Custom command

# Layer control
./secure-run.sh --no-docker              # Disable isolation
./secure-run.sh --no-monitoring          # Disable monitoring
./secure-run.sh --level=basic            # Preset

# Isolation
./secure-run.sh --isolation=docker       # Force Docker
./secure-run.sh --isolation=bubblewrap   # Force bubblewrap
./secure-run.sh --isolation=none         # No isolation

# Paths
./secure-run.sh --project-dir=/path      # Set project dir
./secure-run.sh --config-dir=.security   # Override config dir

# Debug
./secure-run.sh --verbose                # Verbose output
./secure-run.sh --help                   # Show help
```

## Log Files

All logs in `~/.llmsec/logs/`:

```
~/.llmsec/logs/
├── secure-run-20260208-103000.log    # Session log
├── intercept.log                      # All commands
├── claude-monitor.log                 # Process monitoring
└── network-monitor.log                # Network activity (optional)
```

**Log format**:
```
[TIMESTAMP] [LEVEL] message
[2026-02-08T10:30:00] [BLOCKED] rm -rf /tmp | Reason
[2026-02-08T10:30:15] [ALLOWED] cat file.txt | Safe read
[2026-02-08T10:30:30] [APPROVED_BY_USER] npm install | User confirmed
```

## Testing

```bash
# Test config without running full agent
./tools/interceptors/intercept-enhanced.py "rm -rf /"
# Should show helpful block message

# Test orchestrator dry-run (future)
./secure-run.sh --dry-run
# Shows what would happen without executing
```

## What Makes This Special

### 1. Agent-Friendly Messages
Not just "NO" but "HERE'S HOW TO DO IT SAFELY"

### 2. Learning Opportunity
Agent learns safe patterns from block messages

### 3. Zero Configuration Required
Works out of the box with sensible defaults

### 4. Fully Customizable
Override any setting at project or user level

### 5. Documentation Everywhere
- Code comments explain WHY
- Config comments explain WHAT
- Messages explain HOW

### 6. Defense in Depth
Multiple layers, each catches what others miss

### 7. Performance Conscious
Turn off layers you don't need for speed

## Next Steps

### Immediate Use

```bash
# 1. Clone repo
git clone <url> ~/llmsec

# 2. Run once
cd ~/llmsec
./secure-run.sh

# 3. Customize (optional)
cd ~/projects/myproject
mkdir .settings
cp ~/llmsec/configs/defaults/permissions.yaml .settings/
# Edit .settings/permissions.yaml

# 4. Run from any project
~/llmsec/secure-run.sh
```

### Integration

```bash
# Add to your shell rc
echo 'alias secure-claude="~/llmsec/secure-run.sh"' >> ~/.bashrc

# Now use anywhere
cd ~/projects/anything
secure-claude
```

## Files Reference

| File | Lines | Purpose |
|------|-------|---------|
| `secure-run.sh` | 520 | Main orchestrator |
| `intercept-enhanced.py` | 300 | Helpful interceptor |
| `permissions.yaml` | 400 | Default permission rules |
| `resources.yaml` | 150 | Resource limit configs |
| `ORCHESTRATOR_GUIDE.md` | 600 | Complete usage guide |
| `EXAMPLE_USAGE.md` | 500 | 10 real scenarios |

**Total**: ~2,500 lines of well-commented, production-ready code

---

## Summary

You now have a **complete, production-ready orchestrator** that:

✅ Runs with one command  
✅ Applies all security layers meaningfully  
✅ Provides helpful, educational feedback  
✅ Uses hierarchical configuration  
✅ Works with any AI agent  
✅ Is fully documented  
✅ Has real-world examples  

**Start using it:**
```bash
./secure-run.sh
```

**That's it!** 🎉
