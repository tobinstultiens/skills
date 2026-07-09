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

## Cloning into an existing `.claude` (or `.copilot`) directory

The preferred setup is to have this repository live directly inside the Claude Code
config directory (e.g. `~/.claude`). Because that directory is not empty, a plain
`git clone` will fail, so initialize in place and pull instead:

```sh
cd ~/.claude
git init -b master
git remote add origin git@github.com:tobinstultiens/skills.git
git fetch origin master
git pull origin master
```

Only the tracked paths listed above are managed by git; everything else in the
directory (credentials, session state, history, caches) stays ignored via
`.gitignore`. Do not run `git add -A` here — stage skill and doc changes explicitly
so local state and credentials are never committed.

## Restoring skills

To restore on another machine, clone this repository into a temporary location, then copy or sync the `skills\` folder into that machine's `.copilot` folder. Avoid cloning directly over an existing `.copilot` folder because it may already contain local settings, session state, and databases.
