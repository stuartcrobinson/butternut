=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 10:18:20.098 pm
---------------------
z5n ❌ file_write /Users/stuart/repos/slupe-ascii-demo/src/artist.py - Write access denied for
=== END ===
```sh nesl
#!nesl [@three-char-SHA-256: z5n]
action = "file_write"
path = "/Users/stuart/repos/slupe-ascii-demo/src/artist.py"
content = <<'EOT_z5n'
import time
import os
from colorama import init, Fore, Back, Style

init()

def clear_screen():
    os.system('cls' if os.name == 'nt' else 'clear')

def draw_robot():
    robot = """
  [o_o]
  <| |>
   / \\
    """
    return robot

def colorize_line(line, color_map=None):
    if not color_map:
        return line
    
    colored_line = ""
    for char in line:
        color = color_map.get(char, "")
        colored_line += color + char
    return colored_line + Style.RESET_ALL

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

def main():
    frame = 0
    try:
        while True:
            clear_screen()
            print(Fore.CYAN + "Slupe Demo" + Style.RESET_ALL)
            print()
            
            robot = draw_robot()
            render_art(robot)
            
            frame += 1
            time.sleep(0.3)
    except KeyboardInterrupt:
        print("\nRobot signing off!")

if __name__ == "__main__":
    main()
EOT_z5n
#!end_z5n
```