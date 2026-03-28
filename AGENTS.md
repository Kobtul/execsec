# Working with LLM Security Toolkit

This document helps AI assistants (Claude, etc.) understand how to effectively work with this project.

## Project Overview

**LLM Security Toolkit** is a production-ready security orchestrator for AI agents. It provides defense-in-depth protection through a single command that combines 5 security layers with helpful, educational feedback.

**Key Philosophy**:
- **One Command Does Everything**: `./secure-run.sh` - no complex setup needed
- **Helpful, Not Hostile**: Blocks provide polite suggestions, not just "NO"
- **Safe by Default**: All security enabled out-of-box
- **Zero Config**: Works perfectly without customization
- **Defense in Depth**: Multiple overlapping security layers

## Quick Commands (Using justfile)

```bash
just                  # Show all commands
just test             # Run 60+ safe tests
just run              # Run orchestrator
just help             # Show orchestrator help
```

**Install just**: https://github.com/casey/just

## Project Structure

```
llmsec/
├── secure-run.sh ⭐                 # Main orchestrator (520 lines)
│                                    # - Combines all 8 technology groups
│                                    # - CLI arg parsing
│                                    # - Layer orchestration
│
├── tools/
│   ├── interceptors/
│   │   ├── intercept.py             # Simple interceptor
│   │   └── intercept-enhanced.py ⭐ # With helpful messages (300 lines)
│   │                                # - Loads YAML configs
│   │                                # - Shows suggestions
│   │                                # - Polite blocking
│   ├── monitors/
│   │   └── claude-monitor.sh        # Background monitor
│   └── validators/
│
├── configs/defaults/
│   ├── permissions.yaml ⭐          # Permission rules (400 lines)
│   │                                # - deny/ask/allow sections
│   │                                # - Helpful messages
│   │                                # - WHY comments
│   └── resources.yaml ⭐            # Resource limits (150 lines)
│                                    # - CPU/memory/disk
│                                    # - Explanatory comments
│
├── tests/
│   ├── test-orchestrator.sh ⭐      # 60+ safe tests
│   ├── mock-agent.sh                # Mock AI agent
│   └── README.md                    # Test documentation
│
├── docs/
│   ├── ORCHESTRATOR_GUIDE.md ⭐     # Complete reference (600 lines)
│   ├── ARCHITECTURE.md              # System design
│   ├── QUICKSTART.md                # 10-minute guide
│   └── AI_AGENT_SECURITY_RESEARCH.md # 40+ sources
│
├── examples/
│   ├── legacy/                      # Old pre-orchestrator scripts
│   └── alternative-setups/          # Alternative approaches
│
└── justfile ⭐                       # 50+ project commands
```

## Development Workflow

### Making Changes

1. **Understand the Goal**
   - This is a security orchestrator, not a collection of separate tools
   - Everything should work through `secure-run.sh`
   - No "phases" or step-by-step installation

2. **Test First**
   ```bash
   just test              # Run all tests
   just test-quick        # Smoke test
   ```

3. **Make Changes**
   - Update relevant files
   - Maintain helpful, educational tone
   - Add comments explaining WHY

4. **Test Again**
   ```bash
   just test
   just check             # Syntax validation
   ```

5. **Update Documentation**
   - If behavior changes, update docs
   - Keep orchestrator-focused language
   - No "phase" references

### Testing Philosophy

**CRITICAL: Tests Must Be 100% Safe**

- ✅ Use pattern matching only
- ✅ Use fake paths (`/tmp/fake-path-99999`)
- ✅ Dry run testing (analyze, don't execute)
- ❌ NEVER execute `rm -rf` with real paths
- ❌ NEVER run `sudo` in tests
- ❌ NEVER modify system state

**Example Safe Test**:
```bash
# Test pattern matching without execution
OUTPUT=$(intercept-enhanced.py "rm -rf /tmp/fake-nonexistent-123" 2>&1)
assert_contains "$OUTPUT" "blocked"
```

**Test artifacts preserved on failure** - helps debugging.

### Code Style

**Shell Scripts** (`secure-run.sh`, monitors, etc.):
```bash
#!/bin/bash
# ========================================
# SECTION NAME
# ========================================
#
# PURPOSE: What this section does
# WHY: Why we do it this way
#

set -euo pipefail  # Strict mode

# Use meaningful names
ENABLE_FEATURE=true

# Always quote variables
echo "$VARIABLE"

# Comment WHY, not WHAT
# Check if Docker available (needed for isolation)
if command -v docker &> /dev/null; then
    USE_DOCKER=true
fi
```

**Python Scripts** (`intercept-enhanced.py`, validators):
```python
#!/usr/bin/env python3
"""
Brief description

Detailed explanation of what this does and why.
"""

def function_name(param: str) -> bool:
    """
    What this function does.

    Args:
        param: What this parameter means

    Returns:
        What gets returned
    """
    pass
```

**YAML Configs** (`permissions.yaml`, `resources.yaml`):
```yaml
# ============================================
# SECTION NAME
# ============================================
#
# PURPOSE: What this section controls
# WHY BLOCKED: Explanation for users
#

- pattern: "rm:-rf"
  reason: "Prevents data loss"           # For learning
  message: "❌ Deletion blocked"         # What agent sees
  suggestion: |                          # How to do safely
    Instead:
    - Create TODO
    - Ask user
  alternatives:                          # Quick options
    - "Mark for deletion"
```

### Important Patterns

**1. Configuration Hierarchy**

Search order (most restrictive wins):
```
1. .settings/       # Project-specific
2. .claude/         # Claude Code config
3. .codex/          # Codex project config
4. .opencode/       # OpenCode config
5. ~/.llmsec/       # User defaults
6. configs/defaults/# Bundled defaults
```

`.settings/` remains the preferred tool-agnostic location for shared project rules. `.codex/` now participates in shared config discovery and also holds Codex-native hook/config files for hardening.

Code pattern:
```python
SETTINGS_DIRS = [
    os.getcwd() + "/.settings",
    os.getcwd() + "/.claude",
    os.getcwd() + "/.codex",
    os.getcwd() + "/.opencode",
    os.path.expanduser("~/.llmsec/defaults"),
    os.path.dirname(__file__) + "/configs/defaults",
]

for dir in SETTINGS_DIRS:
    config = find_config(dir, "permissions.yaml")
    if config:
        return config
```

**2. Helpful Blocking Messages**

ALWAYS include:
- **reason**: Why it's blocked (educational)
- **message**: What agent sees (polite)
- **suggestion**: How to do it safely (multi-line)
- **alternatives**: Quick safe options (list)

Example:
```python
def print_block_message(rule: Dict, command: str):
    print(f"{rule['message']}")
    print(f"\nReason: {rule['reason']}")
    print(f"\n💡 Suggested Alternative:")
    print(f"   {rule['suggestion']}")
    if 'alternatives' in rule:
        print(f"\n✓ Safe Alternatives:")
        for alt in rule['alternatives']:
            print(f"   • {alt}")
```

**3. Technology Groups Integration**

All 8 groups work together:
1. **Wrappers** - Set ulimit, env before launch
2. **Containers** - Auto-detect Docker/bubblewrap
3. **Interceptors** - Analyze commands, show help
4. **Monitors** - Background process watching
5. **Hooks** - Pre-commit validation
6. **Static Analysis** - Semgrep rules
7. **Configuration** - Hierarchical YAML
8. **Logging** - Centralized audit

Pattern in orchestrator:
```bash
# Layer 1: Input Filter
setup_layer1_input_filter() {
    # Find and apply config
}

# Layer 2: Interceptor
setup_layer2_interceptor() {
    export SECURE_SHELL="$INTERCEPTOR"
}

# ... all layers ...

# Launch with everything active
launch_application
```

## What to Avoid

### ❌ DON'T: Use "Phase" Language

**Bad**:
```
"Phase 1: Quick Wins"
"Install Phase 2 after Phase 1"
"Implementation roadmap with phases"
```

**Good**:
```
"One orchestrator does everything"
"All layers enabled by default"
"Security presets: basic/recommended/maximum"
```

### ❌ DON'T: Create Separate Installation Scripts

**Bad**: `install-layer1.sh`, `install-layer2.sh`, etc.

**Good**: Everything through `secure-run.sh` with flags:
```bash
./secure-run.sh --level=basic
./secure-run.sh --no-docker
```

### ❌ DON'T: Execute Dangerous Commands in Tests

**Bad**:
```bash
rm -rf /tmp/test  # Could fail and delete /tmp
sudo echo test    # Prompts for password
```

**Good**:
```bash
# Test pattern matching only
intercept.py "rm -rf /tmp/fake-path-99999"  # Fake path
intercept.py "sudo echo test"                # Intercepted before sudo runs
```

### ❌ DON'T: Be Hostile in Messages

**Bad**:
```
"FORBIDDEN: This operation is not allowed"
"DENIED: Access prohibited"
```

**Good**:
```
"❌ Recursive deletion blocked for safety

Reason: Can cause permanent data loss

💡 Suggested Alternative:
   Instead, please:
   1. Create TODO comment
   2. Ask user for confirmation
```

## Common Tasks

### Adding a New Security Rule

1. **Edit config**:
```yaml
# In configs/defaults/permissions.yaml
deny:
  - pattern: "dangerous-command:*"
    reason: "Why it's dangerous"
    message: "❌ User-friendly message"
    suggestion: |
      How to do it safely:
      - Option 1
      - Option 2
    alternatives:
      - "Quick alternative 1"
      - "Quick alternative 2"
```

2. **Test it**:
```bash
just test-intercept "dangerous-command arg"
# Should show helpful message
```

3. **Add test**:
```bash
# In tests/test-orchestrator.sh
print_test "New rule blocks dangerous-command"
OUTPUT=$(intercept-enhanced.py "dangerous-command" 2>&1 || true)
assert_contains "$OUTPUT" "blocked"
```

4. **Run tests**:
```bash
just test
```

### Adding Orchestrator Feature

1. **Update secure-run.sh**:
```bash
# Add CLI argument
--new-feature)
    ENABLE_NEW_FEATURE=true
    shift
    ;;

# Add setup function
setup_new_feature() {
    if [ "$ENABLE_NEW_FEATURE" = "true" ]; then
        # Do setup
    fi
}

# Call in main
setup_new_feature
```

2. **Update help**:
```bash
show_usage() {
    cat << EOF
    --new-feature      Enable new feature
EOF
}
```

3. **Test**:
```bash
./secure-run.sh --new-feature --help
just test
```

4. **Document**:
```markdown
# In docs/ORCHESTRATOR_GUIDE.md
### New Feature

Description...

Usage:
\`\`\`bash
./secure-run.sh --new-feature
\`\`\`
```

### Adding a Test

1. **Create test function**:
```bash
# In tests/test-orchestrator.sh
test_new_feature() {
    print_header "Testing New Feature"

    print_test "Feature does X"
    # Test code
    assert_success $? "Should succeed"

    print_test "Feature blocks Y"
    OUTPUT=$(command 2>&1 || true)
    assert_contains "$OUTPUT" "expected"
}
```

2. **Add to main**:
```bash
main() {
    # ... existing tests ...
    test_new_feature
    # ...
}
```

3. **Run**:
```bash
just test
```

## Using Justfile

The justfile contains 50+ common commands:

```bash
# Development
just check           # Validate syntax
just test            # Run all tests
just run             # Run orchestrator

# Monitoring
just monitor         # Start monitor
just logs            # View logs

# Configuration
just config-init     # Create .settings/
just config-check    # Validate YAML

# Utilities
just stats           # Project statistics
just clean           # Clean artifacts

# Installation
just install         # Install to ~/bin
```

**See all commands**: `just` or `just --list`

## Git Workflow

```bash
# 1. Make changes
vim secure-run.sh

# 2. Test
just test

# 3. Commit
git add .
git commit -m "feat: add new feature"

# 4. For releases
just release 0.3.0
```

## Documentation Standards

- **README.md**: Quick overview, key features, basic usage
- **ORCHESTRATOR_GUIDE.md**: Complete reference, all options
- **EXAMPLE_USAGE.md**: Real-world scenarios
- **ARCHITECTURE.md**: System design, how it works
- **This file (CLAUDE.md)**: How to work with the project

**Keep docs**:
- Orchestrator-focused (not phase-based)
- Helpful and educational (not hostile)
- Example-heavy
- Up-to-date with code

## Key Files Reference

| File | Lines | Purpose |
|------|-------|---------|
| `secure-run.sh` | 520 | Main orchestrator |
| `intercept-enhanced.py` | 300 | Helpful interceptor |
| `permissions.yaml` | 400 | Security rules |
| `resources.yaml` | 150 | Resource limits |
| `test-orchestrator.sh` | 400 | Test suite |
| `ORCHESTRATOR_GUIDE.md` | 600 | Complete guide |
| `justfile` | 400 | Project commands |

## Questions to Ask

When working on this project, ask:

1. **Is this safe to test?**
   - Does it use fake paths?
   - Is it dry-run only?

2. **Is this helpful?**
   - Does the error message educate?
   - Do we provide alternatives?

3. **Is this orchestrator-focused?**
   - Can it be done through `secure-run.sh`?
   - Or does it need to be separate?

4. **Is it documented?**
   - Are there comments explaining WHY?
   - Is user documentation updated?

5. **Is it tested?**
   - Are there automated tests?
   - Do tests preserve artifacts on failure?

## Summary

**This project is**:
- ✅ Production-ready security orchestrator
- ✅ One command (`secure-run.sh`) does everything
- ✅ Helpful and educational (not hostile)
- ✅ Well-tested (60+ safe tests)
- ✅ Fully documented (13,000+ words)

**When contributing**:
- Focus on the orchestrator approach
- Be helpful and educational
- Test safely (no dangerous execution)
- Document thoroughly
- Use `just` for common tasks

**Quick start for contributors**:
```bash
just test              # Verify everything works
just run -- echo hi    # Try it out
just config-init       # Create project config
vim .settings/permissions.yaml  # Customize
just test              # Test changes
```

---

**Need help?** See `just --list` for all available commands.
