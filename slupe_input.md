=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 09:58:31.696 pm
---------------------
g5p ❌ file_replace_text /Users/stuart/repos/slupe-ascii-demo/src/artist.py - Read access denied for
=== END ===
```sh nesl
#!nesl [@three-char-SHA-256: g5p]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe-ascii-demo/src/artist.py"
old_text = <<'EOT_g5p'
def render_art(art_text, line_colors=None):
    lines = art_text.strip().split('\n')
    
    if line_colors:
        for i, line in enumerate(lines):
            if i < len(line_colors):
                print(line_colors[i] + line + Style.RESET_ALL)
            else:
                print(line)
    else:
        print(art_text.strip())
EOT_g5p
new_text = <<'EOT_g5p'
def render_art(art_text, line_colors=None):
    lines = art_text.strip('\n').split('\n')
    
    if line_colors:
        for i, line in enumerate(lines):
            if i < len(line_colors):
                print(line_colors[i] + line + Style.RESET_ALL)
            else:
                print(line)
    else:
        for line in lines:
            print(line)
EOT_g5p
#!end_g5p
```