=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 10:02:34.099 pm
---------------------
j8w ❌ file_replace_text /Users/stuart/repos/slupe-ascii-demo/src/artist.py - Read access denied for
=== END ===
```sh nesl
#!nesl [@three-char-SHA-256: j8w]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe-ascii-demo/src/artist.py"
old_text = <<'EOT_j8w'
    render_art(robot)
EOT_j8w
new_text = <<'EOT_j8w'
    render_art(robot, [Fore.YELLOW, Fore.BLUE, Fore.RED])
EOT_j8w
#!end_j8w
```