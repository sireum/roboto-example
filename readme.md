# Sireum Mr. Roboto Example

This repository holds an example for Sireum Mr. Roboto
(Roboto for short).

Roboto takes a demo automation script (e.g.,
[example.md](https://raw.githubusercontent.com/sireum/roboto-example/refs/heads/master/example.md))
and runs it using `java.awt.Robot` to automate GUI interactions such as
typing, keyboard shortcuts, mouse clicks, image-based template matching,
on-screen overlay notifications, and text-to-speech.

The specification language is in the form of either:

1. Slash (Slang universal shell) script that builds objects defined by the
   [Script](https://github.com/sireum/runtime/blob/master/library/shared/src/main/scala/org/sireum/roboto/Script.scala)
   Slang types (outputs JSON consumed by the runner); or,

2. Markdown with YAML frontmatter specifying script attributes and
   OS-specific variables, with each heading (`#`) defining an action and
   bullet points (`*`) specifying commands
   (e.g., [example.md](https://raw.githubusercontent.com/sireum/roboto-example/refs/heads/master/example.md)).

## Running a Script

```
sireum roboto run <option>* <path> <arg>*
```

where `<path>` is either a `.md` (Markdown) or `.cmd` (Slang script) file.

### With Text-to-Speech

By default, Roboto uses MaryTTS for speech synthesis (local, no API key needed).

```
sireum roboto run example.md
```

#### Using Azure

Define the `AZURE_KEY` environment variable using one of your Azure account
text-to-speech service keys.

```
sireum roboto run --service azure example.md
```

#### Using AWS

Install [AWS CLI](https://aws.amazon.com/cli) and configure it (`aws configure`).

```
sireum roboto run --service aws example.md
```

### Screen Recording

Record the script execution as an MP4 video with synchronized audio
(system audio + TTS speech). Requires FFmpeg.

```
sireum roboto run --record output.mp4 example.md
```

The recording captures video with synchronized TTS and system audio.

### Capturing a Screen for Template Images

```
sireum roboto capture <output.png>
```

This captures the full screen after a 5-second countdown.
The captured image can be cropped (e.g., in Preview on macOS) to create
template images for `clickImage` and `waitForImage` commands.

## Roboto Markdown Syntax

### YAML Frontmatter

```yaml
---
name: "Script Name"
defaultCharDelayMs: 50
defaultActionDelayMs: 2000
vars:
  - key: value
varsMac:
  - key: macValue
varsWin:
  - key: winValue
varsLinux:
  - key: linuxValue
varsLinuxArm:
  - key: linuxArmValue
subst:
  - hamr: Hammer
  - sysml: Sis M L
---
```

| Key | Description | Default |
|---|---|---|
| `name` | Script name displayed at startup | `"Roboto"` |
| `defaultCharDelayMs` | Default delay (ms) between characters for `typeText` | `50` |
| `defaultActionDelayMs` | Default delay (ms) between actions | `2000` |
| `vars` | Base variables (all platforms) | |
| `varsMac` | macOS variable overrides | |
| `varsWin` | Windows variable overrides | |
| `varsLinux` | Linux (x86_64) variable overrides | |
| `varsLinuxArm` | Linux (AArch64) variable overrides | |
| `subst` | TTS pronunciation substitutions | |

### Variable Substitution

Variables defined in the frontmatter are substituted in command text using
`$name$` syntax (dollar-sign delimited).

OS-specific variable sections (`varsMac`, `varsWin`, etc.) override base `vars`
for the current platform.

Environment variables in variable values are expanded using `$ENV_VAR` syntax
(single `$` prefix). Use `$$` to escape a literal `$` (e.g., `$$SIREUM_HOME`
produces `$SIREUM_HOME` without expansion).

### TTS Pronunciation Substitution (`subst`)

The `subst` section defines pronunciation overrides for text-to-speech.
When `$term$` syntax is used in `speak` command text, the term is replaced
with its `subst` value before being sent to the TTS engine. This is useful for
terms that TTS engines mispronounce (e.g., CamelCase identifiers, abbreviations,
domain-specific terms).

```yaml
subst:
  - hamr: Hammer
  - sysml: Sis M L
  - isolette: Eye-so-let
```

```markdown
* speak: Let me demonstrate $hamr$ code generation for the $isolette$ model.
```

The TTS engine receives: "Let me demonstrate Hammer code generation for the Eye-so-let model."

### Actions

Each Markdown heading (`#`) defines an action. The heading text is the action
name. Bullet points (`*`) under a heading are the commands for that action.

HTML comments (`<!-- -->`) are ignored and can be used to comment out
sections.

### Commands

Commands use the syntax: `command[(options)]: arguments`

| Command | Syntax | Description |
|---|---|---|
| `typeText` | `typeText[(delay)]: text` | Type text; `delay=0` pastes from clipboard (instant), `delay=-1` or omitted uses `defaultCharDelayMs` |
| `pressKey` | `pressKey[(mod,...)]: key` | Press a key with optional modifiers |
| `typeChar` | `typeChar[(mod,...)]: c` | Type a single character with optional modifiers |
| `wait` | `wait: ms` | Wait for the specified milliseconds |
| `notify` | `notify: message` | Show an on-screen overlay notification |
| `speak` | `speak[(async)]: text` | Speak text using TTS; `async` option for non-blocking playback |
| `waitForSpeech` | `waitForSpeech` | Wait for async speech to finish |
| `mouseMove` | `mouseMove: x, y` | Move mouse to coordinates |
| `mouseClick` | `mouseClick[(button)]: x, y` | Click at coordinates (`Left`/`Middle`/`Right`, default `Left`) |
| `mouseDoubleClick` | `mouseDoubleClick[(button)]: x, y` | Double-click at coordinates |
| `mouseDrag` | `mouseDrag: fromX, fromY, toX, toY` | Drag from one point to another |
| `clickImage` | `clickImage[(similarity, xOff, yOff)]: image.png` | Find image on screen and click it (path relative to `.md` file) |
| `waitForImage` | `waitForImage[(similarity, timeoutMs)]: image.png` | Wait for image to appear on screen |
| `clickText` | `clickText[(timeoutMs, xOff, yOff)]: text` | Find text on screen using OCR and click it |
| `waitForText` | `waitForText[(timeoutMs)]: text` | Wait for text to appear on screen using OCR |
| `screenCapture` | `screenCapture: output.png` | Capture the screen to a file |

### Keys

`Enter`, `Escape` (or `Esc`), `Tab`, `Space`, `Backspace`, `Delete`,
`Up`, `Down`, `Left`, `Right`, `Home`, `End`, `PageUp`, `PageDown`,
`F1`–`F12`

### Modifiers

`Ctrl`, `Shift`, `Alt`, `Meta` (or `Cmd`)

### Template Matching (`clickImage` / `waitForImage`)

Image paths are resolved relative to the `.md` file's directory.
The `similarity` parameter (0.0–1.0, default 0.9) controls how closely the
screen region must match the template image.
`xOff` and `yOff` (default 0) offset the click position from the center of
the matched region.

Template images with transparent backgrounds (PNG alpha channel) are supported.
Transparent pixels (alpha < 128) are ignored during matching, so only the
foreground content needs to match. This allows the same template to work across
different themes or background colors.

Workflow:

1. Run `sireum roboto capture screenshot.png`
2. Crop the target UI element from the screenshot (e.g., in Preview on macOS)
3. Optionally, make the background transparent for theme-independent matching
4. Save the cropped image next to the `.md` file
5. Reference it: `clickImage(0.9): my-button.png`
