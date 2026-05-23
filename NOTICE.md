# NOTICE — Third-party attribution

This project (claude-dev-cycle) includes derivative work and inspiration from
third-party MIT-licensed software. Attribution is given here in compliance with
their respective licenses.

---

## Telegraphic Mode (Caveman technique)

The Telegraphic Mode section in `plugins/dev-cycle/skills/dev-cycle/SKILL.md`
is a re-implementation of the technique originally published by:

- **Author**: Julius Brussee
- **Project**: caveman
- **Repository**: https://github.com/juliusbrussee/caveman
- **License**: MIT
- **Original purpose**: Claude Code skill that cuts ~65% of output tokens by
  forcing the agent to respond in telegraphic style (no filler words, no
  pleasantries, code intact).

The original `caveman` skill is a standalone plugin that users can install
independently (`/plugin marketplace add github:juliusbrussee/caveman`).

In claude-dev-cycle v2.0+, we re-implemented the technique as an internal
"Layer 2" of the cycle. The instruction block is integrated into the prompt
that Opus passes to Sonnet sub-agents during delegation (Step 4), so users get
the savings automatically when running `/dev-cycle` without needing to install
caveman separately.

The rules in our `Telegraphic Mode` section were written independently but
follow the same spirit and intent as Julius's original work. The block
includes attribution at the bottom ("Technique credit: Julius Brussee").

If you use this skill and find the Telegraphic Mode valuable, please also
⭐ Julius's repo at https://github.com/juliusbrussee/caveman.

### Original caveman copyright notice

```
MIT License

Copyright (c) 2026 Julius Brussee

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
IN THE SOFTWARE.
```

---

## /goal command (Claude Code built-in)

The `Self-evaluation loop` (Step 7) in our SKILL.md replicates the behavior
of Claude Code's native `/goal` command (built-in since v2.1.139) for users
who want the loop without installing or invoking `/goal` explicitly.

`/goal` is a feature of Claude Code itself, not a third-party plugin, so no
attribution is technically required. We acknowledge it here for completeness.

- Documentation: https://code.claude.com/docs/en/goal
- Released: May 2026 (Claude Code 2.1.139)
- Vendor: Anthropic

If you have Claude Code 2.1.139+, you can use native `/goal` alongside our
self-evaluation loop. They are complementary: native `/goal` uses an
independent Haiku evaluator with a fresh context, while our internal loop
uses the same Opus instance. For higher confidence on completion, combine
both.

---

## Acknowledgments

- **Anthropic** — for Claude Code, Claude Opus 4.7, Claude Sonnet 4.6, and
  the plugin marketplace infrastructure.
- **Julius Brussee** — for the original caveman technique that inspired our
  Telegraphic Mode layer.
- **the dot mack** — for `claude-mem`, which we recommend as a complementary
  tool for cross-session memory (https://github.com/thedotmack/claude-mem).

If we've missed an attribution, please open an issue and we'll add it.
