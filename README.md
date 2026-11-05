# SDAINTAI Platform — v0.0.1

Standalone scoped repo for the SDAINTAI (PI/PO + S/4HANA) side of the
platform, split out of the original monorepo the same way SDADATAAI
was split into `ZR_SDADATAAI_V0` — same pattern, no shared MCP/env
vars/Docker config between the two products.

## Project Layout

File or Folder | Purpose
---------|----------
`app/aiconx/webapp/btp_cockpit.html` | The PI/PO + MuleSoft cockpit UI
`app/aiconx/webapp/btp_app.js` / `btp_style.css` / `btp_lumi.css` / `style.css` | Its local JS/CSS deps
`mcp/pipo-rfc-server/` | Standalone MCP server exposing SAP PI/PO & S/4HANA RFC calls — SDAINTAI-only, no SDADATAAI references
`mta-sdaintai.yaml` | Deploys the static console to `sdaintai-v0.cfapps.us10.hana.ondemand.com` (a new, separate route — does **not** touch the existing `sdaintai.cfapps.us10.hana.ondemand.com` live route)
`README.md` | This guide

**Not yet deployed from this repo** — `mta-sdaintai.yaml` targets a new
route (`sdaintai-v0...`) specifically so a future `cf deploy` from here
can never collide with or overwrite the live `sdaintai.cfapps...` app
that still deploys from the original SDAINTAI monorepo.

## Developer workflow: one branch + one local Docker preview per developer

Each developer works on their own branch and previews their own
changes in their own local Docker container — nobody's build or `cf`
deploy touches anyone else's work or the shared production route.

1. Clone the repo and create your own branch:
   ```bash
   git clone https://github.com/SDAKB1/ZR_SDAINTAI_V0.git
   cd ZR_SDAINTAI_V0
   git checkout -b <yourname>/<short-feature-name>
   ```
2. Make your changes under `app/aiconx/webapp/btp_cockpit.html`.
3. Preview locally with Docker — no CF route, no quota, fully isolated:
   ```bash
   docker compose -f docker-compose.dev.yml up --build dev1
   ```
   Open `http://localhost:8071/btp_cockpit.html`.

   `docker-compose.dev.yml` has one service block per developer
   (`dev1` on port 8071, `dev2` on port 8072, ...). If you're the next
   developer, copy the commented `dev3` block, give it your own
   name/port, and use that instead of `dev1`/`dev2`.
4. Commit and push your branch, then open a PR into `main` when ready:
   ```bash
   git push -u origin <yourname>/<short-feature-name>
   ```
5. Only merge to `main` — and only run a real `cf deploy` — when the
   change is actually meant to ship. Nothing has been deployed from
   this repo yet.

## PI/PO RFC MCP Server

See `mcp/pipo-rfc-server/README.md` for the full quick-start (connecting
as a remote MCP server, running it locally, or deploying/redeploying it
to Cloud Foundry). It can temporarily point at the same demo SAP system
used for SDADATAAI's BW connection, purely to prove connectivity end to
end, then be swapped to real PI/PO host/credentials later — no code
change needed, just different `cf set-env` values.
