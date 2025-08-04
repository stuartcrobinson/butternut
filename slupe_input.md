=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 01:47:46.677 pm
---------------------
fs1 ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts - Read access denied for
fs2 ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts - Read access denied for
=== END ===
I see the issue now. Looking at the `formatters.ts` file, I can see that the listener component needs to be updated to handle the new partial failure case for `files_read`. The formatter is showing it as a simple success (✅) when it should show a warning (⚠️) with additional details.

Let me update the `formatSummary` function in the formatters to handle the partial failure case:

```sh nesl
#!nesl [@three-char-SHA-256: fs1]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts"
old_text = <<'EOT_fs1'
      const icon = result.success ? '✅' : '❌';
      const primaryParam = getPrimaryParamFromResult(result);

      if (result.success) {
        lines.push(`${result.blockId} ${icon} ${result.action} ${primaryParam}`.trim());
      } else {
EOT_fs1
new_text = <<'EOT_fs1'
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
EOT_fs1
#!end_fs1
```

Now I need to update the `formatFileReadOutput` function to handle the errors in the output section:

```sh nesl
#!nesl [@three-char-SHA-256: fs2]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts"
old_text = <<'EOT_fs2'
  } else if (result.action === 'files_read') {
    // Multiple files read - data contains { paths: string[], content: string[] }
    // Each element in content array corresponds to the file at the same index in paths
    if (result.data.paths && result.data.content) {
      lines.push(`Reading ${result.data.paths.length} files:`);

      // List all files first
      for (const path of result.data.paths) {
        lines.push(`- ${path}`);
      }

      // Add blank line before file contents
      lines.push('');

      // Format each file's content with START/END markers
      for (let i = 0; i < result.data.paths.length; i++) {
        const path = result.data.paths[i];
        const content = result.data.content[i];

        lines.push(`=== START FILE: ${path} ===`);
        lines.push(content || '[empty file]');
        lines.push(`=== END FILE: ${path} ===`);

        // Add blank line between files (except after the last one)
        if (i < result.data.paths.length - 1) {
          lines.push('');
        }
      }
    } else {
      // Fallback for unexpected format
      lines.push(`Reading 0 files:`);
    }
EOT_fs2
new_text = <<'EOT_fs2'
  } else if (result.action === 'files_read') {
    // Multiple files read - data contains { paths: string[], content: string[], errors?: [{path, error}] }
    // Each element in content array corresponds to the file at the same index in paths
    const successCount = result.data.paths?.length || 0;
    const failCount = result.data.errors?.length || 0;
    const totalCount = successCount + failCount;

    // Handle partial failures
    if (result.data.errors && result.data.errors.length > 0) {
      lines.push(`Successfully read ${successCount} of ${totalCount} files (${failCount} failed):`);
      lines.push('');

      // List successful files
      if (successCount > 0) {
        lines.push('✅ Successfully read:');
        for (const path of result.data.paths) {
          lines.push(`- ${path}`);
        }
        lines.push('');
      }

      // List failed files
      lines.push('❌ Failed to read:');
      for (const error of result.data.errors) {
        lines.push(`- ${error.path}: ${error.error}`);
      }
      lines.push('');

      // Show content of successful files
      if (successCount > 0) {
        for (let i = 0; i < result.data.paths.length; i++) {
          const path = result.data.paths[i];
          const content = result.data.content[i];

          lines.push(`=== START FILE: ${path} ===`);
          lines.push(content || '[empty file]');
          lines.push(`=== END FILE: ${path} ===`);

          // Add blank line between files (except after the last one)
          if (i < result.data.paths.length - 1) {
            lines.push('');
          }
        }
      }
    } else if (result.data.paths && result.data.content) {
      // All files read successfully
      lines.push(`Reading ${result.data.paths.length} files:`);

      // List all files first
      for (const path of result.data.paths) {
        lines.push(`- ${path}`);
      }

      // Add blank line before file contents
      lines.push('');

      // Format each file's content with START/END markers
      for (let i = 0; i < result.data.paths.length; i++) {
        const path = result.data.paths[i];
        const content = result.data.content[i];

        lines.push(`=== START FILE: ${path} ===`);
        lines.push(content || '[empty file]');
        lines.push(`=== END FILE: ${path} ===`);

        // Add blank line between files (except after the last one)
        if (i < result.data.paths.length - 1) {
          lines.push('');
        }
      }
    } else {
      // Fallback for unexpected format
      lines.push(`Reading 0 files:`);
    }
EOT_fs2
#!end_fs2
```

Now let's run the test again to see if it passes:

```bash
cd /Users/stuart/repos/slupe && npx vitest run --hideSkippedTests -t "files-read-partial-failure"
```