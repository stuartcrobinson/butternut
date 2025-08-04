=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 09:45:07.701 pm
---------------------
x9k ❌ file_replace_text /Users/stuart/repos/slupe-ascii-demo/src/artist.py - Read access denied for
=== END ===
```sh nesl
#!nesl [@three-char-SHA-256: x9k]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe-ascii-demo/src/artist.py"
old_text = <<'EOT_x9k'
def draw_robot():
    robot = """
  [o_o]
  <| |>
   / \\
    """
    return robot
EOT_x9k
new_text = <<'EOT_x9k'
def draw_robot():
    robot = f"""
  {Fore.YELLOW}[{Fore.GREEN}o_o{Fore.YELLOW}]
  {Fore.BLUE}<| |>
   {Fore.RED}/ \\
    """
    return robot
EOT_x9k
#!end_x9k
```