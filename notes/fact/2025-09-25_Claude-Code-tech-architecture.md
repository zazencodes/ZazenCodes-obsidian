---
date: 2025-09-25
tags:
    - fact
hubs:
    - "[[software-engineering]]"
urls:
    - https://newsletter.pragmaticengineer.com/p/how-claude-code-is-built
    - https://github.com/vadimdemedes/ink
    - https://www.yogalayout.dev/
    - https://bun.com/
---

# Claude Code tech architecture

Claude Code's tech stack:

- **TypeScript**: Claude Code is built on this language
- **React with Ink**: the UI is written in React, using the Ink framework for interactive command-line elements
- **Yoga**: the layout system, open sourced by Meta. It’s a constraints-based layout that works nicely. Terminal-based applications have the disadvantage of needing to support all sizes of terminals, so you need a layout system to do this pragmatically
- **Bun**: for building and packaging. The team chose it for speed compared to other build systems like Webpack, Vite, and others.


