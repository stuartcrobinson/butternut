=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 01:50:58.130 pm
---------------------
fs3 ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts - Read access denied for
fs4 ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts - Read access denied for
=== END ===
You're right! I need to fix the first replacement. Since it appears twice (once in `formatSummary` and once in `formatFullOutput`), I need to be more specific:

```sh nesl
#!nesl [@three-char-SHA-256: fs3]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts"
old_text = <<'EOT_fs3'
  // Add execution results
  if (orchResult.results) {
    for (const result of orchResult.results) {
      const icon = result.success ? '✅' : '❌';
      const primaryParam = getPrimaryParamFromResult(result);

      if (result.success) {
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam}`.trim());
      } else {
EOT_fs3
new_text = <<'EOT_fs3'
  // Add execution results
  if (orchResult.results) {
    for (const result of orchResult.results) {
      let icon = result.success ? '✅' : '❌';
      const primaryParam = getPrimaryParamFromResult(result);

      // Check for partial success in files_read
      if (result.success && result.action === 'files_read' && result.data?.errors) {
        icon = '⚠️ ';
        const totalFiles = (result.data.paths?.length || 0) + (result.data.errors?.length || 0);
        const successCount = result.data.paths?.length || 0;
        const failCount = result.data.errors?.length || 0;
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam} - Read ${successCount} of ${totalFiles} files (${failCount} failed)`.trim());
      } else if (result.success) {
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam}`.trim());
      } else {
EOT_fs3
#!end_fs3
```

Now I need to do the same update in the `formatFullOutput` function:

```sh nesl
#!nesl [@three-char-SHA-256: fs4]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts"
old_text = <<'EOT_fs4'
export function formatFullOutput(orchResult: OrchestratorResult): string {
  const lines = ['=== SLUPE RESULTS ==='];

  // Handle hook errors first
  if (orchResult.hookErrors?.before) {
    for (const error of orchResult.hookErrors.before) {
      lines.push(`def ❌ - Hook failed: ${error.command}`);
      lines.push(`          Error: ${error.error}`);
      if (error.stderr?.trim()) {
        lines.push(`          stderr: ${error.stderr.trim()}`);
      }
      if (error.stdout?.trim()) {
        lines.push(`          stdout: ${error.stdout.trim()}`);
      }
    }
  }

  // Add execution results
  if (orchResult.results) {
    for (const result of orchResult.results) {
      const icon = result.success ? '✅' : '❌';
      const primaryParam = getPrimaryParamFromResult(result);

      if (result.success) {
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam}`.trim());
      } else {
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam} - ${getErrorSummary(result.error, result)}`.trim());
      }
    }
  }
EOT_fs4
new_text = <<'EOT_fs4'
export function formatFullOutput(orchResult: OrchestratorResult): string {
  const lines = ['=== SLUPE RESULTS ==='];

  // Handle hook errors first
  if (orchResult.hookErrors?.before) {
    for (const error of orchResult.hookErrors.before) {
      lines.push(`def ❌ - Hook failed: ${error.command}`);
      lines.push(`          Error: ${error.error}`);
      if (error.stderr?.trim()) {
        lines.push(`          stderr: ${error.stderr.trim()}`);
      }
      if (error.stdout?.trim()) {
        lines.push(`          stdout: ${error.stdout.trim()}`);
      }
    }
  }

  // Add execution results
  if (orchResult.results) {
    for (const result of orchResult.results) {
      let icon = result.success ? '✅' : '❌';
      const primaryParam = getPrimaryParamFromResult(result);

      // Check for partial success in files_read
      if (result.success && result.action === 'files_read' && result.data?.errors) {
        icon = '⚠️ ';
        const totalFiles = (result.data.paths?.length || 0) + (result.data.errors?.length || 0);
        const successCount = result.data.paths?.length || 0;
        const failCount = result.data.errors?.length || 0;
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam} - Read ${successCount} of ${totalFiles} files (${failCount} failed)`.trim());
      } else if (result.success) {
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam}`.trim());
      } else {
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam} - ${getErrorSummary(result.error, result)}`.trim());
      }
    }
  }
EOT_fs4
#!end_fs4
```

Now let's run the test again:

```bash
cd /Users/stuart/repos/slupe && npx vitest run --hideSkippedTests -t "files-read-partial-failure"
```