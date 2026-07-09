# Copilot skills backup

This repository backs up durable Copilot skill assets from this machine. It intentionally tracks only:

- `skills\**`
- `docs\adr\**`
- `.gitattributes`
- `.gitignore`
- `README.md`

Everything else in `.copilot` is treated as local state and stays ignored, including settings, session state, logs, caches, databases, command history, IDE metadata, and other generated files.

## Updating the backup

After creating or changing a skill:

1. Review the changed files with `git status --short`.
2. Stage the intended skill and documentation changes.
3. Check the staged diff before committing.
4. Push with `git push origin master`.

`origin` is configured with push URLs for both the primary and backup repositories, so pushing to `origin` publishes the same commit to both destinations. The `backup` remote remains available as a named remote for inspection.

## Restoring skills

To restore on another machine, clone this repository into a temporary location, then copy or sync the `skills\` folder into that machine's `.copilot` folder. Avoid cloning directly over an existing `.copilot` folder because it may already contain local settings, session state, and databases.
