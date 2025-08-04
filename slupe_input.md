=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 10:00:31.514 pm
---------------------
r2k ❌ file_replace_text /Users/stuart/repos/slupe-ascii-demo/src/artist.py - Read access denied for
=== END ===
```sh nesl
#!nesl [@three-char-SHA-256: r2k]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe-ascii-demo/src/artist.py"
old_text = <<'EOT_r2k'
    robot = draw_robot()
    
    robot_colors = [
        Fore.YELLOW,
        Fore.BLUE,
        Fore.RED
    ]
    
    render_art(robot, robot_colors)
EOT_r2k
new_text = <<'EOT_r2k'
    robot = draw_robot()
    render_art(robot)
EOT_r2k
#!end_r2k
```