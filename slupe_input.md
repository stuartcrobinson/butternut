=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 02:35:37.432 pm
---------------------
tc1 ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md - Read access denied for
tc2 ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md - Read access denied for
tc3 ❌ file_read /Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md - Read access denied for
=== END ===
Looking at the test failures, I can see that the test cases need to be updated to match the new grouped output format. The key differences are:

1. For failed file replacements: The format should be `[fn2 ❌] /path/to/file:` instead of `[fn2] file_replace_text /path/to/file (failed - showing file contents):`
2. For successful file reads: The format should be `[rf2 ✅] /path/to/file:` instead of `[rf2] file_read:`

Let me update the test cases to match the new format:

```sh nesl
#!nesl [@three-char-SHA-256: tc1]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md"
old_text = <<'EOT_tc1'
#### Expected Output File
````sh
=== SLUPE RESULTS ===
fn1 ✅ file_write /tmp/t_listener_no_match/readme.md
fn2 ❌ file_replace_text /tmp/t_listener_no_match/readme.md - old_text not found in file
=== END ===

=== OUTPUTS ===

[fn2] file_replace_text /tmp/t_listener_no_match/readme.md (failed - showing file contents):
=== START FILE: /tmp/t_listener_no_match/readme.md ===
# Project README

This is a sample project.

## Installation

Run the following command:
- npm install

## Usage

Start the application with:
- npm start
=== END FILE: /tmp/t_listener_no_match/readme.md ===
=== END ===
````
EOT_tc1'
new_text = <<'EOT_tc1'
#### Expected Output File
````sh
=== SLUPE RESULTS ===
fn1 ✅ file_write /tmp/t_listener_no_match/readme.md
fn2 ❌ file_replace_text /tmp/t_listener_no_match/readme.md - old_text not found in file
=== END ===

=== OUTPUTS ===

[fn2 ❌] /tmp/t_listener_no_match/readme.md:
=== START FILE: /tmp/t_listener_no_match/readme.md ===
# Project README

This is a sample project.

## Installation

Run the following command:
- npm install

## Usage

Start the application with:
- npm start
=== END FILE: /tmp/t_listener_no_match/readme.md ===
=== END ===
````
EOT_tc1'
#!end_tc1
```

```sh nesl
#!nesl [@three-char-SHA-256: tc2]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md"
old_text = <<'EOT_tc2'
#### Expected Output File
````sh
=== SLUPE RESULTS ===
rf1 ✅ file_write /tmp/t_listener_read/sample.py
rf2 ✅ file_read /tmp/t_listener_read/sample.py
=== END ===

=== OUTPUTS ===

[rf2] file_read:
=== START FILE: /tmp/t_listener_read/sample.py ===
#!/usr/bin/env python3
"""Sample Python file for testing."""

def greet(name):
    """Return a greeting message."""
    return f"Hello, {name}!"

if __name__ == "__main__":
    print(greet("World"))
=== END FILE: /tmp/t_listener_read/sample.py ===
=== END ===
````
EOT_tc2'
new_text = <<'EOT_tc2'
#### Expected Output File
````sh
=== SLUPE RESULTS ===
rf1 ✅ file_write /tmp/t_listener_read/sample.py
rf2 ✅ file_read /tmp/t_listener_read/sample.py
=== END ===

=== OUTPUTS ===

[rf2 ✅] /tmp/t_listener_read/sample.py:
=== START FILE: /tmp/t_listener_read/sample.py ===
#!/usr/bin/env python3
"""Sample Python file for testing."""

def greet(name):
    """Return a greeting message."""
    return f"Hello, {name}!"

if __name__ == "__main__":
    print(greet("World"))
=== END FILE: /tmp/t_listener_read/sample.py ===
=== END ===
````
EOT_tc2'
#!end_tc2
```

Now I need to check if the "failed-file-replace-text-multiple-matches" test case needs updating. Looking at the test output, it seems this test might be passing already since it's not showing file contents (because the error is about multiple matches, not "not found"). But let me check if there's an expected output section for it that needs updating:

```sh nesl
#!nesl [@three-char-SHA-256: tc3]
action = "file_read"
path = "/Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md"
#!end_tc3
```