---
name: bundle-output
description: Format multi-file technical documents (implementation specs, configuration docs, multi-section technical writeups) as one splittable plain-text bundle wrapped in a tilde fence, with FILE markers, a README manifest, fixed folders, and no renderable code fences.
---

# Bundle output format

Use this skill when producing an implementation spec, configuration document, or any multi-section technical document that should be split into separate files afterward.

The whole result is one plain-text block. A splitter can cut it into files by reading the FILE markers.

## Rules

1. **Outer wrapper.** Wrap the entire output in a tilde fence. Use at least four tildes, and use one more tilde than the longest run of tildes anywhere in the content.
   *Why tildes:* a backtick fence closes at the first line of three or more backticks, which can end the wrapper early. Tildes are only closed by tildes.

2. **File markers.** Put each file between these markers, with nothing outside them except the outer fence:
   ```
   === FILE: <relative/path/filename.md> ===
   ...file content...
   === END FILE ===
   ```
   *Why:* markers give the splitter an unambiguous boundary that content cannot easily imitate.

3. **Header line.** Start each file's content with a `# filename` header line.
   *Why:* the header lets a reader identify the file even after it has been split out.

4. **No code fences in the bundle.** Never write a triple-backtick line or any other code fence inside the bundle. Convert each code block to a four-space indented block, with `[code: <language>]` on the line above and `[end code]` on the line below.
   *Why:* indented blocks do not open or close a fence, so they cannot end the wrapper or trigger a renderer. The `[code:]` and `[end code]` lines let the block be found and restored.

5. **No nesting.** Never place one file's markers inside another file.
   *Why:* nested markers make the splitter's boundaries ambiguous.

6. **Long examples go in files.** Any example longer than about 20 lines goes in its own standalone file, with its own FILE markers. Reference it with a standard Markdown link, such as `[example](examples/sync_job.py)`.
   *Why:* long inline examples make the documents hard to read and make the indentation in rule 4 unwieldy.

7. **Diagrams.** Never use ASCII art. Never embed a Mermaid block in a document. Save each Mermaid diagram as its own `.mmd` file and each SVG as its own `.svg` file, both with FILE markers. Refer to them with a relative path such as `diagrams/architecture.mmd`.
   *Why:* embedded Mermaid blocks can be rendered before the wrapper is applied. Separate files keep diagrams inert inside the bundle.

8. **File structure.** Organize the bundle this way:
   - `README.md` at the root. It is the entry point and lists every file in the bundle in reading order, as Markdown links.
   - `docs/` for narrative sections.
   - `examples/` for standalone code files.
   - `diagrams/` for `.mmd` and `.svg` files.
   - Use lowercase, hyphen-separated filenames.
   - Use only relative paths in links.
   *Why:* one entry point, a complete manifest, and fixed folders keep paths resolvable and the layout the same on every run.

## Example

This example is illustrative. Do not copy its filenames or content. The outer wrapper here uses five tildes because the example itself does not contain tilde runs, but your output must follow rule 1.

~~~~~
=== FILE: README.md ===
# Sync Service Spec

Read in order:
1. [Overview](docs/overview.md)
2. [Sync job](docs/sync-job.md)

=== END FILE ===
=== FILE: docs/sync-job.md ===
# Sync job

The job pulls changes every 15 minutes. Full example: [sync job](examples/sync_job.py).

[code: yaml]
interval: 15m
retries: 3
[end code]

=== END FILE ===
=== FILE: examples/sync_job.py ===
def run(client):
    for item in client.pull_changes():
        client.apply(item)
=== END FILE ===
~~~~~

## Checklist before returning output

Verify each item. Fix any failure before returning.

- [ ] The outer fence has more tildes than any tilde run in the content, and at least four.
- [ ] Every `=== FILE` line has a matching `=== END FILE` line, and no file is nested in another.
- [ ] Every file's content starts with a `# filename` header.
- [ ] No triple-backtick or other code fence appears inside the bundle. Code blocks use `[code: ...]` and `[end code]`.
- [ ] No ASCII art appears, and no Mermaid block is embedded.
- [ ] Every link and path points to a file that exists in the bundle.
- [ ] `README.md` lists every other file in reading order.
- [ ] Filenames are lowercase and hyphen-separated, and folders are `docs/`, `examples/`, and `diagrams/` as applicable.
