# Theme format

A theme is one JSON object: the pet's sprite and everything else it changes. The `preview_theme` and `set_theme` tools take it as `theme`, `get_theme` returns it, and a theme file (`<name>.theme.json`) holds the same object. Only `sprite` is required. The mod draws whatever else it gets, repairs what it can, and returns notes on what it did (see [Notes](#notes)). Unknown fields are ignored.

Four example pets ship with the plugin: [`assets/slime.json`](../../assets/slime.json) faces the viewer, and [`assets/duck.json`](../../assets/duck.json) faces left, beak first. Both set only the sprite fields. [`assets/tuxedo.json`](../../assets/tuxedo.json) is a front-facing cat drawn at 16×12 on `scale` 0.9, with a `miniSprite`. [`assets/alien.json`](../../assets/alien.json) sets every field in this file: props (including `think`), `miniSprite`, `lines`, `lineColors`, `hud`, and `scene`.

| Field | Required | What it is |
| --- | --- | --- |
| `sprite` | yes | Rows of text, one character per pixel. `.` is a clear pixel, and `*`, `+`, and `@` draw clear too. The bottom row is where the pet stands. |
| `palette` | | Maps each sprite character to a color. A character without a color draws clear. The mod keeps `*`, `+`, and `@` for its own drawings. Props and `miniSprite` use this palette too. |
| `eyes` | | Two `[x, y]` points: the top-left pixel of each eye's 2×2 pupil. `x` counts from the left and `y` from the top, both from 0. Without `eyes` the pet has no eyes and shows no faces. |
| `name` | | 1 to 24 characters. Default `"pet"`. |
| `scale` | | Default 1, where one sprite pixel is one screen pixel. A sprite too big for the scale draws at the largest scale that fits. |
| `outline` | | The palette character of the outline. It counts extra when the pet squashes, so the outline stays unbroken. |
| `eyeColor` | | The pupils' color. Default `"#000000"`. It recolors the pupils only: eye whites, stars, and hearts keep their own colors. |
| `cheeks` | | Two `[x, y]` pixels, one per cheek. |
| `cheekColor` | | The cheeks' color. Default `"#ff8aaa"`. |
| `mini` | | `{ "top", "body", "edge" }`: the three colors of the drop-shaped mini each running subagent gets. Default the slime's blues: `#9ad2ff`, `#3d84f0`, `#1e3a8a`. See [Minis](#minis). |
| `miniSprite` | | Rows of text, up to 5×7, drawn as the mini in place of the drop. See [Minis](#minis). |
| `props` | | `{ "<mode>": frames }` or `{ "<mode>": false }`: the pet's own prop for a mode, or none. See [Props](#props). |
| `lines` | | `{ "<mode>": ["text", ...] }`: the status lines of a mode. See [Status lines](#status-lines). |
| `lineColors` | | `{ "<mode>": "#rrggbb" }`: the status line's color in a mode. See [Status lines](#status-lines). |
| `hud` | | The HUD's look: its frame, and each bar's label, color, and fill. See [HUD](#hud). |
| `scene` | | The band's background: a ground, a sky drawing, obstacles the pet leaps while it runs, and decor. See [Scene](#scene). |

## Colors

Every color in a theme is `"#rrggbb"` or `"#rgb"`.

The terminal may be dark or light, so mid tones work on both. A dark outline reads on both. Text colors (`lineColors`, a bar's `color`) read best as mid tones; near-white and near-black vanish on one background. A dark pet needs a light `eyeColor`. The preview's light-terminal button shows the pet on a light background.

## Size

The pet draws on a 19×20 pixel canvas, and every pose has to fit it. The largest sprite per scale:

| `scale` | Largest sprite |
| --- | --- |
| 1 | 15×10 |
| 0.9 | 16×12 |
| 0.8 | 19×15 |

A bigger sprite still draws: the mod lowers the scale until it fits. At a scale under 1 the sprite is resampled smaller and loses detail, so a design that fits 15×10 keeps its pixels sharp at scale 1.

## Eyes

Each expression draws an eye in a 3×3 box, over the sprite. A clear pixel in the box shows the sprite beneath. The box's top-left is one pixel above the pupil's top-left. For `"eyes": [[3, 4], [9, 4]]`, the left eye's box covers `x` 3 to 5 and `y` 3 to 5:

```text
x  0123456789...
y3 ...###.......   # is the box
y4 ...PP#.......   P is the pupil the open eye draws
y5 ...PP#.......
```

Eyes read best on a flat patch of one body color with no outline in it. Boxes that overlap merge the wide eyes and the hearts into one shape. Put the pupils 4 pixels apart, for one clear column between the boxes. The duck's eyes, `[[3, 3], [7, 3]]`, do this:

```text
x  0123456789
y2 ...###.###   two boxes, one clear column between them
y3 ...PP#.PP#
y4 ...PP#.PP#
```

The hearts and stars fill the whole box, so a cheek right next to a box touches them. Cheeks read best a pixel below or beside the boxes, on body color.

## How the pet moves

The mod makes every motion and face from the one sprite. It stretches and squashes the sprite to stand, run, jump, think, and cheer, and lifts it up to 9 pixels for a jump. The preview shows each motion under the mode that plays it. A compact shape with a wide, flat bottom reads best.

A squash shrinks the sprite by up to a third, so a feature one pixel thin can vanish in it. A beak, ears, or a tail that is at least 2 pixels thick survives every pose. The duck's beak is 2 rows tall and survives; its 1-pixel tail flickers.

Other drawings share the canvas:

- A question mark and a thought trail appear near the top right while it thinks. A `think` prop replaces them (see [Props](#props)).
- `zzz` appears near the top right while it sleeps.
- Sparkles appear on both sides while it cheers.
- Sweat and dizzy marks appear at the right, around rows 12 to 14.
- A prop appears to the right, from column 17 of the 19-column canvas.

Keep important features off the sprite's right edge, where these drawings land. The duck faces left, so its tail takes that edge.

## Modes

A mode is what the pet is acting out. Props, status lines, and line colors are set per mode.

| Mode | Plays when | The mod's prop | `{}` in a line is |
| --- | --- | --- | --- |
| `idle` | Claude is idle | | |
| `sleep` | Idle for the **Sleep after** time, a minute by default | | |
| `jump` | A turn starts | | |
| `think` | Claude thinks longer | | |
| `read` | `Read` | a book | the file name |
| `search` | `Grep`, `Glob` | a magnifier | the pattern |
| `edit` | `Edit`, `MultiEdit`, `Write`, `NotebookEdit`, `TodoWrite` | a pen and an editor | the file name, or `the todo list` |
| `bash` | `Bash`, and any tool not listed | a terminal | the command's first line, or the tool's name |
| `web` | `WebFetch`, `WebSearch` | a globe | the host, or the search query |
| `agent` | A subagent starts | | |
| `run` | A tool call ends, for 4 seconds | | |
| `cheer` | A turn ends | | |
| `error` | A tool call fails | | |

A target is cut to 24 characters. A mode with no target in the last column has no `{}` value.

## Props

A prop is a drawing the pet works with. `props` maps a mode to its prop:

```json
"props": { "web": [["..y..", ".yyy.", "..y.."], ["...y.", "..yyy", "...y."]], "think": false }
```

Here `web` plays two frames of a star, drawn in the palette's `y`, and `think` draws nothing.

- A frame is a list of rows, drawn in the pet's `palette`. One frame is a list of rows. Several frames are a list of frames, up to 8, played in a loop at 4 frames a second.
- The prop sits in a box 16 pixels wide and 20 high, to the right of the pet, starting at column 17 of the canvas. It draws over the pet's two rightmost columns.
- Rows are bottom-aligned: the last row rests on the pet's bottom row. Rows of `.` at the bottom keep a drawing raised, such as a thought above the pet's head.
- Any mode except `run` may have a prop. The pet's own prop replaces the mod's for that mode. `false` leaves the mode without a prop, even one the mod gives.
- `think` has no mod prop. The question mark and thought trail are the mod's drawing for it. A `think` prop replaces them, and `"think": false` removes them.
- Like the sprite, a prop or `miniSprite` draws `*`, `+`, and `@` clear: the mod keeps them for cheeks, sparkles, and pupils.

## Minis

A mini is the small figure that joins the trail behind the pet for each running subagent, up to 6 at a time. It hops while its subagent runs, leaves with a sparkle when the subagent finishes, and turns grey when the subagent fails.

- `mini` colors the mod's drop shape: `top` is the light tip, `body` the belly, `edge` the base. All three are needed, or the minis keep the slime's blues.
- `miniSprite` replaces the drop. It is rows in the pet's `palette`, up to 5 pixels wide and 7 high. The drop is 5×5. `mini` colors are not used while there is a `miniSprite`.

## Status lines

The status line is the text beside the pet. `lines` replaces a mode's lines, and `lineColors` its color.

```json
"lines": { "read": ["sniffing {}", "page after page…"], "idle": ["purr…"] },
"lineColors": { "idle": "#f0903c" }
```

- `lines` maps a mode to a list of lines. A mode left out keeps the mod's lines, which are `LINES` in [`hooks/status.ts`](../../hooks/status.ts). Read them for tone.
- `{}` in a line stands for the mode's target (see [Modes](#modes)). Only the first `{}` in a line is filled.
- A line with `{}` is skipped while there is no target, such as when the user turns off the **Name files and commands** setting. A mode whose lines all have `{}` shows `something` in its place, so give a mode at least one line without `{}`.
- Each line is cut to 40 characters, counted before `{}` is filled.
- The line changes every 4 seconds.
- `lineColors` maps a mode to a color. A mode left out keeps the mod's color, `LINE_COLOR` in the same file. The color applies to the line and its arrow.

## HUD

The HUD is the window below the prompt, with up to three bars:

| Bar | Shows | Default label |
| --- | --- | --- |
| `hp` | The context window left | `♥ HP` |
| `mp` | The 5-hour rate limit left | `✦ MP` |
| `st` | The 7-day rate limit left | `◆ ST` |

`mp` and `st` appear only once the session has a reading for that limit, which Pro and Max plans report after a response.

```json
"hud": { "frame": "#c8681f", "hp": { "label": "LIVES", "color": "#f0903c", "fill": ["#8a4a0c", "#ffc27a"] }, "st": false }
```

- `frame` is the window frame's color. Default `#5aa9ff`.
- A bar's look is `{ "label", "color", "fill" }`, each optional.
  - `label` is up to 6 characters, each one cell wide, so the bars line up. A custom label stays as written, where the default `♥ HP` turns to `⚠ HP` under 10 %.
  - `color` is the label's color.
  - `fill` is `[from, to]`, the bar's gradient from its left end to its right while it is healthy. The reading, the bold number after the bar, takes the `to` color.
- `false` in place of a look hides that bar. When all three are hidden, the HUD is gone.
- A bar's warning colors replace its `fill` whatever the look: `hp` turns yellow at 50 % or less and red at 25 % or less, and `mp` and `st` turn red under 15 %.

## Scene

A scene is the background of the band, the area above the prompt where the pet and its status line sit. Without one the band is clear, as it is for the slime. A scene shows in a terminal; the Desktop app draws the pet without it.

```json
"scene": {
  "ground": ["rrdrrrrrdrrrrdrr", "dddddrddddddddrd"],
  "sky": [".....ddd.....", "...dkkkkkd...", "yy.kkkkkkk.yy", "..yyyyyyyyy..", "...dkkkkkd...", ".....ddd....."],
  "obstacles": [["..rr..", ".rrrd.", "rrrrdd", "rrdddd"]],
  "decor": [[".y.", "yky", ".y.", "...", "...", "...", "...", "...", "...", "...", "...", "..."]],
  "every": 44
}
```

Every drawing is rows in the pet's `palette`, like the sprite. Add a palette entry for each color the scene needs.

- `ground` is a tile up to 16×2, repeated across the band's full width in a row of its own, below the pet. The pet stands on it.
- `sky` is one drawing up to 16×12 that stays put near the band's top right: a sun, a moon, a planet.
- `obstacles` is a list of up to 4 drawings, each up to 8×6. They stand on the ground in turn, `every` columns apart, the first 24 columns from the left edge. A running pet leaps each one in its path. When it has no room to land past one, it turns around instead.
- `decor` is a list of up to 4 drawings, each up to 16×20, placed at random spots clear of the obstacles. Decor sits behind the pet and is never in its way. A drawing's bottom row sits on the ground, so rows of `.` below it raise it into the sky: a star with 9 clear rows below floats 9 pixels up. Raised decor drifts left like clouds, each drawing at its own speed of a column every 1.1 to 1.9 seconds, passes in front of the sky drawing, and comes back in from the right edge. Decor on the ground stays put.
- `every` is the columns between obstacles, from 30 to 120. Default 40.

The pet runs only between tool calls, so it leaps only then. The scene's layout depends on the terminal's width; the same width always gives the same layout. The status line covers the scene behind it.

## Notes

The tools return a note for each repair. Each row is a repair and its cause.

| Repair | Cause |
| --- | --- |
| Rows padded with `.` | Rows of different widths. |
| `*`, `+`, `@` drawn clear | The sprite, a prop, or `miniSprite` uses one of the mod's own characters. |
| A character drawn clear | It has no palette color, or its color is not a color. In a prop or `miniSprite`, the note names the drawing. |
| Scale lowered | The sprite is too big for the scale. |
| No eyes and no faces | `eyes` is not two `[x, y]` points. |
| No cheeks | `cheeks` is not two `[x, y]` pixels. |
| The slime's blue minis | `mini` lacks one of `top`, `body`, `edge`. |
| Eye box partly off the sprite | A pupil's 3×3 box extends past the sprite's edge. |
| Eye boxes merge | Pupils less than 3 pixels apart across and down. |
| Cheek inside an eye box | Some faces draw over the cheek. |
| `miniSprite` or a prop frame left out | It is not a list of text rows, or every row is empty. |
| Bottom-left part of a drawing kept | A prop frame is past 16×20, or `miniSprite` is past 5×7. |
| A prop keeps its first 8 frames | The prop has more than 8 frames. |
| A scene drawing cut, or the first 4 kept | A ground tile past 16×2, a sky drawing past 16×12, an obstacle past 8×6, decor past 16×20, or more than 4 obstacles or decor. |
| No scene | `scene` is not an object, or has no ground, sky, obstacles, or decor. |
| `every` changed | It is not a number from 30 to 120. |
| `props` left out | It is not an object. |
| A `props` entry left out | Its key is not a mode, or is `run`. |
| `lines`, `lineColors`, or `hud` left out | It is not an object. |
| A `lines` or `lineColors` entry left out | Its key is not a mode. |
| A mode keeps its own lines | `lines.<mode>` holds no text. |
| Lines cut to 40 characters | A line is longer. |
| A mode keeps its own line color | `lineColors.<mode>` is not a color. |
| The frame keeps its own color | `hud.frame` is not a color. |
| A bar keeps its own look | `hud.<bar>` is neither an object nor `false`. |
| A label cut to 6 characters | `hud.<bar>.label` is longer. |
| A bar keeps its own label color | `hud.<bar>.color` is not a color. |
| A bar keeps its own fill | `hud.<bar>.fill` is not two colors. |

The mod ignores these without a note: a palette entry for `.`, `*`, `+`, or `@`, an `outline` character that is not in `palette`, and a bar key other than `hp`, `mp`, or `st`.
