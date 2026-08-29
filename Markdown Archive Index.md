# Markdown Archive Index

```md
# Archive

{{#each files}}
- [{{name}}](./archive/{{name}})
{{/each}}

---

{{#each files}}
## {{name}}

{{preview content}}
{{/each}}
```

- **Source:** `./archive/*.md`
- **Navigation:** Generated from discovered filenames.
- **Preview:** Generated from each file's Markdown content.
- **Behavior:** Adding or removing `.md` files automatically updates the index.