# Figmadeck

Generate Slidev presentations from Figma design templates.

## Usage

```
/figmadeck <figma-url>                   Create preset from Figma template
/figmadeck <figma-url> --learn=N         Create preset + learning loop
/figmadeck <outline>                     Generate presentation (auto-match preset)
/figmadeck --preset <name> <outline>     Generate with specific preset
/figmadeck --export pdf [dir]            Export to PDF
/figmadeck --dev [dir]                   Launch dev server
/figmadeck --edit [dir] <comment>        Edit presentation
```

## How It Works

1. Extract a Figma presentation template into a preset with HTML archetypes
2. Generate presentations by filling archetype slots with outline content (verbatim copy)
3. Iterative QA compares each slide against the Figma original until fidelity >= 9/10
