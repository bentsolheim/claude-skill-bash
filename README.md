# Claude Skill: Bash Best Practices

A Claude Code skill that ensures bash scripts follow enterprise-grade best practices for maintainability, reliability, and user-friendliness.

## Overview

This skill automatically activates when working with bash scripts to enforce consistent patterns including:
- Main function pattern with guard clause
- Comprehensive usage documentation
- Structured argument parsing
- Dependency validation
- Explicit error handling (no `set -e`)
- Organized function structure
- User-friendly colored output

## Features

- **Automatic Activation**: Triggers when creating or editing bash scripts
- **Comprehensive Standards**: 1000+ lines of battle-tested best practices
- **Script Generation**: Includes scaffolding tool for new scripts
- **Template System**: Reusable templates for common patterns
- **Version Controlled**: Deploy consistent standards across projects

## Installation

### Option 1: Direct Installation

Copy the skill to your Claude project:

```bash
# Clone this repository
git clone https://github.com/yourusername/claude-skill-bash.git
cd claude-skill-bash

# Copy entire skill directory to global Claude skills
cp -r . ~/.claude/skills/claude-skill-bash/

# Or for a specific project
cp -r . /path/to/project/.claude/skills/claude-skill-bash/
```

### Option 2: Symlink (for development)

Link the skill for easy updates:

```bash
# Global installation
ln -s $(pwd) ~/.claude/skills/claude-skill-bash

# Project-specific
ln -s $(pwd) /path/to/project/.claude/skills/claude-skill-bash
```

### Option 3: Automated Deployment Script

```bash
#!/usr/bin/env bash
# deploy-skill.sh - Deploy claude-skill-bash

SKILL_NAME="claude-skill-bash"

function deploy_global() {
    local target="$HOME/.claude/skills/$SKILL_NAME"
    mkdir -p "$target"
    cp -r SKILL.md templates/ scripts/ "$target/"
    echo "✅ Deployed globally to: $target"
}

function deploy_project() {
    local project_path="$1"
    local target="$project_path/.claude/skills/$SKILL_NAME"
    mkdir -p "$target"
    cp -r SKILL.md templates/ scripts/ "$target/"
    echo "✅ Deployed to project: $target"
}

# Usage
case "${1:-}" in
    --global)
        deploy_global
        ;;
    --project)
        deploy_project "${2:-.}"
        ;;
    *)
        echo "Usage: $0 --global | --project <path>"
        exit 1
        ;;
esac
```

## Usage

### Automatic Invocation

The skill automatically activates when Claude detects:
- Creating new bash/shell scripts
- Editing `.sh` or `.bash` files
- Requests for automation, deployment, or backup scripts
- Mentions of bash, shell scripting, or script structure

### Example Prompts

```
"Create a backup script for my MySQL databases"
"Review scripts/deploy.sh for best practices"
"I need a script to process log files"
"Make this bash script more maintainable"
```

### Using the Scaffold Tool

Generate new scripts with the included scaffold utility:

```bash
# Basic usage
scripts/scaffold.sh -n backup.sh -d "Backup databases"

# With dependencies
scripts/scaffold.sh \
    --name deploy.sh \
    --description "Deploy application to production" \
    --dependencies "docker,kubectl,jq"

# Minimal template
scripts/scaffold.sh \
    -n process.sh \
    -d "Process data files" \
    --minimal
```

## Project Structure

```
claude-skill-bash/
├── SKILL.md                    # Main skill definition with all best practices
├── templates/
│   └── script-template.sh      # Reusable bash script template
├── scripts/
│   └── scaffold.sh             # Script generator utility
└── README.md                   # Documentation
```

## Skill Components

### SKILL.md

The main skill file containing:
- Frontmatter with name and auto-trigger description
- Comprehensive bash best practices (1000+ lines)
- Template structure with examples
- Common patterns and anti-patterns
- Testing guidelines

### Templates

Reusable script templates with placeholders:
- `{{SCRIPT_NAME}}` - Script filename
- `{{DESCRIPTION}}` - Script purpose
- `{{AUTHOR}}` - Author name
- `{{DATE}}` - Creation date

### Utilities

- `scaffold.sh` - Generates new scripts following all best practices

## Version Management

### Semantic Versioning

This skill follows semantic versioning:
- **Major**: Breaking changes to skill interface
- **Minor**: New features or patterns added
- **Patch**: Bug fixes and minor improvements

### Updating

To update the skill in your projects:

```bash
# Pull latest version
git pull origin main

# Redeploy to projects
./deploy-skill.sh --global
```

## Testing

### Test the Skill

1. **Create a test script**:
   ```
   "Create a script to backup and compress log files"
   ```

2. **Verify patterns**:
   - Main function with guard clause ✓
   - Usage function ✓
   - Argument parsing ✓
   - Dependency checking ✓
   - Error handling ✓

3. **Edit existing script**:
   ```
   "Review and improve this bash script"
   ```

### Validate Generated Scripts

```bash
# Syntax check
bash -n generated-script.sh

# Test with shellcheck (if installed)
shellcheck generated-script.sh

# Run help
./generated-script.sh --help
```

## Customization

### Modify Standards

Edit `SKILL.md` to adjust practices for your organization:

1. Update the template structure
2. Modify color schemes
3. Add organization-specific patterns
4. Include custom utility functions

### Extend Templates

Add new templates for specific use cases:

```bash
# Create specialized template
cp templates/script-template.sh templates/backup-template.sh
# Edit for backup-specific structure
```

## Integration

### With CI/CD

Add validation to your pipeline:

```yaml
# .github/workflows/bash-lint.yml
name: Bash Script Validation
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Validate bash scripts
        run: |
          for script in scripts/*.sh; do
            bash -n "$script"
          done
```

### With Git Hooks

Auto-check scripts before commit:

```bash
#!/usr/bin/env bash
# .git/hooks/pre-commit
for file in $(git diff --cached --name-only | grep '\.sh$'); do
    if ! bash -n "$file"; then
        echo "Syntax error in $file"
        exit 1
    fi
done
```

## Troubleshooting

### Skill Not Activating

1. Check skill is in correct directory:
   ```bash
   ls ~/.claude/skills/bash-best-practices/
   ```

2. Verify SKILL.md frontmatter is valid

3. Try explicit mention: "Apply bash best practices to this script"

### Generated Scripts Have Issues

1. Ensure template file exists and is readable
2. Check scaffold.sh has execute permissions
3. Verify dependencies are installed

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your improvements
4. Test thoroughly
5. Submit a pull request

### Areas for Contribution

- Additional script templates
- New utility functions
- Platform-specific patterns
- Integration examples
- Documentation improvements

## License

MIT License - See LICENSE file for details

## Support

- Issues: [GitHub Issues](https://github.com/yourusername/claude-skill-bash/issues)
- Discussions: [GitHub Discussions](https://github.com/yourusername/claude-skill-bash/discussions)

## Changelog

### Version 1.0.0 (2024-11-03)
- Initial release
- Core bash best practices
- Script scaffolding tool
- Template system

## Acknowledgments

- Based on enterprise bash scripting patterns
- Inspired by Google Shell Style Guide
- Community best practices from DevOps teams