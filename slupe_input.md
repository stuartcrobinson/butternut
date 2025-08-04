=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 09:43:34.118 pm
---------------------
x2m ❌ file_replace_text /Users/stuart/repos/slupe-ascii-demo/src/artist.py - Read access denied for
p9k ❌ file_replace_text /Users/stuart/repos/slupe-ascii-demo/src/artist.py - Read access denied for
=== END ===
```sh nesl
#!nesl [@three-char-SHA-256: x2m]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe-ascii-demo/src/artist.py"
old_text = <<'EOT_x2m'
def draw_robot():
    robot = """
  [o_o]-
  <| |>
   / \\
    """
    return robot
EOT_x2m
new_text = <<'EOT_x2m'
def draw_robot():
    robot = f"""
  {Fore.YELLOW}[o_o]-{Style.RESET_ALL}
  {Fore.GREEN}<| |>{Style.RESET_ALL}
   {Fore.BLUE}/ \\{Style.RESET_ALL}
    """
    return robot
EOT_x2m
#!end_x2m
```

```sh nesl
#!nesl [@three-char-SHA-256: p9k]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe-ascii-demo/src/artist.py"
old_text = <<'EOT_p9k'
    print(Fore.WHITE + draw_robot())
EOT_p9k
new_text = <<'EOT_p9k'
    print(draw_robot())
EOT_p9k
#!end_p9k
```