# Write Review Playground

A single-file HTML tool for reviewing line edits on blog posts and prose. Paste your original and edited text, get a word-level diff, then accept or reject each change individually.

## What it does

1. **Input** - Paste original text on the left, edited version on the right
2. **Compare** - Computes a word-level diff using an LCS (Longest Common Subsequence) algorithm
3. **Review** - Each discrete change appears as a card with surrounding context, showing deleted text (red strikethrough) and inserted text (green)
4. **Decide** - Accept or reject each edit. The final text panel updates live
5. **Copy** - Grab the final merged text with one click

## Layout

After comparing, the screen splits into two panels:

- **Left panel** - Live final text preview. Highlights the active edit as you navigate
- **Right panel** - Edit cards with Accept/Reject buttons

A toolbar above both panels shows stats (pending/accepted/rejected counts), a progress bar, and bulk action buttons.

## Keyboard shortcuts

| Key | Action |
|-----|--------|
| `j` / `Down` | Next edit |
| `k` / `Up` | Previous edit |
| `a` | Accept focused edit |
| `r` | Reject focused edit |
| `u` | Undo (reset to pending) |

## How it works

### Tokenization
Text is split into word tokens with paragraph breaks preserved as special `\n\n` tokens. This maintains paragraph structure through the diff process.

### Diff algorithm
Uses standard LCS-based dynamic programming (`O(m*n)` where m and n are word counts). Produces a sequence of `keep`, `delete`, and `insert` operations.

### Edit grouping
Consecutive non-`keep` operations are grouped into discrete edits. Each edit captures:
- Up to 5 context words before and after
- The deleted words (from original)
- The inserted words (from edited version)

### Final text reconstruction
Iterates through the full diff operations. For each edit:
- **Accepted** - Uses the inserted words, drops the deleted
- **Rejected/Pending** - Keeps the original deleted words, drops the inserted

## Usage

Open `write-review-playground.html` directly in any browser. No build step, no dependencies, no server required.

Click **Load Sample** to see it in action with example text demonstrating typical prose edits: removing filler words, tightening prose, strengthening verbs, and cutting unnecessary qualifiers.

## Technical details

- **Zero dependencies** - Single HTML file with inline CSS and JS
- **Dark theme** - System font UI, easy on the eyes
- **Responsive** - Two-panel layout on desktop, stacks vertically on narrow screens (< 860px)
- **Performance** - LCS diff handles blog-post-length texts (up to ~2000 words) comfortably
