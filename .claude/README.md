# Claude Code automation for this repo

This repo has a Claude Code skill and agent set up to handle the repeated task of
onboarding a new artefact onto the site.

## The problem this solves

Artefacts are usually downloaded from Claude.ai via the browser's "Save As," which
produces a folder like:

```
artefacts/<slug>/
  <Title>.html                          <- NOT the real content
  <Title>_files/
    saved_resource.html                 <- the real content
    frame-shell-*.js
```

`<Title>.html` is just a frame-shell wrapper that loads the actual artifact live
from claude.ai using a session-specific ID. Opened outside an authenticated
claude.ai session, it 404s ("not found"). The real, self-contained content is
sitting in the sibling `saved_resource.html`.

## `new-artefact` skill

`.claude/skills/new-artefact/SKILL.md`

Triggers automatically when you say things like "there's a new artefact," "I
added an artefact," or "process the new artefact." It finds artefact folders
under `artefacts/` that aren't yet linked from the root `index.html`, then hands
each one to the `artefact-publisher` agent.

## `artefact-publisher` agent

`.claude/agents/artefact-publisher.md`

Does the actual work for one artefact folder:

1. Detects whether the claude.ai frame-shell pattern applies.
2. Promotes `saved_resource.html` to `index.html`, deletes the dead wrapper file
   and the now-empty `_files` folder.
3. Scrubs any remaining claude.ai references — in particular, in-page anchor
   links that got saved as absolute `https://<uuid>.frame.claudeusercontent.com/.../#fragment`
   URLs get relativized to `#fragment` so in-page navigation still works.
4. Verifies the page serves real content locally before wiring it up.
5. Adds a matching `<a class="card">` entry to the root `index.html` gallery.
6. Commits, pushes, and triggers a fresh GitHub Pages build (`gh api -X POST
   repos/paulshilling/UsefulArtefacts/pages/builds`) since GitHub's CDN can keep
   serving a stale cached page for a while after a plain push.

## Usage

Just say **"there's a new artefact"** (or similar) after dropping a new folder
into `artefacts/`. No need to specify the path unless there are multiple new
ones at once.
