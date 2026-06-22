# Decoration Component Library

Generator picks: 1 shape + 1 behavior + 1 position + 1 scale → assembles HTML.
All CSS is pre-tested. Never invent decoration CSS — only use components from this library.

## Shapes

### circle
```html
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};border-radius:50%;{{BEHAVIOR}};{{POSITION}};pointer-events:none;z-index:0;"></div>
```

### arc
```html
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};border:2.5px solid rgba(var(--accent-rgb),{{OPACITY}});border-radius:50%;{{POSITION}};pointer-events:none;z-index:0;"></div>
```

### blob
```html
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};border-radius:40% 60% 55% 45%;{{BEHAVIOR}};{{POSITION}};pointer-events:none;z-index:0;"></div>
```

### dot-grid
```html
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};background-image:radial-gradient(circle,rgba(var(--accent-rgb),{{OPACITY}}) 1.2px,transparent 1.2px);background-size:20px 20px;{{POSITION}};pointer-events:none;z-index:0;"></div>
```

### line-horizontal
```html
<div style="position:absolute;width:{{SIZE}};height:2px;background:rgba(var(--accent-rgb),{{OPACITY}});{{POSITION}};pointer-events:none;z-index:0;"></div>
```

### ring
```html
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};border:1.5px solid rgba(var(--accent-rgb),{{OPACITY}});border-radius:50%;{{POSITION}};pointer-events:none;z-index:0;"></div>
```

### diamond
```html
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};transform:rotate(45deg);border:2px solid rgba(var(--accent-rgb),{{OPACITY}});{{POSITION}};pointer-events:none;z-index:0;"></div>
```

## Behaviors (replaces {{BEHAVIOR}})

### gradient-fill
`background:radial-gradient(circle,rgba(var(--accent-rgb),{{OPACITY}}),transparent 65%)`

### solid-fill
`background:rgba(var(--accent-rgb),{{OPACITY}})`

### glow
`background:radial-gradient(circle,rgba(var(--accent-rgb),{{OPACITY}}) 0%,transparent 70%);filter:blur(30px)`

### border-only
(empty — shape already has border defined)

## Positions (replaces {{POSITION}})

### corner-TR
`top:-80px;right:-80px`

### corner-BL
`bottom:-60px;left:-60px`

### corner-TL
`top:-100px;left:-100px`

### corner-BR
`bottom:-80px;right:-60px`

### center-behind
`top:50%;left:50%;transform:translate(-50%,-50%)`

### edge-right
`top:20%;right:-40px`

### edge-left
`top:30%;left:-60px`

### edge-bottom
`bottom:-40px;left:30%`

## Scales (replaces {{SIZE}})

### subtle
`200px`

### medium
`400px`

### bold
`600px`

### hero
`800px`

## Opacity Presets (replaces {{OPACITY}})

### light-theme-atmosphere: 0.20
### light-theme-texture: 0.08
### light-theme-border: 0.18
### dark-theme-atmosphere: 0.10
### dark-theme-texture: 0.04
### dark-theme-border: 0.15

## Assembly Example

To create a "medium glow blob in the bottom-left corner on a light background":
1. Shape: blob → `border-radius:40% 60% 55% 45%`
2. Behavior: glow → `background:radial-gradient(...);filter:blur(30px)`
3. Position: corner-BL → `bottom:-60px;left:-60px`
4. Scale: medium → `400px`
5. Opacity: light-theme-atmosphere → `0.20`

Result:
```html
<div style="position:absolute;width:400px;height:400px;border-radius:40% 60% 55% 45%;background:radial-gradient(circle,rgba(var(--accent-rgb),0.20) 0%,transparent 70%);filter:blur(30px);bottom:-60px;left:-60px;pointer-events:none;z-index:0;"></div>
```
