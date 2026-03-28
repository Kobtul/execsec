# LLM Security Toolkit

**One command to secure them all.**

A comprehensive, production-ready security orchestrator for AI agents (Claude Code, OpenCode, or any agentic system) with defense-in-depth architecture and helpful, educational feedback.

## 🎯 Quick Start

```bash
# Clone the repository
git clone <your-repo-url> llmsec
cd llmsec

# Run with all security enabled (default)
./secure-run.sh

# That's it! Your AI agent launches with full protection.
```

## 🛡️ What You Get

**5-Layer Defense-in-Depth Security:**

```
┌────────────────────────────────────────────────────────────────┐
│ Layer 5: MONITORING & KILL SWITCH                             │
│ Real-time process monitoring, emergency stop capability        │
├────────────────────────────────────────────────────────────────┤
│ Layer 4: OUTPUT VALIDATION                                     │
│ Scan generated code for vulnerabilities before commits         │
├────────────────────────────────────────────────────────────────┤
│ Layer 3: EXECUTION ISOLATION                                   │
│ Docker/bubblewrap sandbox, resource limits, network isolation  │
├────────────────────────────────────────────────────────────────┤
│ Layer 2: COMMAND INTERCEPTION                                  │
│ Analyze & block dangerous commands with helpful feedback       │
├────────────────────────────────────────────────────────────────┤
│ Layer 1: INPUT FILTERING                                       │
│ Permission-based blocklists, prompt injection protection       │
└────────────────────────────────────────────────────────────────┘
```

**All layers enabled by default** - turn off what you don't need.

## ✨ Key Features

### 1. Helpful, Not Hostile

When commands are blocked, agents receive **polite, educational feedback**:

```
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
```

### 2. Zero Configuration Required

Works perfectly out-of-the-box with sensible defaults. Customize only if needed.

### 3. Hierarchical Configuration

Project-specific rules override user defaults:

```
1. .settings/       ← Project-specific (highest priority)
2. .claude/         ← Claude Code config
3. .opencode/       ← OpenCode config
4. ~/.llmsec/       ← Your personal defaults
5. configs/defaults/← Bundled defaults (fallback)
```

Most restrictive setting wins.

### 4. Universal Compatibility

Works with any AI agent:
- ✅ Claude Code
- ✅ OpenCode
- ✅ Custom Python/Node agents
- ✅ Any command-line tool

### 5. Performance Conscious

Choose your security level:
- **Basic**: Essential protection, minimal overhead
- **Recommended**: Balanced security & performance (default)
- **Maximum**: Full protection, all layers active

## 📋 Usage

### Basic Usage

```bash
# Run with all defaults
./secure-run.sh

# Run different agent
./secure-run.sh --app=opencode

# Secure custom command
./secure-run.sh -- python my-agent.py
```

### Security Levels

```bash
# Basic: Layers 1 + 5 (fast, essential protection)
./secure-run.sh --level=basic

# Recommended: Layers 1 + 2 + 3 + 5 (balanced)
./secure-run.sh --level=recommended

# Maximum: All 5 layers (full protection)
./secure-run.sh --level=maximum
```

### Layer Control

```bash
# Disable specific layers
./secure-run.sh --no-docker      # Skip container isolation
./secure-run.sh --no-monitoring  # Skip process monitoring
./secure-run.sh --no-validation  # Skip code scanning
```

### Isolation Methods

```bash
# Auto-detect best method (default)
./secure-run.sh

# Force specific isolation
./secure-run.sh --isolation=docker
./secure-run.sh --isolation=bubblewrap
./secure-run.sh --isolation=none
```

## 🔧 Customization

### Project-Specific Rules

Create `.settings/permissions.yaml` in your project:

```yaml
version: "1.0"

# Block dangerous operations
deny:
  - pattern: "kubectl:apply:production"
    reason: "Production deployments require manual review"
    message: "❌ Production deployment blocked"
    suggestion: |
      To deploy to production:
      1. Create PR with deployment manifest
      2. Get team review
      3. Use CI/CD: gh workflow run deploy-prod

# Require confirmation
ask:
  - pattern: "terraform:apply"
    message: "⚠️  Infrastructure changes require confirmation"
    prompt: "This will modify cloud resources. Proceed?"
```

Run secure-run.sh and your rules are automatically applied.

### Personal Defaults

Create `~/.llmsec/defaults/permissions.yaml` for your preferences:

```yaml
deny:
  - pattern: "rm:*"
    message: "I never want deletions - ask user instead"
```

Applies to all projects by default.

## 📖 Documentation

- **[Quick Start](docs/QUICKSTART.md)** - Get running in 10 minutes
- **[Orchestrator Guide](docs/ORCHESTRATOR_GUIDE.md)** - Complete reference
- **[Example Usage](EXAMPLE_USAGE.md)** - 10 real-world scenarios
- **[Architecture](docs/ARCHITECTURE.md)** - System design details
- **[Security Research](docs/AI_AGENT_SECURITY_RESEARCH.md)** - 40+ sources

## 🧪 Testing

```bash
# Run comprehensive test suite
./tests/test-orchestrator.sh

# Test with mock agent
./tests/mock-agent.sh
```

Tests are safe - artifacts preserved on failure for debugging.

## 🎓 What's Protected

### System Harm Prevention

✅ **Blocked Operations:**
- `rm -rf /` (recursive deletion)
- `sudo` commands (privilege escalation)
- `shutdown` / `reboot` (system control)
- `chmod 777` (insecure permissions)
- `dd` / `mkfs` (disk operations)
- Fork bombs and resource exhaustion

✅ **Helpful Messages:** Agent learns safe alternatives instead of just hitting walls.

### Data Theft Prevention (Optional)

✅ **Credential Protection:**
- Blocks access to `~/.ssh/`, `~/.aws/`, `~/.kube/`
- Blocks reading `.env` files
- Blocks private key access
- Network egress control
- PII detection

Enable with:
```bash
export ENABLE_DATA_THEFT_PREVENTION=true
```

## 🔒 Repo Hardening

Harden any repository with security hooks for your AI coding tool. One command installs native hooks, git hooks, and audit logging.

### Supported Tools

| Tool | Hook Type | Auto-Detected |
|------|-----------|---------------|
| **Claude Code** | `PreToolUse` bash hook (exit 2 = block) | `.claude/` directory |
| **Codex** | `PreToolUse` hook via `.codex/hooks.json` | `.codex/` directory |
| **OpenCode** | TypeScript plugin (`tool.execute.before`) | `opencode.json` |
| **Cursor** | `beforeShellExecution` hook (JSON stdin/stdout) | `.cursor/` directory |
| **Cline** | `PreToolUse` hook (JSON stdin/stdout) | `.clinerules/` |
| **Windsurf** | VS Code deny list settings | `.windsurfrules` |
| **Any tool** | Git hooks (pre-commit, pre-push) | `.git/` |
| **Any tool** | Shell wrapper (`SHELL=guard.sh`) | Manual |

### Quick Start

```bash
# Auto-detect your AI tool and harden
./tools/harden/harden.sh /path/to/your/repo

# Or use just commands
just harden /path/to/your/repo

# Preview what would be installed
just harden-dry /path/to/your/repo

# Install everything for all tools
just harden-all /path/to/your/repo
```

### What Gets Installed

**Tool-native hooks** (block dangerous commands before execution):
- `rm -rf`, `sudo`, `chmod 777`, `dd`, `git push --force`
- Data exfiltration: `curl *pastebin*`, `cat .env`, `base64 credentials`
- Helpful blocking messages with safe alternatives

**Git hooks** (universal, work with all tools):
- `pre-commit`: Scans staged Python files for `os.system()`, `eval()`, `exec()`, `shell=True`
- `pre-push`: Blocks force pushes to protected branches (main/master)

**Audit logging**:
- All commands logged to `~/.llmsec/logs/<project>_audit.log`
- Both ALLOWED and BLOCKED commands recorded with timestamps

### Interactive Wizard

For Claude Code users, use the interactive wizard:

```
/harden-wizard
```

This walks you through: environment detection, tool selection, security level (basic/recommended/maximum), preview, and verification.

### Examples

```bash
# Harden for Claude Code specifically
./tools/harden/harden.sh ~/my-project --tool claude-code

# Harden for Cursor
./tools/harden/harden.sh ~/my-project --tool cursor

# Harden for Codex
./tools/harden/harden.sh ~/my-project --tool codex

# Git hooks only (works with any tool)
./tools/harden/harden.sh ~/my-project --no-hooks

# All tools + shell wrapper for maximum coverage
./tools/harden/harden.sh ~/my-project --tool all --with-wrapper

# Custom project name for audit logs
./tools/harden/harden.sh ~/my-project --project my-app
```

## 🔒 Security Guarantees

**What We Prevent (with all layers enabled):**
- ✅ Accidental destructive commands
- ✅ Common data exfiltration techniques
- ✅ Resource exhaustion attacks
- ✅ Basic privilege escalation
- ✅ Known malware patterns

**Best Effort (defense in depth):**
- ⚠️ Sophisticated prompt injection
- ⚠️ Advanced obfuscation techniques
- ⚠️ Zero-day exploits

**Out of Scope:**
- ❌ Physical access attacks
- ❌ Supply chain compromises
- ❌ Vulnerabilities in the LLM itself

## 📊 Project Stats

- **Lines of Code:** ~3,000
- **Documentation:** ~12,000 words
- **Research Sources:** 40+
- **Test Coverage:** Comprehensive
- **Configuration Comments:** Extensive
- **License:** MIT

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

High-impact areas:
- Additional security rules and patterns
- Integration examples (CI/CD, IDE, etc.)
- Cross-platform testing
- Documentation improvements

## 🔐 Security

Found a vulnerability? See [SECURITY.md](SECURITY.md) for disclosure policy.

## 📜 License

MIT License - See [LICENSE](LICENSE) for details.

## 🌟 Acknowledgments

Based on research from:
- Anthropic Engineering Team
- NVIDIA Security Research
- OWASP AI Security Project
- Google ADK Team
- Open source security community

## 🚀 Status

**Version:** 0.2.0
**Status:** Production Ready
**Platform:** Linux (primary), macOS (partial)

---

**Ready to secure your AI agents?**

```bash
./secure-run.sh
```

**That's it!** 🎉
