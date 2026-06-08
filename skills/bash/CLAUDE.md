# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a Claude Skill package that enforces comprehensive bash scripting best practices. The skill auto-activates when working with bash scripts to ensure consistent patterns including main function pattern, structured argument parsing, dependency validation, and explicit error handling.

## Development Commands

### Testing the Scaffold Tool
```bash
# Generate a test script
scripts/scaffold.sh -n test.sh -d "Test script" --dependencies "jq,curl"

# Validate generated script syntax
bash -n test.sh

# Test with minimal template
scripts/scaffold.sh -n minimal.sh -d "Minimal script" --minimal
```

### Deployment Testing
```bash
# Test local deployment (dry run)
mkdir -p /tmp/test-skill
cp -r SKILL.md templates/ scripts/ /tmp/test-skill/
ls -la /tmp/test-skill/

# Verify skill structure
cat SKILL.md | head -4  # Check frontmatter is valid YAML
```

## Architecture & Key Design Decisions

### Skill Activation Strategy
The skill uses Claude's language understanding for context-aware activation rather than pattern matching. The description in SKILL.md frontmatter is crafted to trigger on:
- File extensions (.sh, .bash)
- Task keywords (automation, deployment, backup)
- Bash-specific terms (shell script, argument parsing, main function)

This ensures the 1000+ lines of best practices only load when relevant, keeping context clean for non-bash work.

### Template System Architecture
The scaffold.sh utility uses string substitution on templates/script-template.sh with placeholders:
- `{{SCRIPT_NAME}}`, `{{DESCRIPTION}}`, `{{AUTHOR}}`, `{{DATE}}`
- Dependencies are injected by modifying the DEPENDENCIES array
- Conditional sections (colors, utility functions) are removed via sed for --minimal and --no-colors flags

### Skill Content Structure
SKILL.md is organized to progressively disclose information:
1. **Frontmatter**: Minimal metadata for skill discovery
2. **When This Skill Applies**: Clear triggering conditions
3. **Core Principles**: High-level requirements (7 key patterns)
4. **Script Structure Template**: Complete reference implementation
5. **Detailed Best Practices**: In-depth explanations with examples

This structure allows Claude to quickly understand what's needed while having comprehensive details available.

## Critical Implementation Details

### Guard Clause Enforcement
The skill mandates the guard clause pattern at script end:
```bash
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
    exit 0
fi
```
This enables scripts to be sourced for testing/composition without execution - a key architectural principle.

### Function Ordering Rules
The skill enforces strict function ordering (not algorithmic, but through instructions):
1. usage() - First after globals
2. main() - Immediately after usage
3. Business logic functions
4. Utility functions

This is NOT enforced with comments like "# UTILITY FUNCTIONS" which the skill explicitly prohibits.

### Error Handling Philosophy
The skill explicitly prohibits `set -e` in favor of explicit error handling. This is a deliberate architectural choice for:
- Better debugging (clear error context)
- Predictable behavior across shells
- Educational value (explicit is better than implicit)

## Version Management Considerations

When updating the skill:
- SKILL.md frontmatter name must remain `claude-skill-bash`
- Description changes affect activation patterns - test carefully
- Template modifications in script-template.sh affect all generated scripts
- The scaffold.sh TEMPLATE_FILE path assumes ../templates/ structure

## Testing Strategy

The skill itself doesn't include automated tests but relies on:
1. Syntax validation: `bash -n` on generated scripts
2. Template testing: scaffold.sh with various flag combinations
3. Activation testing: Manually verify skill triggers appropriately
4. Structure validation: Ensure SKILL.md frontmatter is valid YAML

## GitHub Repository

Remote: https://github.com/bentsolheim/claude-skill-bash.git

The repository is public and intended for distribution as a reusable Claude Skill package.