=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 01:40:38.848 pm
---------------------
ft1 ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md - Read access denied for
=== END ===
I'll add a test case for `files_read` with partial failures to the listener workflow integration tests. This will ensure the output formatting works correctly when some files are missing.

```sh nesl
#!nesl [@three-char-SHA-256: ft1]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md"
old_text = <<'EOT_ft1'
### listener-parsing-errors
EOT_ft1
new_text = <<'EOT_ft1'
### files-read-partial-failure

#### Initial Content
````sh
Testing files read with some missing files.
````

#### New Content
````sh
Testing files read with some missing files.

```sh nesl
#!nesl [@three-char-SHA-256: pf1]
action = "file_write"
path = "/tmp/t_listener_partial_read/exists1.txt"
content = <<'EOT_pf1'
This is the first file that exists.
It has multiple lines.
Line 3 here.
EOT_pf1
#!end_pf1
```

```sh nesl
#!nesl [@three-char-SHA-256: pf2]
action = "file_write"
path = "/tmp/t_listener_partial_read/exists2.txt"
content = <<'EOT_pf2'
Second file content.
Also exists successfully.
EOT_pf2
#!end_pf2
```

```sh nesl
#!nesl [@three-char-SHA-256: pf3]
action = "files_read"
paths = <<'EOT_pf3'
/tmp/t_listener_partial_read/exists1.txt
/tmp/t_listener_partial_read/missing1.txt
/tmp/t_listener_partial_read/exists2.txt
/tmp/t_listener_partial_read/missing2.txt
/tmp/t_listener_partial_read/also_missing.txt
EOT_pf3
#!end_pf3
```
````

#### Expected Prepended Results
````sh
=== SLUPE RESULTS ===
pf1 ✅ file_write /tmp/t_listener_partial_read/exists1.txt
pf2 ✅ file_write /tmp/t_listener_partial_read/exists2.txt
pf3 ⚠️  files_read (5 files) - Read 2 of 5 files (3 failed)
=== END ===
Testing files read with some missing files.

```sh nesl
#!nesl [@three-char-SHA-256: pf1]
action = "file_write"
path = "/tmp/t_listener_partial_read/exists1.txt"
content = <<'EOT_pf1'
This is the first file that exists.
It has multiple lines.
Line 3 here.
EOT_pf1
#!end_pf1
```

```sh nesl
#!nesl [@three-char-SHA-256: pf2]
action = "file_write"
path = "/tmp/t_listener_partial_read/exists2.txt"
content = <<'EOT_pf2'
Second file content.
Also exists successfully.
EOT_pf2
#!end_pf2
```

```sh nesl
#!nesl [@three-char-SHA-256: pf3]
action = "files_read"
paths = <<'EOT_pf3'
/tmp/t_listener_partial_read/exists1.txt
/tmp/t_listener_partial_read/missing1.txt
/tmp/t_listener_partial_read/exists2.txt
/tmp/t_listener_partial_read/missing2.txt
/tmp/t_listener_partial_read/also_missing.txt
EOT_pf3
#!end_pf3
```
````

#### Expected Output File
````sh
=== SLUPE RESULTS ===
pf1 ✅ file_write /tmp/t_listener_partial_read/exists1.txt
pf2 ✅ file_write /tmp/t_listener_partial_read/exists2.txt
pf3 ⚠️  files_read (5 files) - Read 2 of 5 files (3 failed)
=== END ===

=== OUTPUTS ===

[pf3] files_read:
Successfully read 2 of 5 files (3 failed):

✅ Successfully read:
- /tmp/t_listener_partial_read/exists1.txt
- /tmp/t_listener_partial_read/exists2.txt

❌ Failed to read:
- /tmp/t_listener_partial_read/missing1.txt: ENOENT: no such file or directory, open '/tmp/t_listener_partial_read/missing1.txt'
- /tmp/t_listener_partial_read/missing2.txt: ENOENT: no such file or directory, open '/tmp/t_listener_partial_read/missing2.txt'
- /tmp/t_listener_partial_read/also_missing.txt: ENOENT: no such file or directory, open '/tmp/t_listener_partial_read/also_missing.txt'

=== START FILE: /tmp/t_listener_partial_read/exists1.txt ===
This is the first file that exists.
It has multiple lines.
Line 3 here.
=== END FILE: /tmp/t_listener_partial_read/exists1.txt ===

=== START FILE: /tmp/t_listener_partial_read/exists2.txt ===
Second file content.
Also exists successfully.
=== END FILE: /tmp/t_listener_partial_read/exists2.txt ===
=== END ===
````

### listener-parsing-errors
EOT_ft1
#!end_ft1
```

This test case demonstrates:

1. **Summary Line**: Shows `⚠️` (warning) instead of ✅ or ❌, with a clear summary: "Read 2 of 5 files (3 failed)"

2. **Detailed Output**: 
   - Clear summary at the top showing success/failure counts
   - Separate sections for successful and failed files
   - Full error messages for each failed file
   - Complete content for successfully read files

3. **User-Friendly Format**:
   - Uses visual indicators (✅/❌) for clarity
   - Groups successes and failures separately
   - Shows the actual file content only for files that were successfully read

The warning symbol (⚠️) indicates partial success, which is more accurate than showing complete success (✅) or complete failure (❌) for this scenario.