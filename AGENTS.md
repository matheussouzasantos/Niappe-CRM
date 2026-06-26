<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Knowledge base — consult before acting

Curated, durable context lives in [`.claude/context/`](.claude/context/README.md).
It is **not** auto-loaded — open the relevant file **on demand** when a task
matches a trigger below. (Don't `@import` these here; that would load them into
every context and waste tokens.)

| When you are...                                                          | Read |
| ----------------------------------------------------------------------- | ---- |
| Writing, refactoring, or reviewing **any** code                         | [`.claude/context/engineering/codigo-limpo.md`](.claude/context/engineering/codigo-limpo.md) |
| Designing modules, boundaries, dependencies, or folder structure        | [`.claude/context/engineering/arquitetura-limpa.md`](.claude/context/engineering/arquitetura-limpa.md) |
| Touching UI, user-facing naming, colors, copy, or visual identity       | [`.claude/context/brand/README.md`](.claude/context/brand/README.md) |
| Needing product / domain context or business rules                      | [`.claude/context/product/README.md`](.claude/context/product/README.md) |

Index and conventions for adding new context: [`.claude/context/README.md`](.claude/context/README.md).
