=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 09:47:34.802 pm
---------------------
v8m ❌ file_replace_text /Users/stuart/repos/slupe-ascii-demo/src/artist.py - Read access denied for
=== END ===
The issue might be that `autoreset=True` in the colorama init is resetting colors too early. Let me try a different approach with explicit color codes in the string:

```sh nesl
#!nesl [@three-char-SHA-256: v8m]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe-ascii-demo/src/artist.py"
old_text = <<'EOT_v8m'
def draw_robot():
    robot = f"""
  {Fore.YELLOW}[{Fore.GREEN}o_o{Fore.YELLOW}]
  {Fore.BLUE}<| |>
   {Fore.RED}/ \\
    """
    return robot
EOT_v8m
new_text = <<'EOT_v8m'
def draw_robot():
    lines = []
    lines.append(f"  {Fore.YELLOW}[{Fore.GREEN}o_o{Fore.YELLOW}]")
    lines.append(f"  {Fore.BLUE}<| |>")
    lines.append(f"   {Fore.RED}/ \\")
    return "\n".join(lines)
EOT_v8m
#!end_v8m
```