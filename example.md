---
name: "Roboto Example"
defaultCharDelayMs: 30
defaultActionDelayMs: 2000
vars:
  - editor: TextEdit
  - greeting: Hello from Roboto!
  - osMod: Ctrl
  - modLabel: Ctrl
varsMac:
  - editor: TextEdit
  - osMod: Meta
  - modLabel: Cmd
varsLinux:
  - editor: gedit
varsWin:
  - editor: notepad
---

# Open Editor

* notify: Opening $editor$...
* typeText(0): open -a $editor$
* pressKey: Enter
* wait: 3000

# Type Greeting

* notify: Typing greeting...
* speak: Let me type a greeting message.
* typeText: $greeting$
* wait: 1000

# Select All and Copy

* notify: Selecting all text ($modLabel$+A)...
* typeChar($osMod$): a
* wait: 500
* notify: Copying to clipboard ($modLabel$+C)...
* typeChar($osMod$): c
* wait: 500

# Done

* speak: The example script has completed.
* notify: Done!
* wait: 3000
