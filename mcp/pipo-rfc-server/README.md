# SDAINTAI PI/PO RFC MCP Server — Quick Start

How to actually **call** the deployed MCP server (or run it locally) —
not how it was built.

## What this is, in one line

A standalone MCP server that lets Claude (or any MCP client, or
SDAINTAI's own CAP backend) call SAP RFCs/BAPIs by name on the
**SDAINTAI** (PI/PO) system. No other system name is recognized —
this server is scoped to SDAINTAI only, so it can live in its own
repo/deployment with no connection to SDADATAAI's BW bridge.

## Option A — Connect to it as a remote MCP server (Claude Desktop / Claude Code)

Once deployed, the server is reachable at a CF route ending in `/mcp`
(confirm the exact URL with `cf app sdaintai-pipo-rfc-mcp` — defaults to
`https://sdaintai-pipo-rfc-mcp.cfapps.us10.hana.ondemand.com/mcp`, but CF
may have suffixed it if that route was already taken).

Every request needs **HTTP Basic Auth**. Ask whoever deployed the server
for the current username/password (don't assume the placeholder
defaults in `server.js` are still in use — they're meant to be rotated).

```json
{
  "mcpServers": {
    "sdaintai-pipo-rfc": {
      "url": "https://<your-route>.cfapps.us10.hana.ondemand.com/mcp",
      "headers": {
        "Authorization": "Basic <base64 of username:password>"
      }
    }
  }
}
```

## Option B — Run it locally

```bash
cd mcp/pipo-rfc-server
cp .env.example .env   # fill in SDAINTAI_SAP_* values
npm install
npm start
```

## Connecting to a real PI/PO system

Set these as CF env vars (never commit real values):

```bash
cf set-env sdaintai-pipo-rfc-mcp SDAINTAI_SAP_ASHOST '<PI/PO host>'
cf set-env sdaintai-pipo-rfc-mcp SDAINTAI_SAP_SYSNR '<sysnr>'
cf set-env sdaintai-pipo-rfc-mcp SDAINTAI_SAP_CLIENT '<client>'
cf set-env sdaintai-pipo-rfc-mcp SDAINTAI_SAP_USER '<user>'
cf set-env sdaintai-pipo-rfc-mcp SDAINTAI_SAP_PASSWD '<password>'
cf restage sdaintai-pipo-rfc-mcp
```

**Interim connectivity option:** until real PI/PO credentials are
available, these five values can temporarily point at the same demo SAP
system used for SDADATAAI's BW connection, purely to prove the MCP
bridge works end-to-end. Swap to the real PI/PO host/credentials later —
same five commands, different values, no code change.

## Known dependency caveat

Uses `open-rfc` (public npm, no SAP NW RFC SDK, no private registry
token) instead of `@sap-rfc/node-rfc-library`. SDADATAAI's `bw-rfc-server`
had to make this same switch after its private-registry token went dead.
`open-rfc` is 0.x/beta — verify real RFC calls work before relying on it
beyond initial wiring.
