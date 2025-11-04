# Claude Skill: Bash Best Practices

A Claude Code skill that ensures bash scripts follow enterprise-grade best practices for maintainability, reliability, and user-friendliness.

## Overview

This skill automatically activates when working with bash scripts to enforce consistent patterns. It recognizes two script types:

### Simple Scripts (<30 lines, no arguments)
For scripts that do one simple thing well:
- Direct execution without main function
- No argument parsing needed
- Minimal boilerplate
- Clear purpose comment

### Ordinary Scripts (larger scope)
For scripts with broader functionality:
- Main function pattern with guard clause
- Comprehensive usage documentation
- Structured argument parsing
- Dependency validation
- Organized function structure
- User-friendly colored output

Both types enforce:
- Explicit error handling (no `set -e`)
- Proper stream usage (stdout vs stderr)
- Meaningful exit codes
- Variable safety

## Features

- **Automatic Activation**: Triggers when creating or editing bash scripts
- **Comprehensive Standards**: 1000+ lines of battle-tested best practices
- **Script Generation**: Includes scaffolding tool for new scripts
- **Template System**: Reusable templates for common patterns
- **Version Controlled**: Deploy consistent standards across projects

## Installation

### Recommended: Symlink Installation

Using a symlink ensures you always have the latest version and can easily pull updates from git:

```bash
# Clone this repository
git clone https://github.com/bentsolheim/claude-skill-bash.git
cd claude-skill-bash

# Create symlink for global installation (recommended)
ln -s $(pwd) ~/.claude/skills/claude-skill-bash

# Or for a specific project
ln -s $(pwd) /path/to/project/.claude/skills/claude-skill-bash
```

To update the skill later:
```bash
cd claude-skill-bash
git pull
```

### Alternative: Direct Copy

If you prefer a static installation without symlinks:

```bash
# Clone this repository
git clone https://github.com/bentsolheim/claude-skill-bash.git
cd claude-skill-bash

# Copy entire skill directory to global Claude skills
cp -r . ~/.claude/skills/claude-skill-bash/

# Or for a specific project
cp -r . /path/to/project/.claude/skills/claude-skill-bash/
```

Note: With this method, you'll need to manually copy files again after updates.

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

## Troubleshooting

### Skill Not Activating

1. Check skill is in correct directory:
   ```bash
   ls ~/.claude/skills/claude-skill-bash/
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

- Issues: [GitHub Issues](https://github.com/bentsolheim/claude-skill-bash/issues)
- Discussions: [GitHub Discussions](https://github.com/bentsolheim/claude-skill-bash/discussions)

## Changelog

### Version 1.1.0 (2025-11-04)
- Add support for simple scripts (<30 lines, no arguments)
- New --simple flag in scaffold tool
- Separate templates for simple vs ordinary scripts
- Decision tree for choosing script complexity
- Examples of CI/CD and data transformation scripts

### Version 1.0.0 (2025-11-03)
- Initial release
- Core bash best practices
- Script scaffolding tool
- Template system

## Acknowledgments

- Based on enterprise bash scripting patterns
- Inspired by Google Shell Style Guide
- Community best practices from DevOps teams