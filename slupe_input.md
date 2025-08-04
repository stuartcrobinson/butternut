=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 02:37:19.055 pm
---------------------
tc4 ❌ file_replace_text /Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md - Read access denied for
=== END ===
You're right! Looking at the excerpt, I can see the "failed-file-replace-text-multiple-matches" test case. It currently shows no outputs, but based on the new format, it should show the file contents when a replacement fails due to multiple matches. Let me fix it:

```sh nesl
#!nesl [@three-char-SHA-256: tc4]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe/proj/comp/listener/test-data/integration/listener-workflow-v2.cases.md"
old_text = <<'EOT_tc4'
#### Expected Output File
````sh
=== SLUPE RESULTS ===
fm1 ✅ file_write /tmp/t_listener_multi_match/app.js
fm2 ❌ file_replace_text /tmp/t_listener_multi_match/app.js - old_text appears 2 times, must appear exactly once
=== END ===

=== OUTPUTS ===
=== END ===
````
EOT_tc4'
new_text = <<'EOT_tc4'
#### Expected Output File
````sh
=== SLUPE RESULTS ===
fm1 ✅ file_write /tmp/t_listener_multi_match/app.js
fm2 ❌ file_replace_text /tmp/t_listener_multi_match/app.js - old_text appears 2 times, must appear exactly once
=== END ===

=== OUTPUTS ===

[fm2 ❌] /tmp/t_listener_multi_match/app.js:
=== START FILE: /tmp/t_listener_multi_match/app.js ===
// Application code
function process() {
  const value = 100;
  console.log(value);
  
  if (value > 50) {
    console.log("High value");
  }
  
  return value;
}

function validate() {
  const value = 100;
  return value > 0;
}
=== END FILE: /tmp/t_listener_multi_match/app.js ===
=== END ===
````
EOT_tc4'
#!end_tc4
```

This update adds the expected file contents output for the failed replacement due to multiple matches, following the new grouped format `[fm2 ❌] /path/to/file:`.