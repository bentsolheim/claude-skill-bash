# Bash Script Best Practices

## Script Bootstrapping

### Main Function Pattern

Always use a `main` function as the entry point for your script logic. This provides clear structure and enables the script to be safely sourced by other scripts.

```bash
#!/usr/bin/env bash

function main() {
    # Main script logic goes here
    echo "Executing main logic..."
}

# Guard clause - only execute main if script is run directly
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
    exit 0
fi
```

### The Guard Clause

The `if [[ "${BASH_SOURCE[0]}" == "${0}" ]]` construct is critical for creating reusable bash scripts:

- **`${BASH_SOURCE[0]}`** - The filename of the current script
- **`${0}`** - The name of the currently executing script

When these match, the script is being executed directly. When they differ, the script is being sourced.

#### Benefits:

1. **Safe Sourcing**: Other scripts can source your script to access its functions or variables without triggering execution
2. **Testing**: Test frameworks can source your script to test individual functions
3. **Reusability**: Functions defined in the script become available to other scripts when sourced

#### Example Use Case:

```bash
# script1.sh
#!/usr/bin/env bash

function process_data() {
    echo "Processing..."
}

function main() {
    process_data
}

if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
    exit 0
fi
```

```bash
# script2.sh can safely source script1.sh
#!/usr/bin/env bash

source ./script1.sh  # Won't execute main, but makes process_data() available
process_data         # Now we can use the function
```

### Explicit Exit

Always include an explicit `exit 0` after calling main to ensure proper exit codes and prevent any code below from accidentally executing.

## Usage Function and Argument Parsing

### Usage Function

Define a `usage()` function near the top of your script, after any global variables but before other functions. This provides clear documentation and a consistent help interface.

```bash
#!/usr/bin/env bash

SCRIPT_NAME=$(basename "$0")
VERSION="1.0.0"

function usage() {
    cat <<EOM

Processes data files and generates reports.

usage: ${SCRIPT_NAME} [options]

options:
    -i|--input        <file>     Input file to process (required)
    -o|--output       <file>     Output file (optional, defaults to stdout)
    -f|--format       <type>     Output format: json|csv|text (default: text)
    -v|--verbose                 Enable verbose output
    -h|--help                    Show this help message
    --version                    Show version information

examples:
    ${SCRIPT_NAME} -i data.txt -o report.json -f json
    ${SCRIPT_NAME} --input data.txt --verbose
    cat data.txt | ${SCRIPT_NAME} -f csv > report.csv

EOM
    exit 1
}
```

Key elements of a good usage function:
- Use a heredoc (`cat <<EOM`) for clean formatting
- Include a brief description of what the script does
- List all options with clear descriptions
- Show practical examples
- Exit with status 1 (error) when called for help

### Argument Parsing in Main

Parse arguments at the beginning of your `main` function using a `while` loop with `case` statements:

```bash
function main() {
    # Default values
    local input_file=""
    local output_file=""
    local format="text"
    local verbose=false
    
    # Parse arguments
    while [ "$1" != "" ]; do
        case $1 in
        -i | --input)
            shift
            input_file="$1"
            ;;
        -o | --output)
            shift
            output_file="$1"
            ;;
        -f | --format)
            shift
            format="$1"
            ;;
        -v | --verbose)
            verbose=true
            ;;
        --version)
            echo "${SCRIPT_NAME} version ${VERSION}"
            exit 0
            ;;
        -h | --help)
            usage
            ;;
        *)
            echo "Error: Unknown option '$1'"
            usage
            ;;
        esac
        shift
    done
    
    # Validate required arguments
    if [ -z "$input_file" ]; then
        echo "Error: Input file is required"
        usage
    fi
    
    # Main script logic
    process_file "$input_file" "$output_file" "$format" "$verbose"
}
```

Best practices for argument parsing:
- Set default values for optional parameters
- Use `local` for function variables
- Handle both short (`-i`) and long (`--input`) option forms
- Call `shift` twice for options with values
- Validate required parameters after parsing
- Call `usage` for unknown options or missing requirements
- Keep the parsing logic clean and consistent

## Dependency Declaration and Checking

### Declaring Dependencies

Declare external tools your script depends on using a global `DEPENDENCIES` array at the top of your script. This makes requirements explicit and enables automatic validation.

```bash
#!/usr/bin/env bash

# Declare required external tools
DEPENDENCIES=(jq curl git docker)

SCRIPT_NAME=$(basename "$0")
VERSION="1.0.0"
```

### Dependency Validation Function

Include a dependency checking function that validates all required tools are available:

```bash
function exit_on_missing_tools() {
    for cmd in "$@"; do
        if command -v "$cmd" &>/dev/null; then
            continue
        fi
        printf "Error: Required tool '%s' is not installed or not in PATH\n" "$cmd"
        exit 1
    done
}
```

### Including Dependencies in Usage

Always list dependencies in your usage function to help users understand requirements:

```bash
function usage() {
    cat <<EOM

Processes data files and generates reports.

usage: ${SCRIPT_NAME} [options]

options:
    -i|--input        <file>     Input file to process (required)
    -o|--output       <file>     Output file (optional, defaults to stdout)
    -h|--help                    Show this help message

dependencies: ${DEPENDENCIES[@]}

EOM
    exit 1
}
```

### Complete Script Structure with Dependencies

Here's how to properly structure a script with dependency checking:

```bash
#!/usr/bin/env bash

# Global declarations
DEPENDENCIES=(jq curl git)
SCRIPT_NAME=$(basename "$0")

function usage() {
    cat <<EOM

My script description here.

usage: ${SCRIPT_NAME} [options]

options:
    -h|--help         Show this help message

dependencies: ${DEPENDENCIES[@]}

EOM
    exit 1
}

function exit_on_missing_tools() {
    for cmd in "$@"; do
        if command -v "$cmd" &>/dev/null; then
            continue
        fi
        printf "Error: Required tool '%s' is not installed or not in PATH\n" "$cmd"
        exit 1
    done
}

function main() {
    # Parse arguments
    while [ "$1" != "" ]; do
        case $1 in
        -h | --help)
            usage
            ;;
        *)
            echo "Error: Unknown option '$1'"
            usage
            ;;
        esac
        shift
    done
    
    # Validate dependencies after argument parsing
    exit_on_missing_tools "${DEPENDENCIES[@]}"
    
    # Main script logic
    echo "All dependencies are satisfied, proceeding..."
}

# Guard clause with dependency check placed right before
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
    exit 0
fi
```

### Key Benefits

1. **Early Failure**: Scripts fail fast with clear error messages when dependencies are missing
2. **Self-Documenting**: Dependencies are explicitly declared and shown in help
3. **Reusable**: Other scripts can source your script to inspect its `DEPENDENCIES` array
4. **Maintainable**: Adding or removing dependencies is a simple array modification

### Advanced: Shared Dependency Checking

For multiple scripts in a project, consider extracting the `exit_on_missing_tools` function to a shared utilities file:

```bash
# utils.sh
function exit_on_missing_tools() {
    for cmd in "$@"; do
        if command -v "$cmd" &>/dev/null; then
            continue
        fi
        printf "Error: Required tool '%s' is not installed or not in PATH\n" "$cmd"
        exit 1
    done
}
```

```bash
# main script
#!/usr/bin/env bash

source "$(dirname "${BASH_SOURCE[0]}")/utils.sh"

DEPENDENCIES=(jq curl)

function main() {
    exit_on_missing_tools "${DEPENDENCIES[@]}"
    # ... rest of script
}
```

## Function Organization and Separation of Concerns

### Keep Main Function Lean

The `main()` function should act as a coordinator, not an implementer. It should handle:
1. Argument parsing
2. Input validation
3. Dependency checking
4. Routing to appropriate functions
5. High-level error handling

Business logic should be extracted into well-named, focused functions.

### Anti-Pattern: Everything in Main

```bash
# ❌ BAD: Main function doing too much
function main() {
    while [ "$1" != "" ]; do
        case $1 in
        -b | --backup)
            shift
            backup_dir="$1"
            ;;
        esac
        shift
    done
    
    # Business logic mixed with coordination
    echo "Starting backup..."
    tar -czf backup.tar.gz "$backup_dir"
    
    echo "Uploading to S3..."
    aws s3 cp backup.tar.gz s3://my-bucket/
    
    echo "Cleaning old backups..."
    find /tmp -name "*.tar.gz" -mtime +7 -delete
    
    echo "Sending notification..."
    mail -s "Backup complete" admin@example.com < /dev/null
}
```

### Best Practice: Separated Functions

```bash
# ✅ GOOD: Main delegates to focused functions
function create_backup() {
    local source_dir="$1"
    local backup_file="$2"
    
    echo "Creating backup of ${source_dir}..."
    if ! tar -czf "$backup_file" "$source_dir" 2>/dev/null; then
        echo "Error: Failed to create backup" >&2
        return 1
    fi
    echo "✅ Backup created: ${backup_file}"
    return 0
}

function upload_to_s3() {
    local file="$1"
    local bucket="$2"
    
    echo "Uploading to S3..."
    if ! aws s3 cp "$file" "s3://${bucket}/" 2>/dev/null; then
        echo "Error: Failed to upload to S3" >&2
        return 1
    fi
    echo "✅ Uploaded to s3://${bucket}/"
    return 0
}

function cleanup_old_backups() {
    local directory="$1"
    local days="$2"
    
    echo "Cleaning backups older than ${days} days..."
    local count=$(find "$directory" -name "*.tar.gz" -mtime +${days} | wc -l)
    find "$directory" -name "*.tar.gz" -mtime +${days} -delete
    echo "✅ Removed ${count} old backups"
    return 0
}

function send_notification() {
    local status="$1"
    local details="$2"
    
    echo "Sending notification..."
    echo "$details" | mail -s "Backup ${status}" admin@example.com
    return 0
}

function main() {
    local backup_dir=""
    local s3_bucket="my-bucket"
    local retention_days=7
    local notify=true
    
    while [ "$1" != "" ]; do
        case $1 in
        -b | --backup)
            shift
            backup_dir="$1"
            ;;
        -s | --s3-bucket)
            shift
            s3_bucket="$1"
            ;;
        -r | --retention)
            shift
            retention_days="$1"
            ;;
        --no-notify)
            notify=false
            ;;
        -h | --help)
            usage
            ;;
        *)
            echo "Error: Unknown option '$1'"
            usage
            ;;
        esac
        shift
    done
    
    if [ -z "$backup_dir" ]; then
        echo "Error: Backup directory required" >&2
        usage
    fi
    
    exit_on_missing_tools "${DEPENDENCIES[@]}"
    
    local backup_file="/tmp/backup-$(date +%Y%m%d-%H%M%S).tar.gz"
    local overall_status="SUCCESS"
    local status_details=""
    
    if ! create_backup "$backup_dir" "$backup_file"; then
        overall_status="FAILED"
        status_details="Failed to create backup"
    elif ! upload_to_s3 "$backup_file" "$s3_bucket"; then
        overall_status="FAILED"
        status_details="Failed to upload to S3"
    else
        cleanup_old_backups "/tmp" "$retention_days"
        status_details="Backup completed successfully"
    fi
    
    if [ "$notify" = true ]; then
        send_notification "$overall_status" "$status_details"
    fi
    
    rm -f "$backup_file"
    
    [ "$overall_status" = "SUCCESS" ] && exit 0 || exit 1
}
```

### Function Design Principles

#### 1. Single Responsibility
Each function should do one thing well:
```bash
# ✅ GOOD: Clear, single purpose
function validate_email() {
    local email="$1"
    [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

# ❌ BAD: Multiple responsibilities
function process_user() {
    local email="$1"
    # Validates AND creates user AND sends email - too much!
    [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]] || return 1
    echo "$email" >> users.txt
    mail -s "Welcome" "$email" < welcome.txt
}
```

#### 2. Clear Inputs and Outputs
Functions should have clear parameters and return values:
```bash
# ✅ GOOD: Clear contract
function calculate_percentage() {
    local value="$1"
    local total="$2"
    
    if [ "$total" -eq 0 ]; then
        echo "0"
        return 1
    fi
    
    local percentage=$((value * 100 / total))
    echo "$percentage"
    return 0
}

# Usage
if percentage=$(calculate_percentage 25 100); then
    echo "Percentage: ${percentage}%"
else
    echo "Error: Cannot calculate percentage"
fi
```

#### 3. Function Naming Conventions
Use descriptive verb-noun combinations:
```bash
# ✅ GOOD: Clear action and target
function create_backup() { }
function validate_input() { }
function send_notification() { }
function cleanup_temp_files() { }

# ❌ BAD: Vague or non-descriptive
function process() { }
function handle() { }
function do_stuff() { }
```

#### 4. Keep Functions Small
If a function is longer than 50 lines, consider breaking it down:
```bash
# ✅ GOOD: Composed of smaller functions
function deploy_application() {
    validate_environment || return 1
    run_tests || return 1
    build_artifacts || return 1
    upload_to_server || return 1
    restart_services || return 1
    verify_deployment || return 1
}
```

#### 5. Use Local Variables
Always declare variables as local within functions:
```bash
function process_file() {
    local input_file="$1"
    local output_file="$2"
    local temp_file=$(mktemp)
    
    # Process...
    
    rm -f "$temp_file"
}
```

### Function Ordering

Functions MUST be organized in this specific order for maximum readability:

1. **Usage function**: Always first after global declarations (before main)
2. **Main function**: Immediately after usage() so readers see the script's flow
3. **Business logic functions**: Core functionality that main() calls, ordered by importance/relevance
4. **Utility/helper functions**: Generic helpers at the end (print functions, validators, etc.)

**Important**: Do NOT add comments like "# 1. USAGE FUNCTION" or "# 2. MAIN FUNCTION" - these violate the "avoid excessive comments" rule. The function names and ordering speak for themselves.

**Correct ordering example:**
```bash
#!/usr/bin/env bash

# Global declarations
DEPENDENCIES=(jq curl)
SCRIPT_NAME=$(basename "$0")

function usage() {
    cat <<EOM
usage: ${SCRIPT_NAME} [options]
...
EOM
    exit 1
}

function main() {
    # Argument parsing
    # Dependency checking
    # Orchestration logic
}

# Business logic functions
function process_data() { }
function validate_input() { }
function generate_report() { }

# Utility functions
function exit_on_missing_tools() { }
function print_header() { }
function print_error() { }

# Guard clause at the very end
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
    exit 0
fi
```

**Acceptable section comments**: You may use minimal section markers ONLY when there are many functions and visual separation helps navigation:
```bash
# Business logic functions
function process_data() { }
function validate_input() { }
function generate_report() { }

# Utility functions  
function print_header() { }
function print_error() { }
```

This ordering ensures readers find:
- Help documentation first (usage)
- Script flow overview second (main)
- Important business logic next
- Implementation details last

### Testing Individual Functions

With proper separation, functions can be tested independently:
```bash
# test_functions.sh
source ./my_script.sh

# Test individual functions
test_create_backup() {
    local test_dir=$(mktemp -d)
    local test_file=$(mktemp)
    
    if create_backup "$test_dir" "$test_file"; then
        echo "✅ create_backup test passed"
    else
        echo "❌ create_backup test failed"
    fi
    
    rm -rf "$test_dir" "$test_file"
}

test_validate_email() {
    validate_email "user@example.com" && echo "✅ Valid email test passed"
    ! validate_email "invalid-email" && echo "✅ Invalid email test passed"
}
```

## Error Handling

### Avoid `set -e` - Use Explicit Error Handling

While `set -e` seems convenient for making scripts exit on errors, it has significant drawbacks and should generally be avoided in favor of explicit error handling.

#### Problems with `set -e`

1. **Inconsistent behavior**: Doesn't exit in many contexts (conditionals, pipelines, functions)
2. **Poor debugging**: No context about what failed or why
3. **Hidden complexity**: Future maintainers may not realize the script depends on it
4. **Unreliable**: Different shells and versions behave differently

#### Recommended: Explicit Error Handling

Handle errors explicitly with meaningful messages:

```bash
#!/usr/bin/env bash

function main() {
    # Check command success explicitly
    if ! git status &>/dev/null; then
        echo "Error: Not in a git repository" >&2
        exit 1
    fi
    
    # Capture output and check status
    output=$(docker build . 2>&1)
    if [ $? -ne 0 ]; then
        echo "Error: Docker build failed:" >&2
        echo "$output" >&2
        exit 1
    fi
    
    # Use || for simple error handling
    cd /some/directory || {
        echo "Error: Cannot change to directory /some/directory" >&2
        exit 1
    }
    
    # Chain commands with && for success dependency
    make clean && make build && make test || {
        echo "Error: Build process failed" >&2
        exit 1
    }
}
```

#### Error Handling Best Practices

1. **Always redirect errors to stderr**: Use `>&2` for error messages
2. **Provide context**: Explain what failed and possibly why
3. **Use meaningful exit codes**: Different codes for different error types
4. **Log before exiting**: Ensure errors are captured for debugging
5. **Clean up on failure**: Remove temporary files, restore state

#### When `set -e` Might Be Acceptable

Only consider `set -e` for:
- Very simple, linear scripts with no conditionals
- Quick prototypes or one-off scripts
- Scripts where any failure should stop execution

Even then, combine with:
- `set -u` (error on undefined variables)
- `set -o pipefail` (pipeline fails if any command fails)
- `set -x` for debugging (prints commands)

```bash
#!/usr/bin/env bash
# Only for simple scripts
set -euo pipefail

# Simple linear commands
compile_source
run_tests
deploy_application
```

## Code Comments

### Avoid Excessive Comments

Well-structured code with descriptive function names should be self-documenting. Reserve comments for:

```bash
# ❌ BAD: Commenting the obvious
function backup_database() {
    # Print message that we're starting backup
    echo "Starting backup..."
    
    # Run mysqldump command
    mysqldump mydb > backup.sql
    
    # Check if the command succeeded
    if [ $? -eq 0 ]; then
        # Print success message
        echo "Backup complete"
    fi
}

# ✅ GOOD: Comments only when necessary
function backup_database() {
    echo "Starting backup..."
    
    # Use --single-transaction for InnoDB consistency without locking
    mysqldump --single-transaction mydb > backup.sql
    
    if [ $? -eq 0 ]; then
        echo "Backup complete"
    fi
}
```

### When to Use Comments

- **Complex algorithms**: Explain the "why" not the "what"
- **Non-obvious workarounds**: Document why a strange approach is necessary
- **External dependencies**: Note version requirements or API quirks
- **Business logic**: Explain domain-specific rules that aren't obvious from code
- **Section markers**: Use comments to separate function groups for visual scanning

```bash
# ✅ GOOD: Explaining non-obvious logic
function calculate_retry_delay() {
    local attempt="$1"
    
    # Exponential backoff with jitter to prevent thundering herd
    local base_delay=$((2 ** attempt))
    local jitter=$((RANDOM % 1000))
    echo $((base_delay * 1000 + jitter))
}

function cleanup_old_files() {
    # Keep 7 days of logs per compliance requirement GDPR-2018-47
    find /var/log/app -mtime +7 -delete
}
```

### Section Markers

Use comments to mark major sections of your script for easier navigation:

```bash
#!/usr/bin/env bash

# Constants and configuration
SCRIPT_NAME=$(basename "$0")
DEPENDENCIES=(jq curl)

function main() {
    # Main logic here
}

# Business logic functions
function process_data() {
    # Processing logic
}

function validate_input() {
    # Validation logic
}

# Utility functions
function print_header() {
    # Output formatting
}

function log_message() {
    # Logging logic
}
```

These section markers help readers quickly locate different parts of the script, especially in longer files.

## User-Friendly Output Structure

### Organizing Script Output

Create clear, scannable output that helps users understand what's happening. Define color constants at the script's top level and organize output within functions:

```bash
#!/usr/bin/env bash

# Color definitions for terminal output (defined at top of script)
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Allow users to disable colors via environment variable
if [[ -n "${NO_COLOR:-}" ]] || [[ "${TERM:-}" == "dumb" ]]; then
    RED=""
    GREEN=""
    YELLOW=""
    BLUE=""
    NC=""
fi

SCRIPT_NAME=$(basename "$0")
DEPENDENCIES=(mysqldump aws)

function main() {
    print_header "Database Backup Script"
    
    if ! check_prerequisites; then
        print_error "Prerequisites check failed"
        exit 1
    fi
    
    local backup_file="/backups/backup-$(date +%Y%m%d-%H%M%S).sql"
    if ! perform_backup "mydb" "$backup_file"; then
        print_error "Backup failed"
        exit 1
    fi
    
    if ! upload_backup "$backup_file" "s3://my-bucket/"; then
        print_error "Upload failed"
        exit 1
    fi
    
    local size=$(du -h "$backup_file" | cut -f1)
    print_summary "$backup_file" "$size"
    
    exit 0
}

# Business logic functions
function check_prerequisites() {
    print_step 1 3 "Checking prerequisites..."
    
    if ! exit_on_missing_tools "${DEPENDENCIES[@]}"; then
        print_error "Missing required tools"
        return 1
    fi
    
    print_success "Prerequisites satisfied"
    return 0
}

function perform_backup() {
    local database="$1"
    local output_file="$2"
    
    print_step 2 3 "Backing up database..."
    
    if mysqldump "$database" > "$output_file" 2>/dev/null; then
        local size=$(du -h "$output_file" | cut -f1)
        print_success "Backup completed (${size})"
        return 0
    else
        print_error "Failed to backup database"
        return 1
    fi
}

function upload_backup() {
    local file="$1"
    local destination="$2"
    
    print_step 3 3 "Uploading to remote storage..."
    
    if aws s3 cp "$file" "$destination" 2>/dev/null; then
        print_success "Upload successful"
        return 0
    else
        print_error "Failed to upload backup"
        return 1
    fi
}

function print_summary() {
    local backup_file="$1"
    local backup_size="$2"
    
    echo
    print_header "Summary"
    print_success "Database backed up successfully"
    echo -e "   Backup size: ${backup_size}"
    echo -e "   Location: ${backup_file}"
    echo
}

# Utility functions
function print_header() {
    local title="$1"
    echo -e "${BLUE}==================================================${NC}"
    echo -e "${BLUE}${title}${NC}"
    echo -e "${BLUE}==================================================${NC}"
    echo
}

function print_step() {
    local step_num="$1"
    local total_steps="$2"
    local message="$3"
    echo -e "${YELLOW}[${step_num}/${total_steps}] ${message}${NC}"
}

function print_success() {
    local message="$1"
    echo -e "${GREEN}✅ ${message}${NC}"
}

function print_error() {
    local message="$1"
    echo -e "${RED}❌ Error: ${message}${NC}" >&2
}

function print_warning() {
    local message="$1"
    echo -e "${YELLOW}⚠️  Warning: ${message}${NC}"
}

if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
    exit 0
fi
```

### Best Practices for Output

1. **Use consistent color coding**:
   - 🔵 Blue: Headers and sections
   - 🟡 Yellow: Warnings and prompts
   - 🟢 Green: Success messages
   - 🔴 Red: Errors (always to stderr)

2. **Provide visual structure**:
   - Section separators for major parts
   - Step counters ([1/3], [2/3]) for progress
   - Indentation for details
   - Empty lines for readability

3. **Include status indicators**:
   - ✅ Success
   - ❌ Error  
   - ⚠️ Warning
   - 🔄 In progress
   - ℹ️ Information

4. **Interactive prompts**:
```bash
# Yes/no confirmation
read -p "Do you want to continue? (y/n): " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo -e "${YELLOW}Operation cancelled${NC}"
    exit 0
fi

# Menu selection
echo -e "${YELLOW}Select an option:${NC}"
echo "1) Full backup"
echo "2) Incremental backup"
echo "3) Restore from backup"
read -p "Enter choice (1-3): " choice

# Password input (hidden)
read -s -p "Enter password: " password
echo  # Add newline after password input
```

5. **Respect NO_COLOR environment variable**:
```bash
# Allow users to disable colors
if [[ -n "${NO_COLOR:-}" ]] || [[ "${TERM:-}" == "dumb" ]]; then
    RED=""
    GREEN=""
    YELLOW=""
    BLUE=""
    NC=""
fi
```

6. **Provide next steps**:
```bash
echo -e "${YELLOW}Next steps:${NC}"
echo "1. Verify the backup file:"
echo "   ${BLUE}ls -la /backups/${backup_file}${NC}"
echo "2. Test restoration process:"
echo "   ${BLUE}./restore.sh --file ${backup_file} --test${NC}"
```