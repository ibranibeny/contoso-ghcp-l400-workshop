# GitHub Copilot L400 Workshop — Contoso

Agenda site for the GitHub Copilot Level 400 hands-on workshop at the Contoso Internal Developer Gathering.

- **Time:** 13:00–17:00 WIB, with a 20-minute coffee break (15:00–15:20)
- **Delivered by:** Microsoft Indonesia
- **Runs on:** participants' own laptops; the lab application stack runs locally with Docker Compose

> **Status: DRAFT v0.6 — published for organizer review. Finish the checklist below before sharing the link with participants.**

## What's in this repository

| File | Purpose |
| --- | --- |
| `index.html` | The complete agenda site, including the Docker guide (WSL 2 and Docker Engine on Windows, then build and deploy) and the demo story. One self-contained HTML file: inline CSS and SVG, with the demo replay embedded (it plays inside the page, so it also works in previews and as a single downloaded file); no external fonts, images or scripts. |
| `simulation.html` | The same replay on its own page (full window), linked from the embedded player. Replay of the demo in a VS Code mock-up: Copilot with the superpowers skills and the `contoso-fullstack-docker` skill builds JagaJalur and runs it on Docker. Self-contained (inline script, no external files); play, pause, step, speed and jump to any act. |
| `DEMO-SCRIPT.md` | Facilitator run-of-show: prompts, expected superpowers behaviour, suggested answers, timings, git tags per act and fallbacks. |
| `README.md` | This file. |

The demo application itself lives in its own repository, `jagajalur-demo` (delivered as `jagajalur-demo.zip`, with git history and one tag per act). Push it as a separate repository; it is not part of this site.

## Publish with GitHub Pages

1. Create a repository (for example `contoso-ghcp-l400-workshop`) in the account or organization that will host the site.
2. Commit `index.html`, `simulation.html`, `README.md` and `DEMO-SCRIPT.md` to the root of the default branch.
3. Open **Settings → Pages**. Under **Build and deployment**, set **Source** to **Deploy from a branch**, choose `main` and `/ (root)`, then select **Save**.
4. When the deployment finishes, the site is live at `https://<owner>.github.io/<repository>/`.

For an internal-only audience, use an organization on GitHub Enterprise Cloud and publish the site privately so only people with repository access can view it.

## Before publishing

- [ ] Remove the draft ribbon: delete the `<div class="draft-ribbon" …>` line in `index.html`.
- [ ] Replace placeholders marked `[TBC]` and `<lab-repository-url>` (lab repository link, demo repository link, co-facilitators).
- [ ] Push `jagajalur-demo` with its tags (`git push origin main --tags`) and link it from the demo section.
- [ ] Confirm the agenda times with the Contoso organizers.
- [ ] Confirm with Contoso IT that participants can install WSL 2 (virtualization on, WSL allowed by device policy, administrator rights or an IT-run install) and reach the Ubuntu archive, download.docker.com, Docker Hub, PyPI and npm from inside WSL. The labs use Docker Engine inside WSL 2, which is open source, so no Docker Desktop licences are needed.
- [ ] Test the full setup on a Contoso Windows laptop on the Contoso network: WSL 2 and Docker Engine (Docker guide steps 1a to 1e), then `docker compose up -d --build` and the smoke test.
- [ ] Re-check feature names against the GitHub Copilot changelog in the week before delivery.

## Content notes

- All lab material uses synthetic data only.
- External links point to public GitHub Docs, the GitHub changelog, Microsoft Learn, VS Code Docs, Docker Docs and the superpowers repository.
- superpowers is a third-party open-source project (MIT) by Jesse Vincent. It is installed on laptops as a plugin, not copied into these repositories; agree its use with Contoso IT like any other developer tool.
