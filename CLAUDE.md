# Awesome Claude Code - Development Guide

## Project Overview

**Awesome Claude Code** is a curated repository of resources for enhancing Claude Code workflows. This is a community-driven project that maintains a structured catalog of slash-commands, CLAUDE.md files, workflows, tooling, and other resources that help users get the most out of Claude Code.

### Key Characteristics

- **Data-Driven**: All resources are stored in `THE_RESOURCES_TABLE.csv` (single source of truth)
- **Automated**: Heavy use of Python scripts and GitHub Actions for validation, PR creation, and maintenance
- **Template-Based**: README.md is auto-generated from templates using category definitions
- **Community-Focused**: Streamlined issue-based submission process with automated validation

### Primary Stakeholders

- **Contributors**: Submit resources via GitHub issue forms
- **Maintainers**: Review submissions, approve/reject, manage categories
- **Users**: Browse and discover Claude Code resources

## Repository Structure

```
/
├── THE_RESOURCES_TABLE.csv       # Single source of truth for all resources
├── README.md                      # Auto-generated from CSV + templates
├── CONTRIBUTING.md                # Contributor guidelines
├── HOW_IT_WORKS.md               # Technical architecture documentation
├── Makefile                       # Development task automation
├── pyproject.toml                # Python project configuration
├── .pre-commit-config.yaml       # Pre-commit hooks
│
├── templates/                     # README generation templates
│   ├── README.template.md        # Main README structure
│   ├── categories.yaml           # SINGLE SOURCE OF TRUTH for categories
│   ├── resource-overrides.yaml   # Manual overrides for specific resources
│   └── announcements.yaml        # Announcements section content
│
├── scripts/                       # Automation scripts (23+ Python files)
│   ├── parse_issue_form.py       # Parse GitHub issue submissions
│   ├── validate_links.py         # Bulk link validation
│   ├── validate_single_resource.py # Single resource validation
│   ├── generate_readme.py        # Generate README from CSV
│   ├── create_resource_pr.py     # Auto-create PRs from approvals
│   ├── add_resource.py           # Interactive resource addition
│   ├── add_category.py           # Add new categories
│   ├── submit_resource.py        # One-command submission workflow
│   ├── sort_resources.py         # Sort CSV by category/name
│   └── ...                       # Additional utility scripts
│
├── .github/
│   ├── workflows/                # GitHub Actions automation
│   │   ├── validate-resource-submission.yml   # Validates new submissions
│   │   ├── approve-resource-submission.yml    # Handles approvals
│   │   ├── validate-links.yml                 # Scheduled link checking
│   │   ├── protect-labels.yml                 # Prevents unauthorized label changes
│   │   └── notify-on-merge.yml                # Post-merge notifications
│   │
│   └── ISSUE_TEMPLATE/
│       └── submit-resource.yml   # Structured submission form
│
└── resources/                     # Downloaded/hosted resources
    ├── official-documentation/
    ├── claude.md-files/
    ├── slash-commands/
    └── workflows-knowledge-guides/
```

## Core Data Structure

### THE_RESOURCES_TABLE.csv

The CSV file is the **single source of truth** for all resources. Each row represents one resource with these fields:

| Field | Description | Auto-populated? |
|-------|-------------|-----------------|
| ID | Unique identifier (`{prefix}-{hash}`) | Yes |
| Display Name | Resource name as shown in README | No |
| Category | Main category | No |
| Sub-Category | Optional subcategory | No |
| Primary Link | Main URL | No |
| Secondary Link | Additional URL (docs, npm, etc.) | No |
| Author Name | Creator name/username | No |
| Author Link | Creator profile URL | No |
| Active | TRUE/FALSE status | Yes (validation) |
| Date Added | ISO timestamp of addition | Yes |
| Last Modified | GitHub last commit date | Yes (from API) |
| Last Checked | Validation timestamp | Yes |
| License | SPDX identifier | Yes (from GitHub) |
| Description | 1-2 sentence description | No |
| Removed From Origin | TRUE if removed from source | No |

### Categories System

All categories are defined in `templates/categories.yaml`. This is the **single source of truth** for category definitions.

**Current Categories** (in order):
1. **Agent Skills** - Specialized capabilities for Claude Code
2. **Workflows & Knowledge Guides** - Comprehensive workflow systems
3. **Tooling** - CLI applications and executables
   - IDE Integrations
   - Usage Monitors
   - Orchestrators
4. **Status Lines** - Status bar configurations
5. **Hooks** - Lifecycle event handlers
6. **Output Styles** - Custom output formatting
7. **Slash-Commands** - Individual command files
   - Version Control & Git
   - Code Analysis & Testing
   - Context Loading & Priming
   - Documentation & Changelogs
   - CI / Deployment
   - Project & Task Management
   - Miscellaneous
8. **CLAUDE.md Files** - Project configuration files
   - Language-Specific
   - Domain-Specific
   - Project Scaffolding & MCP
9. **Alternative Clients** - Alternative UIs/front-ends
10. **Official Documentation** - Anthropic resources

## Development Workflows

### Initial Setup

```bash
# Create virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
make install
# Or manually:
pip install -e ".[dev]"

# Install pre-commit hooks (optional but recommended)
pre-commit install
```

### Common Tasks

```bash
# Validate all resource links
make validate

# Validate a single URL
make validate-single URL=https://example.com

# Generate README from CSV
make generate

# Sort resources in CSV
make sort

# Add a new resource interactively
make add_resource

# Add a new category
make add-category
# Or with arguments:
make add-category ARGS='--name "My Category" --prefix mycat --icon 🎯'

# Format code with ruff
make format

# Run tests
make test
```

### Resource Submission Flow

**For Contributors (via GitHub):**
1. Submit resource via issue form: [Submit Resource](https://github.com/hesreallyhim/awesome-claude-code/issues/new?template=submit-resource.yml)
2. Automated validation runs and posts results as comment
3. Maintainer reviews and uses `/approve`, `/request-changes`, or `/reject`
4. On approval, bot auto-creates PR, adds resource to CSV, regenerates README
5. Maintainer merges PR

**For Maintainers (direct):**
```bash
# Interactive submission
make add_resource

# One-command workflow (advanced)
make submit ARGS='--name "Resource Name" --url https://...'
```

### Validation System

**Automated Checks:**
- URL accessibility (200 OK status)
- Duplicate detection
- Required field validation
- License extraction (from GitHub API)
- Last modified date (from GitHub API)

**Manual Overrides:**
Use `templates/resource-overrides.yaml` to:
- Lock specific fields from updates
- Skip validation for special cases
- Manually correct auto-detected data

Example override:
```yaml
overrides:
  resource-id-abc123:
    license: "Custom-License"
    skip_validation: true
```

### README Generation

The README is **completely generated** from:
1. `THE_RESOURCES_TABLE.csv` (resource data)
2. `templates/README.template.md` (structure)
3. `templates/categories.yaml` (category definitions)
4. `templates/announcements.yaml` (announcements section)

**Never manually edit README.md** - all changes will be overwritten. Instead:
- Modify templates for structural changes
- Update CSV for resource changes
- Update categories.yaml for category changes

### Git Workflow

**Branch Strategy:**
- `main` - Production branch
- `add-resource/{category}/{name}-{timestamp}` - Auto-created for new resources
- Feature branches as needed

**Commit Guidelines:**
- Use clear, descriptive commit messages
- Reference issue numbers when applicable
- Auto-generated commits follow the pattern: "Add {resource_name} to {category}"

**Important:**
- The repository is configured to work with branch names starting with `claude/`
- When pushing, use: `git push -u origin <branch-name>`
- Only push to branches matching the session pattern

## Key Scripts and Their Purposes

### Validation Scripts
- **`validate_links.py`** - Bulk validation of all resources in CSV
- **`validate_single_resource.py`** - Validate a single URL
- **`validate_new_resource.py`** - Validate new submissions from issues

### Resource Management
- **`parse_issue_form.py`** - Extract data from GitHub issue submissions
- **`create_resource_pr.py`** - Auto-create PRs from approved submissions
- **`add_resource.py`** - Interactive CLI for adding resources
- **`submit_resource.py`** - One-command submission workflow

### Generation & Organization
- **`generate_readme.py`** - Generate README from CSV + templates
- **`sort_resources.py`** - Sort CSV by category, subcategory, name
- **`generate_resource_id.py`** - Create unique IDs for new resources

### Category Management
- **`add_category.py`** - Add new categories to the system
- **`category_utils.py`** - Category-related utility functions

### Utilities
- **`download_resources.py`** - Download resources from GitHub for local hosting
- **`badge_issue_notification.py`** - Create notification issues on featured repos
- **`git_utils.py`** - Git-related helper functions

## GitHub Actions Workflows

### validate-resource-submission.yml
**Trigger:** Issue creation/edit with `resource-submission` label
**Purpose:** Validate submission data and post results as comment
**Actions:**
- Parses issue body
- Validates URLs, fields, duplicates
- Updates issue labels (`validation-passed`/`validation-failed`)
- Posts detailed comment with results

### approve-resource-submission.yml
**Trigger:** Maintainer comments `/approve`, `/request-changes`, or `/reject`
**Purpose:** Handle maintainer decisions
**Actions:**
- `/approve` → Creates PR automatically
- `/request-changes` → Adds label, requests edits
- `/reject` → Closes issue with reason

### protect-labels.yml
**Trigger:** Label changes on issues
**Purpose:** Prevent unauthorized label manipulation
**Actions:**
- Reverts label changes by non-maintainers
- Preserves system integrity

### validate-links.yml
**Trigger:** Scheduled (periodic) or manual
**Purpose:** Check all resources for broken links
**Actions:**
- Validates all URLs in CSV
- Creates issues for broken links
- Updates `Active` status

### notify-on-merge.yml
**Trigger:** PR merge to main
**Purpose:** Notify submitters and create badge notifications
**Actions:**
- Posts comment on original issue
- Creates notification issue on featured GitHub repos (if enabled)

## Code Style and Conventions

### Python Code
- **Python Version:** 3.11+
- **Line Length:** 100 characters max
- **Formatter:** ruff (with auto-fix)
- **Linter:** ruff (pycodestyle, pyflakes, isort, pep8-naming, etc.)
- **Naming:**
  - Functions/variables: `snake_case`
  - Classes: `PascalCase`
  - Constants: `UPPER_CASE`
- **Imports:** Organized by isort (stdlib → third-party → local)
- **Type Hints:** Recommended for new code

### CSV Guidelines
- Use UTF-8 encoding
- Preserve header row exactly
- Quote fields containing commas
- Maintain field order
- Use ISO format for dates: `YYYY-MM-DD:HH-MM-SS`

### YAML Files
- 2-space indentation
- Use `-` for lists
- Quote strings containing special characters
- Maintain alphabetical order where logical

## Testing

### Running Tests
```bash
# Run all tests
make test
# Or directly:
pytest tests/ -v

# Run specific test file
pytest tests/test_validation.py -v
```

### Test Coverage
Tests should cover:
- URL validation logic
- CSV parsing and generation
- Category management
- Resource ID generation
- Template rendering

## Important Conventions for AI Assistants

### DO:
✅ Always regenerate README after CSV changes: `make generate`
✅ Sort CSV before generating README: `make sort`
✅ Validate links before committing: `make validate`
✅ Use Makefile commands for common tasks
✅ Check `templates/categories.yaml` for valid categories
✅ Reference issues by number in commits
✅ Use descriptive commit messages
✅ Respect the CSV structure exactly
✅ Read HOW_IT_WORKS.md for technical details
✅ Read CONTRIBUTING.md for contribution guidelines

### DON'T:
❌ Never manually edit README.md (it's auto-generated)
❌ Never modify CSV structure without updating all scripts
❌ Never add categories without using `add-category.py`
❌ Never commit broken links (validate first)
❌ Never modify GitHub Action workflows without testing
❌ Never change label names (breaks automation)
❌ Never skip validation for new resources
❌ Never commit unformatted code (run `make format`)

## Common Operations

### Adding a New Resource

**Method 1: GitHub Issue (Recommended for external contributors)**
1. Use the issue template
2. Wait for validation
3. Maintainer approval
4. Auto PR creation

**Method 2: Interactive CLI (For maintainers)**
```bash
make add_resource
# Follow prompts
```

**Method 3: Manual (Advanced)**
1. Add row to `THE_RESOURCES_TABLE.csv`
2. Generate unique ID using `generate_resource_id.py`
3. Run `make sort`
4. Run `make generate`
5. Run `make validate`
6. Commit changes

### Removing a Resource

1. Set `Active` field to `FALSE` in CSV
2. Run `make generate`
3. Commit with reason

Or completely remove the row from CSV (less preferred).

### Updating a Resource

1. Edit the relevant fields in CSV
2. Run `make generate`
3. Commit changes

**Note:** `Last Checked`, `Last Modified`, and `License` are auto-updated by validation scripts.

### Adding a New Category

```bash
make add-category
# Or with arguments:
make add-category ARGS='--name "Extensions" --prefix ext --icon 🧩'
```

This automatically:
- Updates `templates/categories.yaml`
- Updates `.github/ISSUE_TEMPLATE/submit-resource.yml`
- Regenerates README
- Optionally commits changes

## Troubleshooting

### "Validation failed" on submission
- Check URL accessibility (must return 200 OK)
- Ensure URL starts with `https://`
- Verify no duplicate exists
- Check all required fields are filled

### "CSV is malformed"
- Ensure UTF-8 encoding
- Check for unescaped quotes
- Verify header row is intact
- Look for missing commas

### "Category not found"
- Check `templates/categories.yaml` for valid categories
- Use exact category name (case-sensitive)
- Subcategory must match parent category

### "README generation failed"
- Validate CSV structure
- Check template files exist
- Ensure categories.yaml is valid YAML
- Run `make sort` first

## Security Considerations

### Resource Submission
- All submissions are manually reviewed
- URLs are validated for accessibility
- Security risks are evaluated by maintainers
- Advanced tools require additional review time

### Script Execution
- Scripts run in CI/CD with minimal permissions
- GitHub tokens are scoped appropriately
- No external data modification without approval

### Best Practices
- Never commit sensitive data (API keys, tokens)
- Use environment variables for secrets
- Review all PR changes carefully
- Report security issues immediately

## Environment Variables

**For Local Development:**
```bash
export GITHUB_TOKEN=ghp_...  # Optional, avoids rate limiting
```

**For GitHub Actions (automatically set):**
- `GITHUB_TOKEN` - For API access
- `AWESOME_CC_PAT_PUBLIC_REPO` - For creating notification issues
- `CREATE_ISSUES` - Enable/disable badge notifications

## Related Documentation

- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Detailed contribution guidelines
- **[HOW_IT_WORKS.md](HOW_IT_WORKS.md)** - Technical architecture deep-dive
- **[code-of-conduct.md](code-of-conduct.md)** - Community standards
- **[scripts/README.md](scripts/README.md)** - Script-specific documentation

## Quick Reference

### File Modification Matrix

| Task | Files to Modify | Commands to Run |
|------|----------------|-----------------|
| Add resource | `THE_RESOURCES_TABLE.csv` | `make sort && make generate` |
| Update resource | `THE_RESOURCES_TABLE.csv` | `make generate` |
| Add category | `templates/categories.yaml` | `make add-category` |
| Change README structure | `templates/README.template.md` | `make generate` |
| Add announcement | `templates/announcements.yaml` | `make generate` |
| Override validation | `templates/resource-overrides.yaml` | `make validate` |
| Fix formatting | Any `.py` file | `make format` |

### Label Reference

**Submission Process:**
- `resource-submission` - Auto-applied to submissions
- `validation-passed` - Submission passed all checks
- `validation-failed` - Submission has issues
- `approved` - Maintainer approved
- `rejected` - Maintainer rejected
- `changes-requested` - Edits needed
- `pr-created` - PR auto-created
- `error-creating-pr` - PR creation failed

**Maintenance:**
- `broken-links` - Resource links are broken
- `automated` - Auto-detected issue

## For Maintainers

### Approval Commands
Comment on resource submission issues:
- `/approve` - Accept and auto-create PR
- `/request-changes [reason]` - Request modifications
- `/reject [reason]` - Decline submission

### Maintenance Tasks
- Review submissions regularly
- Monitor scheduled link validation
- Update categories as needed
- Merge approved PRs
- Handle edge cases with overrides

## Getting Help

- **Technical Issues:** Check [HOW_IT_WORKS.md](HOW_IT_WORKS.md)
- **Contribution Questions:** See [CONTRIBUTING.md](CONTRIBUTING.md)
- **Bug Reports:** Open a GitHub issue
- **Discussions:** Use GitHub Discussions tab

---

**Last Updated:** 2025-11-22

This guide is maintained for AI assistants (like Claude Code) to understand the repository structure, development workflows, and key conventions when working with this codebase.
