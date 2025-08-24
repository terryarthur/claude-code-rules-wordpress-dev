# Setting Up Claude Code for WordPress Development

This guide explains how to configure Claude Code to use your WordPress development rules and security instructions for consistent, professional plugin and theme development.

## Quick Setup

### 1. Create Configuration Directory
In your WordPress project root, create the Claude Code configuration directory:

```bash
mkdir .claude
```

### 2. Add Configuration Files
Place your WordPress development files in the `.claude` directory:

```
your-wordpress-project/
├── .claude/
│   ├── rules.md              # General development rules
│   ├── instructions.md       # Technical WordPress patterns
│   └── config.md            # Project-specific settings
├── wp-content/
├── wp-config.php
└── ...
```

### 3. Update .gitignore
Add these lines to your project's `.gitignore`:

```
# Claude Code files
.claude/
*.md
!README.md

# WordPress
wp-config.php
wp-content/uploads/
.htaccess

# Development
node_modules/
.env
.DS_Store
```

## Configuration Files

### rules.md
Contains your general development rules:
- Git management and branching strategy
- Package folder requirements
- Professional coding standards
- Zero technical debt policy
- PHPCS/WPCS validation requirements

### instructions.md  
Contains detailed WordPress-specific technical patterns:
- WordPress Coding Standards implementation
- Security patterns (nonces, sanitization, escaping)
- Database security with prepared statements
- User capability verification
- File upload security
- Performance optimization
- Accessibility requirements

### config.md (Optional)
Project-specific configuration that references the other files and adds:
- Current project type (plugin/theme)
- WordPress version requirements
- Project prefix and text domain
- Security checklist
- Development workflow

## Using Claude Code with Your Configuration

### Automatic Discovery
Claude Code automatically reads configuration files from:
- `.claude/` directory in your project root
- Any `.md` files in the `.claude` directory
- Common names like `rules.md`, `instructions.md`

### Starting Claude Code Sessions

```bash
# Start with automatic config discovery
claude-code

# Or explicitly reference config files
claude-code --config .claude/rules.md,.claude/instructions.md

# Or use combined config
claude-code --config .claude/config.md
```

### Example Commands

Reference your configuration files in your requests:

```bash
# Create new functionality
claude-code "Create a new WordPress plugin following the rules in rules.md and security patterns in instructions.md"

# Add features
claude-code "Add a settings page to this plugin using the WordPress admin patterns from instructions.md"

# Code review
claude-code "Review this code for security issues according to our WordPress security requirements"

# Specific implementations
claude-code "Add AJAX functionality using the nonce patterns from instructions.md"
```

## Project Template Setup

Create a script to quickly set up new WordPress projects:

```bash
#!/bin/bash
# setup-wp-project.sh

# Create Claude Code directory
mkdir -p .claude

# Copy your standard configuration files
cp /path/to/your/templates/rules.md .claude/
cp /path/to/your/templates/instructions.md .claude/
cp /path/to/your/templates/config.md .claude/

# Initialize git if not already done
git init

# Create gitignore
cat > .gitignore << EOF
# Claude Code files
.claude/
*.md
!README.md

# WordPress
wp-config.php
wp-content/uploads/
.htaccess

# Development
node_modules/
.env
.DS_Store
EOF

echo "WordPress project configured for Claude Code"
```

Make it executable and use it:

```bash
chmod +x setup-wp-project.sh
./setup-wp-project.sh
```

## Complete Project Structure

Your WordPress project should look like this:

```
my-wordpress-plugin/
├── .claude/
│   ├── rules.md              # Development rules
│   ├── instructions.md       # WordPress technical patterns
│   └── config.md            # Project configuration
├── .gitignore               # Excludes .claude and *.md files
├── my-plugin.php            # Main plugin file
├── includes/
│   ├── class-my-plugin.php
│   ├── admin/
│   │   ├── class-admin.php
│   │   └── partials/
│   └── public/
│       ├── class-public.php
│       └── partials/
├── assets/
│   ├── css/
│   ├── js/
│   └── images/
├── languages/
├── package/                 # Clean files for distribution
└── README.md               # Only README.md is not gitignored
```

## Testing Your Setup

### 1. Verify Configuration Loading
```bash
# Test that Claude Code reads your rules
claude-code "What are the key security requirements from our project rules?"

# Test technical pattern recognition
claude-code "Show me the correct way to handle form processing according to our instructions"
```

### 2. Create Test Implementation
```bash
# Initialize test project
mkdir test-wp-plugin
cd test-wp-plugin

# Copy your .claude configuration files
cp -r /path/to/your/template/.claude .

# Test plugin creation
claude-code "Create a simple WordPress plugin with a settings page"
```

### 3. Verify Compliance
```bash
# Check code against your standards
claude-code "Review the code you just created against our security requirements"

# Verify specific patterns
claude-code "Show me how this code implements nonce verification according to our standards"
```

## Configuration File Templates

### Basic config.md Template
```markdown
# WordPress Project Configuration for Claude Code

## Project Overview
This is a WordPress [plugin/theme] development project following strict security and coding standards.

## Development Rules
Please read and follow ALL rules in `rules.md` - these are mandatory.

## Technical Instructions  
Please implement ALL patterns in `instructions.md` - these provide required security implementations.

## Project Settings
- **Project Type**: [Plugin/Theme]
- **WordPress Version**: Minimum 6.0, Tested up to 6.4
- **PHP Version**: Minimum 7.4, Tested up to 8.2
- **Prefix**: `your_prefix_`
- **Text Domain**: `your-text-domain`

## Security Checklist
Before ANY commit:
- [ ] All inputs sanitized
- [ ] All outputs escaped  
- [ ] Nonces implemented
- [ ] Capabilities checked
- [ ] Prepared statements used
- [ ] PHPCS validation passes
```

## Best Practices

### Development Workflow
1. **Start New Feature**: Create feature branch
2. **Reference Config**: Use Claude Code with explicit config references
3. **Follow Standards**: Implement per rules.md and instructions.md
4. **Test Thoroughly**: Multiple user roles and scenarios
5. **Validate Code**: Run PHPCS/WPCS validation
6. **Fix Issues**: Zero tolerance for errors/warnings
7. **Commit**: Push feature branch when complete

### Maintenance
- **Update Config**: Keep rules and instructions current with WordPress updates
- **Review Regularly**: Ensure Claude Code is following all requirements
- **Test Changes**: Verify configuration updates work as expected
- **Document Updates**: Note any changes to development patterns

### Troubleshooting

#### Claude Code Not Following Rules
- Verify `.claude` directory exists in project root
- Check file names match: `rules.md`, `instructions.md`
- Explicitly reference files in commands
- Test with simple configuration verification commands

#### Configuration Not Loading
- Check file permissions on `.claude` directory
- Verify files are valid markdown
- Test with `claude-code --config .claude/rules.md`
- Review Claude Code documentation for latest configuration options

## Additional Resources

- **PHPCS Installation**: `composer global require "squizlabs/php_codesniffer=*"`
- **WordPress Standards**: `composer global require wp-coding-standards/wpcs`
- **Claude Code Documentation**: Check official Anthropic documentation
- **WordPress Coding Standards**: https://developer.wordpress.org/coding-standards/

---

**Remember**: These configuration files ensure consistent, secure, professional WordPress development. Every rule and pattern is mandatory for maintaining code quality and security standards.
