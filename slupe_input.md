=== SLUPE RESULTS ===
📋 Output copied to clipboard @ 09:50:00.160 pm
---------------------
t4p ❌ file_write /Users/stuart/repos/slupe-ascii-demo/src/color_test.py - Write access denied for
=== END ===
```sh nesl
#!nesl [@three-char-SHA-256: t4p]
action = "file_write"
path = "/Users/stuart/repos/slupe-ascii-demo/src/color_test.py"
content = <<'EOT_t4p'
from colorama import init, Fore, Back, Style

init()

print(Fore.RED + "This should be red")
print(Fore.GREEN + "This should be green")
print(Fore.BLUE + "This should be blue")
print(Fore.YELLOW + "This should be yellow")
print(Fore.MAGENTA + "This should be magenta")
print(Fore.CYAN + "This should be cyan")

print(Style.RESET_ALL)

print(Fore.RED + "R" + Fore.GREEN + "A" + Fore.BLUE + "I" + Fore.YELLOW + "N" + Fore.MAGENTA + "B" + Fore.CYAN + "O" + Fore.RED + "W")

print(Style.RESET_ALL)

print(Back.RED + "Red background" + Style.RESET_ALL)
print(Back.GREEN + Fore.BLACK + "Green background with black text" + Style.RESET_ALL)

print("\nSimple colored art:")
print(Fore.YELLOW + "  ☀")
print(Fore.BLUE + "~~~~")
print(Fore.GREEN + " 🌱 ")
EOT_t4p
#!end_t4p
```

Run this with:
```bash
python /Users/stuart/repos/slupe-ascii-demo/src/color_test.py
```