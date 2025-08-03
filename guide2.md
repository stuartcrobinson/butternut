# butternut: Design & Implementation Guide

## Overview

Node.js package providing both CLI and programmatic API for non-interactive git commit squashing. Enables automation of commit cleanup without `git rebase -i`.

## Architecture

### Dual-Use Design
- Single package exposing both CLI (`bin/butternut`) and library (`import`)
- Core logic isolated from CLI concerns
- No console output in core modules
- All git operations via simple-git

### Package Structure
```
butternut/
├── package.json
├── tsconfig.json
├── README.md
├── bin/
│   └── butternut      # #!/usr/bin/env node
├── src/
│   ├── index.ts           # Library exports only
│   ├── cli.ts             # Commander setup, console output
│   ├── core/
│   │   ├── squasher.ts    # Main logic, returns data
│   │   ├── analyzer.ts    # Commit analysis
│   │   ├── git-wrapper.ts # simple-git abstraction
│   │   └── errors.ts      # Typed errors
│   └── utils/
│       ├── messages.ts    # Message generation
│       └── logger.ts      # Logger interface
└── dist/                  # Built output
```

### Package.json
```json
{
  "name": "butternut",
  "version": "1.0.0",
  "description": "Automated git commit squashing - CLI and library",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "bin": {
    "butternut": "./bin/butternut",
    "gas": "./bin/butternut"
  },
  "files": ["dist", "bin"],
  "engines": {
    "node": ">=14.0.0"
  },
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  },
  "dependencies": {
    "simple-git": "^3.x",
    "commander": "^11.x",
    "chalk": "^5.x",
    "date-fns": "^3.x"
  },
  "devDependencies": {
    "@types/node": "^20.x",
    "typescript": "^5.x"
  }
}
```

## Library API

### Core Types
```typescript
interface SquashOptions {
  last?: number;
  branch?: string;
  containing?: string;
  afterDate?: Date | string;
  message?: string;
  force?: boolean;
  gitDir?: string;
}

interface SquashResult {
  squashedCount: number;
  newCommitSha: string;
  backupTag: string;
  originalCommits: CommitInfo[];
}

interface AnalyzeResult {
  matchingCommits: CommitInfo[];
  contiguous: boolean;
  reason?: 'non-contiguous' | 'no-matches' | 'merge-commit';
}

interface CommitInfo {
  hash: string;
  message: string;
  date: Date;
  author: string;
}
```

### Public API
```typescript
// src/index.ts
export { Squasher } from './core/squasher';
export { SquashOptions, SquashResult, AnalyzeResult } from './core/types';
export { SquashError, SquashErrorCode } from './core/errors';

// Usage
import { Squasher } from 'butternut';

const squasher = new Squasher({ 
  gitDir: '/path/to/repo',  // Optional, defaults to cwd
  logger: customLogger      // Optional, no-op by default
});

// Analyze without squashing
const analysis = await squasher.analyze({ last: 5 });
if (!analysis.contiguous) {
  console.log(`Cannot squash: ${analysis.reason}`);
}

// Perform squash
try {
  const result = await squasher.squash({ 
    branch: 'main',
    message: 'Feature complete' 
  });
  console.log(`Squashed ${result.squashedCount} commits`);
} catch (error) {
  if (error.code === 'DIRTY_WORKING_DIR') {
    // Handle specific error
  }
}
```

### Error Handling
```typescript
enum SquashErrorCode {
  DIRTY_WORKING_DIR = 'DIRTY_WORKING_DIR',
  NO_COMMITS = 'NO_COMMITS',
  PROTECTED_BRANCH = 'PROTECTED_BRANCH',
  NOT_GIT_REPO = 'NOT_GIT_REPO',
  INVALID_DATE = 'INVALID_DATE'
}

class SquashError extends Error {
  code: SquashErrorCode;
  details?: any;
}
```

## CLI API

### Commands

| Command | Description | Example |
|---------|-------------|---------|
| `--last, -n N` | Squash last N commits | `butternut -n 5` |
| `--branch, -b [BRANCH]` | Squash since branch divergence | `butternut --branch main` |
| `--containing, -c STRING` | Squash commits with STRING | `butternut -c "WIP"` |
| `--after-date, -a DATE` | Squash after date | `butternut -a "2 days ago"` |

### Options

| Option | Description |
|--------|-------------|
| `--message, -m` | Override commit message |
| `--dry-run, -d` | Preview without executing |
| `--force, -f` | Skip safety checks |
| `--no-backup` | Skip backup tag creation |

### Auto-generated Messages

| Criteria | Pattern | Example |
|----------|---------|---------|
| `--last 1` | Original message | "Fix: navbar styling" |
| `--last N` | "Squashed N commits: [first]" | "Squashed 5 commits: Add auth" |
| `--branch` | "Feature: [branch-name]" | "Feature: user-authentication" |
| `--containing` | "Squashed N commits containing '[STRING]'" | "Squashed 3 commits containing 'WIP'" |
| `--after-date` | "Squashed commits since [DATE]" | "Squashed commits since 2024-01-15" |

## Core Implementation

### Squasher Class
```typescript
// src/core/squasher.ts
export class Squasher {
  private git: SimpleGit;
  private logger: Logger;

  constructor(options?: { gitDir?: string; logger?: Logger }) {
    this.git = simpleGit(options?.gitDir || process.cwd());
    this.logger = options?.logger || new NoOpLogger();
  }

  async squash(options: SquashOptions): Promise<SquashResult> {
    // 1. Validate
    await this.validateWorkingDirectory();
    
    // 2. Analyze
    const analysis = await this.analyze(options);
    if (!analysis.contiguous) {
      throw new SquashError(
        `Cannot squash: ${analysis.reason}`,
        SquashErrorCode.NO_COMMITS
      );
    }

    // 3. Safety checks
    if (!options.force) {
      await this.checkProtectedBranch();
    }

    // 4. Create backup
    const backupTag = await this.createBackupTag();

    // 5. Perform squash
    const commits = analysis.matchingCommits;
    await this.git.reset(['--soft', `HEAD~${commits.length}`]);
    
    const message = options.message || 
      generateMessage(commits, options);
    
    await this.git.commit(message);

    // 6. Return result
    const newCommit = await this.git.revparse(['HEAD']);
    
    return {
      squashedCount: commits.length,
      newCommitSha: newCommit,
      backupTag,
      originalCommits: commits
    };
  }

  async analyze(options: SquashOptions): Promise<AnalyzeResult> {
    const commits = await this.getCommits(options);
    const contiguous = this.checkContiguous(commits);
    
    return {
      matchingCommits: commits,
      contiguous: contiguous.isContiguous,
      reason: contiguous.reason
    };
  }
}
```

### CLI Layer
```typescript
// src/cli.ts
export async function runCLI(argv: string[]) {
  const program = new Command();
  
  program
    .option('-n, --last <count>', 'Squash last N commits')
    .option('-b, --branch [branch]', 'Squash since branch')
    .option('-c, --containing <string>', 'Squash commits containing')
    .option('-a, --after-date <date>', 'Squash after date')
    .option('-m, --message <message>', 'Commit message')
    .option('-d, --dry-run', 'Preview only')
    .option('-f, --force', 'Skip safety checks');

  const options = program.parse(argv).opts();
  
  // Transform CLI options to library options
  const squashOptions: SquashOptions = {
    last: options.last ? parseInt(options.last) : undefined,
    branch: options.branch,
    containing: options.containing,
    afterDate: options.afterDate,
    message: options.message,
    force: options.force
  };

  const squasher = new Squasher({
    logger: new ConsoleLogger(chalk)
  });

  try {
    if (options.dryRun) {
      const analysis = await squasher.analyze(squashOptions);
      printAnalysis(analysis);
    } else {
      const result = await squasher.squash(squashOptions);
      printSuccess(result);
    }
  } catch (error) {
    printError(error);
    process.exit(1);
  }
}
```

## Behavior Rules

### Contiguous-Only Squashing
- Only squashes unbroken chains from HEAD
- Non-contiguous matches throw `NO_COMMITS` error with reason
- Merge commits break contiguity

### Parent Branch Detection
```typescript
async function detectParentBranch(): Promise<string> {
  // Priority order:
  // 1. origin/main if exists
  // 2. origin/master if exists  
  // 3. main if exists
  // 4. master if exists
  // 5. Throw error
}
```

### Date Parsing
Supports:
- ISO dates: "2024-01-15"
- Relative: "yesterday", "2 days ago"
- Natural: "last Monday"
- Via date-fns parse()

### Safety Features
- Backup tag: `backup/pre-squash-[timestamp]`
- Protected branches: main, master (require --force)
- Clean working directory required
- Atomic operations (all or nothing)

## Critical Decisions

### Why Not Support Non-Contiguous?
Would require full history rewriting via rebase, breaking:
- Shared branch workflows
- Existing PR references
- Tag positions
- Performance (rewriting 1000s of commits)

### Why simple-git?
- Structured error responses vs parsing stderr
- Cross-platform command construction
- Handles edge cases (empty repos, detached HEAD)
- ~10 git commands × ~20 lines each = 200 lines saved

### Why Single Package?
- Simpler deployment (one npm package)
- Easier versioning
- Library users don't install unused CLI deps
- Monorepo tools add complexity for v1

## Testing Requirements

### Unit Tests
- Commit analysis logic (contiguous checking)
- Message generation
- Date parsing
- Error scenarios

### Integration Tests
```typescript
// Test with real git repos
beforeEach(() => {
  createTempRepo();
  createCommits(['initial', 'feat: add', 'WIP 1', 'WIP 2']);
});
```

### CLI Tests
- Argument parsing
- Output formatting
- Exit codes
- --dry-run accuracy

## Performance Considerations

- Limit log retrieval: `git.log({ maxCount: 1000 })`
- Stream large outputs if needed
- Avoid loading full file contents for large repos
- Cache branch detection per invocation

## Error Messages

Library throws typed errors, CLI formats them:

```
SquashError: DIRTY_WORKING_DIR
↓ (CLI formats to) ↓
Error: Uncommitted changes detected
  Run 'git stash' or commit changes before squashing

SquashError: NO_COMMITS (reason: 'non-contiguous')  
↓ (CLI formats to) ↓
Cannot squash: Found 3 matching commits but they are not contiguous from HEAD
  Matching commits must form an unbroken chain from most recent commit
```

## Future Considerations (Not v1)

- Interactive mode for commit selection
- Config file (`.gitautosquash.json`)
- Squash multiple groups in one operation
- Custom commit message templates
- Undo command using backup tags