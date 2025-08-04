=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 10:13:39.959 pm
---------------------
d7p ❌ file_replace_text /Users/stuart/repos/slupe-ascii-demo/src/artist.py - Read access denied for
=== END ===
```sh nesl
#!nesl [@three-char-SHA-256: d7p]
action = "file_replace_text"
path = "/Users/stuart/repos/slupe-ascii-demo/src/artist.py"
old_text = <<'EOT_d7p'
def draw_robot(frame=0):
    frames = [
        """
        ◇◆◇
       ◆◇◆◇◆
        ◇◆◇
         |
  [o_o]
  <| |>
   / \\
    """,
        """
        ◆◇◆
       ◇◆◇◆◇
        ◆◇◆
         |
     [o_o]
     \\| |/
      / \\
    """,
        """
        ◇◆◇
       ◆◇◆◇◆
        ◇◆◇
         |
          [o_o]
          /| |\\
           / \\
    """,
        """
        ◆◇◆
       ◇◆◇◆◇
        ◆◇◆
         |
[^_^]
\\| |/
 \\ /
    """
    ]
    return frames[frame % len(frames)]
EOT_d7p
new_text = <<'EOT_d7p'
def draw_robot(frame=0):
    frames = [
        """
        ◇◆◇
       ◆◇◆◇◆
        ◇◆◇
         |
[o_o]
<| |>
 / \\
    """,
        """
        ◆◇◆
       ◇◆◇◆◇
        ◆◇◆
         |
  [o_o]
  \\| |/
   / \\
    """,
        """
        ◇◆◇
       ◆◇◆◇◆
        ◇◆◇
         |
    [o_o]
    /| |\\
     / \\
    """,
        """
        ◆◇◆
       ◇◆◇◆◇
        ◆◇◆
         |
      [^_^]
      \\| |/
       \\ /
    """,
        """
        ◇◆◇
       ◆◇◆◇◆
        ◇◆◇
         |
    [o_o]
    <| |>
     / \\
    """,
        """
        ◆◇◆
       ◇◆◇◆◇
        ◆◇◆
         |
  [o_o]
  /| |\\
   / \\
    """
    ]
    return frames[frame % len(frames)]
EOT_d7p
#!end_d7p
```