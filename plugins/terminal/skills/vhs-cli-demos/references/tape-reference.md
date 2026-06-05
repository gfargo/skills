# VHS tape DSL reference

A `.tape` is a sequence of commands executed top to bottom against a live PTY.
Comments start with `#`. Generate a starter with `vhs new demo.tape`, render with
`vhs demo.tape`.

## Table of contents

- [Output commands](#output-commands)
- [Settings (`Set`)](#settings-set)
- [Typing and keys](#typing-and-keys)
- [Timing](#timing)
- [Hiding setup (`Hide` / `Show`)](#hiding-setup-hide--show)
- [Other commands](#other-commands)
- [The pixel-dimension formula](#the-pixel-dimension-formula)
- [Annotated example: a still PNG](#annotated-example-a-still-png)
- [Annotated example: a motion GIF](#annotated-example-a-motion-gif)

## Output commands

| Command | Effect |
|---|---|
| `Output demo.gif` | Record the whole session to an animated GIF. |
| `Output demo.mp4` | Record to MP4 (smaller than GIF for long/complex motion; needs a player). |
| `Output demo.webm` | Record to WebM. |
| `Output frames/` | Write the raw PNG frame sequence to a directory (advanced). |
| `Screenshot still.png` | Capture a **single frame** at this point in the script. |

You can have multiple `Output` lines (emit gif + mp4 in one run) and multiple
`Screenshot` lines (capture several stills mid-session). **`Output x.png` is NOT
a still** — it records a frame sequence directory. For a single image use
`Screenshot`.

## Settings (`Set`)

Set these near the top, before the session starts.

| Setting | Purpose | Notes |
|---|---|---|
| `Set Shell "bash"` | Shell to spawn. | `bash` is the most predictable for scripted setup. |
| `Set FontSize 14` | Font size in px. | Drives the pixel cell size — see the formula. |
| `Set FontFamily "..."` | Font. | Pick one present on the render machine. |
| `Set Width 1200` | Output width in **pixels**. | Compute from cols (see formula). |
| `Set Height 700` | Output height in **pixels**. | Compute from rows. |
| `Set Padding 30` | Window padding in px. | VHS default padding is large (~60px/side) and eats columns; set it explicitly and account for it in the size formula. |
| `Set Theme "Catppuccin Mocha"` | Terminal colour theme. | Use a named VHS theme, or a JSON palette object. Lock this for determinism. |
| `Set CursorBlink false` | Stop cursor blink. | Removes frame-to-frame noise. |
| `Set TypingSpeed 50ms` | Delay between typed chars. | Lower = faster typing. Set `0ms` for instant. |
| `Set LoopOffset 50%` | Where the GIF loop "starts" when embedded. | Cosmetic. |
| `Set PlaybackSpeed 1.0` | Speed multiplier for the final render. | |
| `Set WindowBar Colorful` | Draw a fake title bar (traffic lights). | Optional chrome; many pipelines draw their own. |
| `Set Margin` / `Set MarginFill` | Outer margin + fill colour. | Optional framing. |

## Typing and keys

| Command | Effect |
|---|---|
| `Type "text"` | Type literal characters (respects `TypingSpeed`). |
| `Type@10ms "text"` | Override typing speed for this line. |
| `Enter` | Press Return. `Enter 3` presses it 3×. |
| `Backspace` / `Tab` / `Space` / `Escape` | The named keys. All accept a count. |
| `Up` `Down` `Left` `Right` | Arrows (accept a count: `Down 5`). |
| `Ctrl+C` `Ctrl+D` `Ctrl+L` | Modifier combos. |
| `Alt+.` `Shift+Tab` | Other modifiers. |
| `PageUp` / `PageDown` | Paging keys. |
| `Sleep 800ms` / `Sleep 2s` | Pause (see Timing). |

To send an app-specific chord (two keys the app reads as a unit), type them
together: `Type "gd"` rather than `Type "g"` then `Type "d"` — though a tiny
`Sleep` between them can be useful if you *want* the intermediate state on camera.

## Timing

`Sleep` is your camera-hold. Units: `ms` or `s`. Timing is the whole craft of a
good demo:

- Lead-in / settle before the first capture: enough for cold-start + data load.
- Hold after each meaningful action so a viewer can read it.
- Keep total duration tight — every second is frames, and frames are bytes.

## Hiding setup (`Hide` / `Show`)

Everything between `Hide` and `Show` executes but is **not recorded**. Use it for
the unglamorous setup so the recording starts on a clean, ready state:

```
Hide
Type "export PATH=/repo/node_modules/.bin:$PATH" Enter   # unquoted $PATH!
Type "export MYAPP_NOW=2024-01-15T12:00:00Z" Enter        # pin time
Type "cd /tmp/fixture && clear" Enter
Show
Type "myapp ui" Enter
Sleep 2s
```

## Other commands

| Command | Effect |
|---|---|
| `Source other.tape` | Include another tape (share common setup). |
| `Require myapp` | Fail fast if `myapp` isn't on PATH before recording. |
| `Env KEY value` | Set an environment variable for the session. |

## Sizing the canvas (and the kerning trap)

VHS sizes output in *pixels* (`Set Width`/`Set Height`), but the terminal renders
a *character grid* whose cell size is determined by the font at the chosen
`FontSize`. These two don't automatically agree, and that mismatch is a real trap:

> **The kerning trap.** If you pick a Width/Height that doesn't land on a whole
> number of character cells for the font's *actual* advance width, the renderer
> stretches the glyphs to fill the canvas. The capture comes out with loose,
> spaced-out letters and broken-looking kerning. A `cols × some-multiplier`
> formula feels precise but is exactly how you fall into this — the multiplier is
> never quite the font's true cell width, so the grid is fractional and the
> glyphs stretch.

**The reliable approach: pick a `FontSize` and let VHS size the canvas.** Omit
`Set Width`/`Set Height` entirely — VHS uses its known-good defaults (1200×600 at
the default font), which render crisp, tight monospace. To make the capture
bigger or smaller, change `FontSize` (≈18-22 reads well), not the pixel
dimensions.

Only set explicit `Width`/`Height` when you genuinely need a fixed canvas across a
set of captures. Then use round numbers with a comfortable font size, and **look
at the rendered output** to confirm the glyphs are tight — never trust a computed
formula. If text looks spaced-out, your Width/Height is fighting the font: drop
them and let VHS size it.

## Annotated example: a still PNG

```
# --- determinism + framing ---
Set Shell "bash"
Set FontSize 20         # readable; canvas left to VHS defaults (crisp glyphs)
Set Padding 24
Set Theme "Catppuccin Mocha"
Set CursorBlink false
Set TypingSpeed 50ms
# No Set Width/Height — VHS defaults render tight monospace. Set them only if you
# need a fixed canvas across many shots, and eyeball the result for stretched text.

# --- hidden setup (not recorded) ---
Hide
Type "export PATH=/repo/node_modules/.bin:$PATH" Enter
Type "export MYAPP_NOW=2024-01-15T12:00:00Z" Enter   # pin relative dates
Type "cd /tmp/fixture && clear" Enter
Show

# --- the scene ---
Type "myapp ui --repo /tmp/fixture" Enter
Sleep 5s                # generous settle: cold-start + async load, stills get one frame
Type "gd"               # drive to the diff view
Sleep 1s
Screenshot myapp-diff.png
```

## Annotated example: a motion GIF

```
Set Shell "bash"
Set FontSize 20          # canvas left to VHS defaults — crisp glyphs, no kerning trap
Set Padding 24
Set Theme "Catppuccin Mocha"
Set CursorBlink false
Set TypingSpeed 50ms

Output demo-views.gif    # motion → Output (gif), not Screenshot

Hide
Type "export PATH=/repo/node_modules/.bin:$PATH" Enter
Type "cd /tmp/fixture && clear" Enter
Show

Type "myapp ui --view history" Enter
Sleep 1500ms             # shorter settle than a still — loading frames read as startup
Type "gd"                # open diff
Sleep 2400ms             # hold so the viewer can read it
Escape
Sleep 600ms
Type "gb"                # switch to branches — show contrast
Sleep 2400ms
# NOTE: no trailing quit — end on the UI, not a shell prompt
```

After rendering, optimize losslessly:

```bash
gifsicle -O3 --batch demo-views.gif     # or scripts/optimize_gif.sh demo-views.gif
```
