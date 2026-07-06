---
name: new-artefact
description: Use when the user says a new artefact/article/deck/report has been added, dumped, or downloaded into this UsefulArtefacts repo's artefacts/ folder and needs onboarding onto the site (e.g. "there's a new artefact", "I added a new artefact", "process the new artefact", "add this to the site"). Fixes claude.ai export quirks, links it from the main gallery page, and pushes.
---

# Onboard a new artefact

This repo (`paulshilling/UsefulArtefacts`, live at usefulartefacts.com) follows a recurring pattern: the user drops a raw artefact export into `artefacts/<slug>/`, and it needs fixing up (it usually won't work standalone as saved) and linking from the root `index.html` gallery.

## Steps

1. Find the new artefact folder(s): list `artefacts/*/` and cross-check against which ones already have a matching `<a class="card" href="artefacts/<slug>/...">` in the root `index.html`. Any folder not yet linked is new. If the user named the folder/artefact explicitly, use that instead of searching.

2. For each new artefact folder, delegate the fix-and-publish work to the `artefact-publisher` subagent (Agent tool, `subagent_type: artefact-publisher`), passing it the folder path. That agent knows the claude.ai frame-shell export quirk (dead wrapper `.html` + real content in `*_files/saved_resource.html`) and handles extraction, link scrubbing, adding the gallery card, committing, pushing, and triggering a Pages rebuild.

3. If multiple new artefacts are found, handle them one at a time (sequentially) rather than in parallel — they all edit the shared root `index.html`, and parallel edits would conflict.

4. After the agent(s) finish, don't just relay their summary — spot check with `git log -1 --stat` (or per-commit if several ran) that the diff actually matches what was claimed, and that the root `index.html` still looks correct.

5. Report the new artefact's live URL(s) back to the user: `https://usefulartefacts.com/artefacts/<slug>/index.html`.

If you're doing the fix inline yourself instead of delegating (e.g. a trivial one-off tweak), the full claude.ai export pattern and fix procedure is documented in `.claude/agents/artefact-publisher.md` — read it rather than re-deriving from scratch.
