# git-autosquash: Automated Git Commit Squashing Tool

https://claude.ai/chat/55ec5a5f-8ff9-4bb3-a6fe-34c87b3d920a

## Project Overview

A Node.js CLI tool for automatic, non-interactive git commit squashing based on various patterns and criteria. Solves the problem of squashing commits without interactive rebase, enabling automation and scripting.

## Core Technical Approach

### Strategy
- Use `git reset --soft` to move HEAD back while preserving changes
- Create new squashed commit with `git commit`
- Avoid `git rebase -i` entirely (requires user interaction)
- Only squash commits that are contiguous from HEAD (no history rewriting)

### Key Dependencies
```json
{
  "dependencies": {
    "simple-git": "^3.x",     // Git operations
    "commander": "^11.x",      // CLI framework
    "chalk": "^5.x",          // Colored output
    "date-fns": "^3.x"        // Date parsing
  }
}
```

## CLI API

### Core Commands

#### `--last, -n N`
Squash the last N commits
```bash
git-autosquash --last 5
git-autosquash -n 3 -m "Custom message"
```

#### `--branch, -b [BRANCH]`
Squash all commits since diverging from branch
```bash
git-autosquash --branch          # Auto-detect parent branch
git-autosquash --branch develop  # Explicit branch
git-autosquash -b -m "Feature complete"
```

#### `--containing, -c STRING`
Squash commits containing STRING in message (if contiguous from HEAD)
```bash
git-autosquash --containing "WIP"
git-autosquash -c "fixup" -m "Bug fixes"
```

#### `--after-date, -a DATE`
Squash commits after specified date (if contiguous from HEAD)
```bash
git-autosquash --after-date "2024-01-15"
git-autosquash --after-date "yesterday"
git-autosquash -a "2 days ago"
```

### Options

#### `--message, -m STRING`
Override auto-generated commit message
```bash
git-autosquash --last 5 --message "Feature: authentication system"
```

#### `--dry-run, -d`
Preview what would be squashed without executing
```bash
git-autosquash --containing "WIP" --dry-run
```

#### `--force, -f`
Skip confirmation prompts and safety checks
```bash
git-autosquash --branch main --force  # Allow squashing on main
```

### Auto-generated Commit Messages

When `--message` not provided:

| Command | Auto-message Pattern | Example |
|---------|---------------------|---------|
| `--last N` (N=1) | Keep original message | "Fix: navbar styling" |
| `--last N` (N>1) | "Squashed N commits: [first summary]" | "Squashed 5 commits: Add user auth" |
| `--branch` | "Feature: [current-branch-name]" | "Feature: user-authentication" |
| `--containing` | "Squashed N commits containing '[STRING]'" | "Squashed 3 commits containing 'WIP'" |
| `--after-date` | "Squashed commits since [DATE]" | "Squashed commits since 2024-01-15" |

## Behavior Rules

### Contiguous-only Squashing
- Only squashes commits that form unbroken chain from HEAD
- Non-contiguous matches: Reports "No contiguous commits matching criteria from HEAD"
- Not treated as error - tool simply reports and exits cleanly

### Safety Checks
1. Working directory must be clean (no uncommitted changes)
2. Creates backup tag before operations: `backup/pre-squash-[timestamp]`
3. Protected branches (main/master) require `--force`
4. Skips merge commits by default

### Error Conditions
- Uncommitted changes present
- No commits match criteria
- Protected branch without `--force`
- Invalid date format
- Git repository not found

## Implementation Details

### Project Structure
```
git-autosquash/
├── package.json
├── tsconfig.json         # Use provided config
├── README.md
├── bin/
│   └── git-autosquash    # Executable wrapper
├── proj/
│   ├── index.ts          # Entry point
│   ├── cli.ts            # Commander setup
│   ├── core/
│   │   ├── squasher.ts   # Main squashing logic
│   │   ├── analyzer.ts   # Commit analysis
│   │   └── git.ts        # Git operations wrapper
│   └── utils/
│       ├── messages.ts   # Message generation
│       └── logger.ts     # Output formatting
```

### Core Algorithm

```typescript
// Pseudo-code for main squashing logic
async function squash(options) {
  // 1. Validate clean working directory
  if (!isWorkingDirectoryClean()) throw new Error("Uncommitted changes");
  
  // 2. Get commits based on criteria
  const commits = await getCommits(options);
  
  // 3. Check if contiguous from HEAD
  if (!areCommitsContiguousFromHead(commits)) {
    console.log("No contiguous commits matching criteria from HEAD");
    return;
  }
  
  // 4. Create backup tag
  await createBackupTag();
  
  // 5. Reset soft to target
  await git.reset(['--soft', `HEAD~${commits.length}`]);
  
  // 6. Create squashed commit
  const message = options.message || generateMessage(commits, options);
  await git.commit(message);
  
  // 7. Report success
  console.log(`Squashed ${commits.length} commits`);
}
```

### Package.json Configuration
```json
{
  "name": "git-autosquash",
  "version": "1.0.0",
  "description": "Automatic, non-interactive git commit squashing",
  "bin": {
    "git-autosquash": "./bin/git-autosquash",
    "gas": "./bin/git-autosquash"
  },
  "engines": {
    "node": ">=14.0.0"
  },
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  }
}
```

## Usage Examples

### Development Workflow
```bash
# Squash WIP commits before PR
git-autosquash --containing "WIP" -m "Feature: Add user authentication"

# Squash all work on feature branch
git checkout feature/auth
git-autosquash --branch

# Preview squashing last day's work
git-autosquash --after-date "yesterday" --dry-run
```

### Automation
```bash
# In CI/CD pipeline
npx git-autosquash --branch main --force -m "Release: v1.2.0"

# Git alias
git config --global alias.squash '!npx git-autosquash'
```

## Edge Cases and Limitations

### Supported
- Squashing up to 1000 commits (practical limit)
- Commits with multiline messages
- Empty commit messages (rare but possible)
- Symlinks and binary files

### Not Supported (v1)
- Non-contiguous commit squashing
- Squashing across merge commits
- Interactive commit selection
- Squashing in middle of history (requires rewriting)
- Conflict resolution (operation aborts)

### Platform Considerations
- Windows: Git bash recommended
- Line endings: Handled by git config
- File paths: Use Node.js path module

## Error Messages

Clear, actionable messages:
```
Error: Uncommitted changes detected
  Run 'git stash' or commit changes before squashing

Error: No commits found on current branch since main
  Ensure you're on a feature branch with commits

Info: No contiguous commits containing 'WIP' from HEAD
  Found 3 matching commits but they are not contiguous
```

## Future Enhancements (Not v1)

1. `--grep` with regex support
2. `--interactive` mode for commit selection
3. `--squash-groups` to squash multiple contiguous groups
4. Configuration file support
5. Undo command using backup tags

## Testing Strategy

- Unit tests for commit analysis logic
- Integration tests with real git repos
- Edge case coverage (empty repos, single commit, etc.)
- Cross-platform testing (Windows, Mac, Linux)

# https://claude.ai/chat/995b8344-d63e-4daf-9ff1-e7a2184e245e

## Architecture for Dual-Use

### 1. Separate Core Logic from CLI

```typescript
// src/index.ts - Library entry point
export { Squasher, SquashOptions, SquashResult } from './core/squasher';
export { CommitAnalyzer } from './core/analyzer';
export { GitError, SquashError } from './core/errors';

// src/cli.ts - CLI entry point (not exported)
import { Command } from 'commander';
import { Squasher } from './core/squasher';
```

### 2. Package.json Setup

```json
{
  "main": "dist/index.js",      // Library entry
  "types": "dist/index.d.ts",   // TypeScript support
  "bin": {
    "git-autosquash": "./bin/git-autosquash"
  },
  "exports": {
    ".": {
      "require": "./dist/index.js",
      "import": "./dist/index.mjs",
      "types": "./dist/index.d.ts"
    }
  }
}
```

### 3. Core API Design

```typescript
// Programmatic API
interface SquashOptions {
  last?: number;
  branch?: string;
  containing?: string;
  afterDate?: Date | string;
  message?: string;
  force?: boolean;
  gitDir?: string;  // For library users
}

class Squasher {
  constructor(options?: { gitDir?: string }) {}
  
  async squash(options: SquashOptions): Promise<SquashResult> {}
  async analyze(options: SquashOptions): Promise<CommitInfo[]> {}
  async validateWorkingDirectory(): Promise<void> {}
}
```

## Critical Design Decisions

### 1. **Error Handling Divergence**
- CLI: Pretty-print errors, exit codes
- Library: Throw typed exceptions with error codes
- Solution: Error transformer layer in CLI

### 2. **Output/Logging**
- CLI: Console output, progress indicators
- Library: Return structured data, optional event emitter
```typescript
// Library usage
const squasher = new Squasher();
squasher.on('progress', (msg) => customLogger(msg));
```

### 3. **Git Directory Resolution**
- CLI: Use cwd, walk up to find .git
- Library: Accept explicit gitDir parameter
```typescript
// Library usage
const result = await squasher.squash({
  last: 5,
  gitDir: '/path/to/repo'
});
```

## Potential Issues

1. **Dependency Weight**: simple-git is 2.5MB. Consider using Node's child_process directly for library users who care about bundle size.

2. **State Management**: CLI can use global state; library must be stateless/instance-based.

3. **Testing Complexity**: Need separate test suites for CLI behavior vs library API.

4. **Version Conflicts**: If a project using your library also uses simple-git, version mismatches could occur.

## Alternative Approach

Consider making the CLI a separate package that depends on the core library:
- `git-autosquash` - Core library
- `git-autosquash-cli` - CLI wrapper

This is cleaner but requires maintaining two packages. The monorepo approach (single package, dual entry points) is simpler for v1.

## Recommendation

Start with the monorepo approach. Key principles:
1. Zero CLI-specific code in core modules
2. All console output through injectable logger
3. All process.exit() calls isolated to CLI layer
4. Return rich result objects, not just success/failure

This architecture supports both `npx git-autosquash --last 5` and:
```javascript
import { Squasher } from 'git-autosquash';
const squasher = new Squasher();
const result = await squasher.squash({ last: 5 });
```

The document doesn't describe the monorepo approach I mentioned. Key changes needed:

## 1. Project Structure
Current structure mixes CLI and library concerns. Should be:

```
git-autosquash/
├── src/                  # Not "proj/"
│   ├── index.ts         # Library exports only
│   ├── cli.ts           # CLI-specific code
│   ├── core/
│   │   ├── squasher.ts  # Pure logic, no console
│   │   ├── analyzer.ts
│   │   └── git.ts
│   └── utils/
│       └── logger.ts    # Interface, not implementation
```

## 2. Add Library API Section

```markdown
## Library API

### Installation
```bash
npm install git-autosquash
```

### Programmatic Usage
```typescript
import { Squasher } from 'git-autosquash';

const squasher = new Squasher({ gitDir: '/path/to/repo' });

// Analyze without squashing
const commits = await squasher.analyze({ last: 5 });

// Perform squash
const result = await squasher.squash({
  branch: 'main',
  message: 'Custom message'
});

// Result structure
{
  squashedCount: 5,
  newCommitSha: 'abc123...',
  backupTag: 'backup/pre-squash-1234567890'
}
```
```

## 3. Core Algorithm Issues

The pseudo-code has console.log directly in core logic:
```typescript
console.log("No contiguous commits matching criteria from HEAD");
```

Should be:
```typescript
return { 
  status: 'no-match', 
  reason: 'non-contiguous',
  matchedCount: 3 
};
```

## 4. Missing Package.json Fields

Add:
```json
{
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": ["dist", "bin"]
}
```

## 5. Error Handling Ambiguity

Document shows error messages but doesn't specify error types/codes for library users. Need:
```typescript
export enum SquashErrorCode {
  DIRTY_WORKING_DIR = 'DIRTY_WORKING_DIR',
  NO_COMMITS = 'NO_COMMITS',
  PROTECTED_BRANCH = 'PROTECTED_BRANCH'
}
```

The current document is CLI-focused. These changes make the dual-use architecture explicit.

Yes, significantly cleaner. Compare real implementations:

## With simple-git
```typescript
async getCommits(count: number): Promise<Commit[]> {
  const log = await this.git.log({ maxCount: count });
  return log.all.map(c => ({
    hash: c.hash,
    message: c.message,
    date: new Date(c.date)
  }));
}

async squash(commits: Commit[], message: string): Promise<void> {
  await this.git.reset(['--soft', `HEAD~${commits.length}`]);
  await this.git.commit(message);
}
```

## Without simple-git
```typescript
async getCommits(count: number): Promise<Commit[]> {
  try {
    const output = execSync(
      `git log --format="%H%x00%s%x00%aI" -n ${count}`,
      { encoding: 'utf8', cwd: this.gitDir }
    );
    
    return output.trim().split('\n').map(line => {
      const [hash, message, date] = line.split('\x00');
      return { hash, message, date: new Date(date) };
    });
  } catch (error) {
    if (error.code === 'ENOENT') throw new Error('Git not found');
    if (error.status === 128) throw new Error('Not a git repository');
    throw error;
  }
}
```

## The real complexity comes in:

1. **Error differentiation**: simple-git parses stderr to identify specific git failures
2. **Working directory checks**: `git.status()` returns structured data vs parsing `--porcelain` output
3. **Cross-platform**: Windows command quoting, path separators handled automatically
4. **Edge cases**: Empty repos, detached HEAD, all have clean APIs

For a tool doing 10+ different git operations, simple-git eliminates ~200 lines of error-prone parsing code. The dependency weight concern is largely theoretical for a CLI tool.

Use simple-git.