---
title: Gum TUI Patterns
type: reference
permalink: projects/blogg/patterns/gum-tui-patterns
entity_type: pattern
status: evergreen
tags:
- domain/tui
- domain/charmbracelet-gum
---

# Gum TUI Patterns (charmbracelet/gum v0.17.0)

Battle-tested patterns for building fast, beautiful terminal UIs with gum.

## Installation

```bash
brew install gum           # macOS
go install github.com/charmbracelet/gum@latest  # Go
```

## Core Patterns

### Styled Output
```bash
gum style --foreground 212 --border double --padding "1 2" --bold "Title"
```

### Interactive Selection
```bash
echo "Option A\nOption B" | gum choose --header "Pick one:" --cursor ">" --height 10
```

### Filtered Search
```bash
echo "items..." | gum filter --placeholder "Search..." --limit 1
```

### Text Input
```bash
gum input --placeholder "Enter value..." --width 60 --char-limit 200
```

### Confirmation
```bash
gum confirm --default=false --affirmative "Yes" --negative "Cancel"
```

### Spinner/Loading
```bash
gum spin --spinner dot --title "Loading..." -- long_command
```

### Multi-column Layout
```bash
left=$(gum style --width 30 --border normal "Left")
right=$(gum style --width 30 --border normal "Right")
gum join --horizontal "$left" "$right"
```

## Performance Tips

- Cache expensive data, display from cache immediately
- Background-refresh stale caches (`&` + `disown`)
- Use `gum spin` only when fetch > 100ms
- Pre-compute at install time, not at runtime

## Relations

- used_by [[TUI Team Browser Architecture]]
- related_to [[CLI Performance Analysis]]
