# ASCII Art Tools (Static)

Quick reference for terminal-based ASCII art generation. These are standalone CLI tools — no animation framework needed.

## Text Banners

**pyfiglet** (local, 571 fonts):
```bash
pip install pyfiglet --break-system-packages -q
python3 -m pyfiglet "TEXT" -f slant          # Recommended fonts: slant, doom, big, banner3, cyberlarge, 3-d, gothic
python3 -m pyfiglet --list_fonts               # List all 571 fonts
```

**asciified API** (remote, no install, 250+ fonts):
```bash
curl -s "https://asciified.thelicato.io/api/v2/ascii?text=Hello+World"
curl -s "https://asciified.thelicato.io/api/v2/ascii?text=Hello&font=Slant"
curl -s "https://asciified.thelicato.io/api/v2/fonts"  # List fonts
```

## Message Art

**cowsay** (50+ characters):
```bash
sudo apt install cowsay -y
cowsay "Hello World"              # Speech bubble
cowsay -f tux "Linux rules"       # Specific character (tux, dragon, stegosaurus, vader, etc.)
cowsay -l                          # List all characters
cowthink "Hmm..."                  # Thought bubble
```

## Decorative Borders

**boxes** (70+ designs):
```bash
sudo apt install boxes -y
echo "Hello" | boxes -d stone       # stone, parchment, cat, dog, diamonds, c-cmt, html-cmt
echo "Hello" | boxes -a c            # Center text
boxes -l                             # List all designs
```

**Combine banners + borders:**
```bash
python3 -m pyfiglet "HERMES" -f slant | boxes -d stone
```

## Colored Text Art

**TOIlet** (ANSI color effects):
```bash
sudo apt install toilet toilet-fonts -y
toilet "Hello World"                # Basic
toilet --gay "Rainbow!"             # Rainbow coloring
toilet --metal "Metal!"             # Metallic effect
toilet -F border --gay "Fancy!"     # Combined
```

## Image to ASCII

**ascii-image-converter** (recommended):
```bash
sudo snap install ascii-image-converter
ascii-image-converter image.png -C       # Color output
ascii-image-converter image.png -d 60,30 # Set dimensions
ascii-image-converter image.png -b       # Braille characters
```

**jp2a** (lightweight, JPEG only):
```bash
sudo apt install jp2a -y
jp2a --width=80 image.jpg --colors
```

## Fun Utilities

```bash
curl -s "qrenco.de/Hello+World"          # QR code as ASCII
curl -s "wttr.in/London"                # Weather as ASCII art
curl -s "wttr.in/Moon"                  # Moon phase
curl -s https://api.github.com/octocat  # Random Octocat
```

## Pre-Made ASCII Art

Fetch from ascii.co.uk:
```bash
curl -s 'https://ascii.co.uk/art/cat' -o /tmp/ascii_art.html
python3 -c "import re,html; [print(html.unescape(re.sub(r'<[^>]+>','',a)).strip()) for a in re.findall(r'<pre[^>]*>(.*?)</pre>',open('/tmp/ascii_art.html').read(),re.DOTALL) if len(a)>30]"
```

## Decision Flow

1. **Text banner** → pyfiglet (installed) or asciified API
2. **Fun character + message** → cowsay
3. **Decorative border** → boxes
4. **Specific subject art** → ascii.co.uk
5. **Image → ASCII** → ascii-image-converter
6. **QR code / weather** → qrenco.de / wttr.in
7. **Custom creative** → LLM generation with box-drawing + block Unicode chars
