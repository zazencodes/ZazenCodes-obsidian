---
date: 2024-03-02
tags:
    - fact
hubs:
    - "[[macos]]"
---


# MacOS-fonts

Install nerd fonts with homebrew, e.g.
```
brew tap homebrew/cask-fonts
brew install --cask font-dejavu-sans-mono-nerd-font
```

Find font family name
```
fc-list | grep "DejaVu"
```
e.g.
```
/Users/alex/Library/Fonts/DejaVuSansMNerdFontPropo-Regular.ttf: DejaVuSansM Nerd Font Propo:style=Regular
/Users/alex/Library/Fonts/DejaVuSansMNerdFontMono-Regular.ttf: DejaVuSansM Nerd Font Mono:style=Regular
/Users/alex/Library/Fonts/DejaVuSansMNerdFont-Regular.ttf: DejaVuSansM Nerd Font:style=Regular
/Users/alex/Library/Fonts/DejaVuSansMNerdFont-Bold.ttf: DejaVuSansM Nerd Font:style=Bold
/Users/alex/Library/Fonts/DejaVuSansMNerdFont-Oblique.ttf: DejaVuSansM Nerd Font:style=Oblique
/Users/alex/Library/Fonts/DejaVuSansMNerdFontPropo-BoldOblique.ttf: DejaVuSansM Nerd Font Propo:style=Bold Oblique
/Users/alex/Library/Fonts/DejaVuSansMNerdFontMono-BoldOblique.ttf: DejaVuSansM Nerd Font Mono:style=Bold Oblique
/Users/alex/Library/Fonts/DejaVuSansMNerdFontMono-Bold.ttf: DejaVuSansM Nerd Font Mono:style=Bold
/Users/alex/Library/Fonts/DejaVuSansMNerdFontPropo-Oblique.ttf: DejaVuSansM Nerd Font Propo:style=Oblique
/Users/alex/Library/Fonts/DejaVuSansMNerdFont-BoldOblique.ttf: DejaVuSansM Nerd Font:style=Bold Oblique
/Users/alex/Library/Fonts/DejaVuSansMNerdFontPropo-Bold.ttf: DejaVuSansM Nerd Font Propo:style=Bold
/Users/alex/Library/Fonts/DejaVuSansMNerdFontMono-Oblique.ttf: DejaVuSansM Nerd Font Mono:style=Oblique
```

Configure in `~/.config/alacritty.toml`

```toml
[font]
size = 15

[font.normal]
family = "DejaVuSansM Nerd Font Mono"
# style = "Regular"

[font.bold]
family = "DejaVuSansM Nerd Font Mono"
# style = "Bold"

[font.italic]
family = "DejaVuSansM Nerd Font Mono"
# style = "Italic"
```


