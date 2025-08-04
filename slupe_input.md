=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 01:51:46.611 pm
---------------------
fs5 ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts - Read access denied for
=== END ===
I see, the second replacement worked and I can see the current state. Now I need to fix the first occurrence in `formatSummary`:

```sh nesl
#!nesl [@three-char-SHA-256: fs5]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts"
old_text = <<'EOT_fs5'
  // Add execution results
  if (orchResult.results) {
    for (const result of orchResult.results) {
      const icon = result.success ? '✅' : '❌';
      const primaryParam = getPrimaryParamFromResult(result);

      if (result.success) {
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam}`.trim());
      } else {
EOT_fs5
new_text = <<'EOT_fs5'
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
EOT_fs5
#!end_fs5
```

Now let's run the test again:

```bash
cd /Users/stuart/repos/slupe && npx vitest run --hideSkippedTests -t "files-read-partial-failure"
```