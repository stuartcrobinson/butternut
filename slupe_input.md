=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 02:32:04.198 pm
---------------------
fmt ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts - Read access denied for
=== END ===
Looking at your test case and the desired output format, I can see what you're trying to achieve. The new format groups all operations on the same file together, showing which blocks affected that file and whether they succeeded or failed. This is indeed more concise than the current implementation which shows each operation separately.

Let me analyze the test case to make sure I understand correctly:

1. `fs1` fails on file1.txt (old_text appears 2 times)
2. `fs2` fails on file2.txt (old_text not found)
3. `fs3` succeeds on file3.txt
4. `fs4` fails on file1.txt (old_text not found)
5. `fs5` succeeds reading file1.txt

The expected output groups operations by file:
- For file1.txt: shows `[fs1 ❌, fs4 ❌, fs5 ✅]` - two failed replacements and one successful read
- For file2.txt: shows `[fs2 ❌]` - one failed replacement
- file3.txt is not shown because fs3 succeeded (and successful replacements don't show output)

This makes sense! The format is more concise and shows file contents once per file rather than once per operation.

Here's the updated `formatFullOutput` function to implement this new format:

```sh nesl
#!nesl [@three-char-SHA-256: fmt]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/src/formatters.ts"
old_text = <<'EOT_fmt'
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

  // Add parse errors - group by blockId
  if (orchResult.parseErrors) {
    const errorsByBlock = new Map<string, any[]>();

    // Group errors by blockId
    for (const error of orchResult.parseErrors) {
      const blockId = error.blockId || 'unknown';
      if (!errorsByBlock.has(blockId)) {
        errorsByBlock.set(blockId, []);
      }
      errorsByBlock.get(blockId)!.push(error);
    }

    // Format grouped errors
    for (const [blockId, errors] of errorsByBlock) {
      const firstError = errors[0];
      const action = firstError.action || '-';
      const lineInfo = firstError.blockStartLine ? ` (line ${firstError.blockStartLine})` : '';

      // Pad action to 10 characters for alignment
      const paddedAction = action.padEnd(10);

      if (errors.length === 1) {
        // Single error
        lines.push(`${blockId} ❌ ${paddedAction} ERROR: ${firstError.message}${lineInfo}`);
      } else {
        // Multiple errors - count unique messages
        const messageCount = new Map<string, number>();
        for (const error of errors) {
          const msg = error.message;
          messageCount.set(msg, (messageCount.get(msg) || 0) + 1);
        }

        // First line shows total count
        lines.push(`${blockId} ❌ ${paddedAction} ERROR: ${errors.length} syntax errors${lineInfo}`);

        // Sub-bullets for each unique error type
        const indent = ' '.repeat(20); // Align with ERROR: column
        for (const [msg, count] of messageCount) {
          if (count > 1) {
            lines.push(`${indent}- ${msg} (${count} occurrences)`);
          } else {
            lines.push(`${indent}- ${msg}`);
          }
        }
      }
    }
  }

  lines.push('=== END ===', '', '=== OUTPUTS ===');

  // Add outputs for successful actions based on output_display rules
  if (orchResult.results) {
    for (const result of orchResult.results) {
      // Special case: show file contents for failed text replacements
      if (!result.success &&
        ['file_replace_text', 'file_replace_all_text'].includes(result.action) &&
        result.error?.includes('not found in file') &&
        result.params?.path) {
        lines.push('', `[${result.blockId}] ${result.action} ${result.params.path} (failed - showing file contents):`);
        try {
          const content = readFileSync(result.params.path, 'utf-8');
          lines.push(`=== START FILE: ${result.params.path} ===`);
          lines.push(content);
          lines.push(`=== END FILE: ${result.params.path} ===`);
        } catch (err: any) {
          lines.push(`[Error reading file: ${err.message}]`);
        }
        continue;
      }

      if (result.data && shouldShowOutput(result.action)) {
        // Special formatting for failed exec commands
        if (result.action === 'exec' && !result.success) {
          lines.push('', `[${result.blockId}] exec ${result.params.lang || 'bash'} (failed):`);
          lines.push('command:');
          lines.push(result.data.command || result.params.code || '[command not available]');
          lines.push('');

          if (result.data.stdout) {
            lines.push('stdout:');
            lines.push(result.data.stdout.trimEnd());
            lines.push('');
          }

          if (result.data.stderr) {
            lines.push('stderr:');
            // Indent stderr content for clarity
            const stderrLines = result.data.stderr.trimEnd().split('\n');
            lines.push(...stderrLines.map((line: string) => '  ' + line));
            lines.push('');
          }

          if (result.data.exit_code !== undefined) {
            lines.push(`exit code: ${result.data.exit_code}`);
          }
          continue;
        }

        // Normal formatting for successful actions
        const primaryParam = getPrimaryParamFromResult(result);
        // For file read operations, don't include path in header since it's shown in the formatted output
        const includeParam = !['file_read', 'file_read_numbered', 'files_read'].includes(result.action);
        const header = (primaryParam && includeParam)
          ? `[${result.blockId}] ${result.action} ${primaryParam}:`
          : `[${result.blockId}] ${result.action}:`;
        lines.push('', header);

        // Special formatting for file read operations
        if (['file_read', 'file_read_numbered', 'files_read'].includes(result.action)) {
          const formattedOutput = formatFileReadOutput(result);
          lines.push(...formattedOutput);
        } else if (typeof result.data === 'string') {
          lines.push(result.data.trimEnd());
        } else if (result.data.stdout !== undefined || result.data.stderr !== undefined) {
          if (result.data.stdout) {
            lines.push(`stdout:\n${result.data.stdout.trimEnd()}`);
          }
          if (result.data.stderr) {
            lines.push(`stderr:\n${result.data.stderr.trimEnd()}`);
          }
        } else {
          lines.push(JSON.stringify(result.data, null, 2));
        }
      }
    }
  }

  lines.push('=== END ===');
  return lines.join('\n');
}
EOT_fmt'
new_text = <<'EOT_fmt'
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

  // Add parse errors - group by blockId
  if (orchResult.parseErrors) {
    const errorsByBlock = new Map<string, any[]>();

    // Group errors by blockId
    for (const error of orchResult.parseErrors) {
      const blockId = error.blockId || 'unknown';
      if (!errorsByBlock.has(blockId)) {
        errorsByBlock.set(blockId, []);
      }
      errorsByBlock.get(blockId)!.push(error);
    }

    // Format grouped errors
    for (const [blockId, errors] of errorsByBlock) {
      const firstError = errors[0];
      const action = firstError.action || '-';
      const lineInfo = firstError.blockStartLine ? ` (line ${firstError.blockStartLine})` : '';

      // Pad action to 10 characters for alignment
      const paddedAction = action.padEnd(10);

      if (errors.length === 1) {
        // Single error
        lines.push(`${blockId} ❌ ${paddedAction} ERROR: ${firstError.message}${lineInfo}`);
      } else {
        // Multiple errors - count unique messages
        const messageCount = new Map<string, number>();
        for (const error of errors) {
          const msg = error.message;
          messageCount.set(msg, (messageCount.get(msg) || 0) + 1);
        }

        // First line shows total count
        lines.push(`${blockId} ❌ ${paddedAction} ERROR: ${errors.length} syntax errors${lineInfo}`);

        // Sub-bullets for each unique error type
        const indent = ' '.repeat(20); // Align with ERROR: column
        for (const [msg, count] of messageCount) {
          if (count > 1) {
            lines.push(`${indent}- ${msg} (${count} occurrences)`);
          } else {
            lines.push(`${indent}- ${msg}`);
          }
        }
      }
    }
  }

  lines.push('=== END ===', '', '=== OUTPUTS ===');

  // Group operations by file path for concise output
  const fileOperations = new Map<string, { blockId: string, action: string, success: boolean }[]>();

  // Collect all file operations that need to show content
  if (orchResult.results) {
    for (const result of orchResult.results) {
      // Check if this operation should show file contents
      let shouldShow = false;
      let filePath = '';

      // Failed replace operations where text wasn't found
      if (!result.success &&
        ['file_replace_text', 'file_replace_all_text'].includes(result.action) &&
        result.error?.includes('not found in file') &&
        result.params?.path) {
        shouldShow = true;
        filePath = result.params.path;
      }
      // Failed replace operations with duplicate text
      else if (!result.success &&
        ['file_replace_text', 'file_replace_all_text'].includes(result.action) &&
        result.error?.includes('appears') &&
        result.error?.includes('times') &&
        result.params?.path) {
        shouldShow = true;
        filePath = result.params.path;
      }
      // Successful file_read operations
      else if (result.success && result.action === 'file_read' && result.params?.path) {
        shouldShow = true;
        filePath = result.params.path;
      }

      if (shouldShow && filePath) {
        if (!fileOperations.has(filePath)) {
          fileOperations.set(filePath, []);
        }
        fileOperations.get(filePath)!.push({
          blockId: result.blockId,
          action: result.action,
          success: result.success
        });
      }
    }
  }

  // Output grouped file contents
  for (const [filePath, operations] of fileOperations) {
    // Format the operations list: [blockId ✅/❌, ...]
    const opsStr = operations
      .map(op => `${op.blockId} ${op.success ? '✅' : '❌'}`)
      .join(', ');

    lines.push('', `[${opsStr}] ${filePath}:`);
    
    try {
      const content = readFileSync(filePath, 'utf-8');
      lines.push(`=== START FILE: ${filePath} ===`);
      lines.push(content);
      lines.push(`=== END FILE: ${filePath} ===`);
    } catch (err: any) {
      lines.push(`[Error reading file: ${err.message}]`);
    }
  }

  // Handle other output types (non-file operations)
  if (orchResult.results) {
    for (const result of orchResult.results) {
      // Skip file operations we already handled
      if (['file_read', 'file_replace_text', 'file_replace_all_text'].includes(result.action)) {
        continue;
      }

      if (result.data && shouldShowOutput(result.action)) {
        // Special formatting for failed exec commands
        if (result.action === 'exec' && !result.success) {
          lines.push('', `[${result.blockId}] exec ${result.params.lang || 'bash'} (failed):`);
          lines.push('command:');
          lines.push(result.data.command || result.params.code || '[command not available]');
          lines.push('');

          if (result.data.stdout) {
            lines.push('stdout:');
            lines.push(result.data.stdout.trimEnd());
            lines.push('');
          }

          if (result.data.stderr) {
            lines.push('stderr:');
            const stderrLines = result.data.stderr.trimEnd().split('\n');
            lines.push(...stderrLines.map((line: string) => '  ' + line));
            lines.push('');
          }

          if (result.data.exit_code !== undefined) {
            lines.push(`exit code: ${result.data.exit_code}`);
          }
          continue;
        }

        // Handle other output types
        const primaryParam = getPrimaryParamFromResult(result);
        const includeParam = !['file_read_numbered', 'files_read'].includes(result.action);
        const header = (primaryParam && includeParam)
          ? `[${result.blockId}] ${result.action} ${primaryParam}:`
          : `[${result.blockId}] ${result.action}:`;
        lines.push('', header);

        if (['file_read_numbered', 'files_read'].includes(result.action)) {
          const formattedOutput = formatFileReadOutput(result);
          lines.push(...formattedOutput);
        } else if (typeof result.data === 'string') {
          lines.push(result.data.trimEnd());
        } else if (result.data.stdout !== undefined || result.data.stderr !== undefined) {
          if (result.data.stdout) {
            lines.push(`stdout:\n${result.data.stdout.trimEnd()}`);
          }
          if (result.data.stderr) {
            lines.push(`stderr:\n${result.data.stderr.trimEnd()}`);
          }
        } else {
          lines.push(JSON.stringify(result.data, null, 2));
        }
      }
    }
  }

  lines.push('=== END ===');
  return lines.join('\n');
}
EOT_fmt'
#!end_fmt
```

The key changes in this updated implementation:

1. **Grouping by file**: Instead of showing each operation separately, we group all operations that affect the same file together.

2. **Consolidated block IDs**: The format `[fs1 ❌, fs4 ❌, fs5 ✅]` shows all blocks that touched that file with their success/failure status.

3. **Single file content display**: Each file's content is shown only once, even if multiple operations affected it.

4. **Cleaner logic**: The code now:
   - First collects all file operations that need to show content
   - Groups them by file path
   - Shows the content once per file with all relevant block IDs

5. **Handles both failure types**: It catches both "not found in file" errors and "appears X times" errors for file replacement operations.

This new format is indeed more concise and easier to read, especially when multiple operations affect the same file. The test case you provided makes perfect sense with this new format.