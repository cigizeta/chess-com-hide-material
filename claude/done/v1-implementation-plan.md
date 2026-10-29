# v1 implementation plan

## Decisions

- **Browser:** Chrome only, Manifest V3.
- **Mechanism:** a single content stylesheet injected via `content_scripts[].css`. No JavaScript,
  no popup, no toggle: disabling the extension is the off switch. CSS is applied before first
  paint (no flash of captured pieces) and fails safe (a selector that stops matching hides
  nothing and breaks nothing).
- **What is hidden:** the captured pieces and the "+N" material advantage score of both players.
  The evaluation bar and engine lines are out of scope.
- **Where:** content script matches `https://www.chess.com/*`; the selector decides where it
  applies.
- **Selector:** the whole `<wc-captured-pieces>` custom element, hidden with
  `visibility: hidden` so the player box keeps its layout. Observed on a daily game page:

  ```html
  <wc-captured-pieces board-id="board-single" class="player-pieces" ...>
    <div>
      <span class="captured-pieces-cpiece captured-pieces-w-4-pawns"></span>
      ...
      <span class="captured-pieces-cpiece captured-pieces-score">+1</span>
    </div>
  </wc-captured-pieces>
  ```

  The custom element tag name is a deliberate, semantic hook, more stable than the sprite
  class names inside it.

## Files

- `manifest.json`: MV3, name, version `0.1.0`, description, one `content_scripts` entry
  (`matches: ["https://www.chess.com/*"]`, `css: ["src/hide-material.css"]`,
  `run_at: "document_start"`). No permissions, no host permissions beyond the match.
- `src/hide-material.css`: the only place selectors live; each rule carries a comment naming
  the page and element it matches.
- `.gitattributes`: `* text=auto eol=native` style rules as described in CLAUDE.md, with `*.sh`
  forced to LF and `*.png` binary.
- `README.md`: what it does, how to load it unpacked, AGPL-3.0-only notice.
- `claude/TODO.md`, `claude/done/`, `claude/archived/`: repository organization skeleton.

## Manual test

Load unpacked via `chrome://extensions` (Developer mode, "Load unpacked", repository root).
On each page below, captured pieces and the score must be invisible, the player box must not
shift, and the rest of the page must work normally:

1. Daily game.
2. Live game (blitz or rapid).
3. Game against a bot.
4. Analysis / game review of a finished game.

Any page where `<wc-captured-pieces>` is absent or different gets inspected, and its selector
added to `src/hide-material.css` with a comment.

Finally, disable the extension and confirm the material display comes back after a reload.

**Result:** daily and analysis pages pass. Live and bot games were not tested, since the
developer does not play them.
