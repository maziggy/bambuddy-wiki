---
title: Community Add-ons
description: Third-party hardware and software add-ons built by the Bambuddy community
---

# Community Add-ons

Third-party projects that extend Bambuddy &mdash; built by the community, for the community. These are independent projects maintained outside the main Bambuddy repository, talking to Bambuddy through its public API.

!!! info "Want to build one?"
    Bambuddy exposes a documented [REST API](../reference/api.md) and supports [API keys](../features/api-keys.md) for headless integrations. If you build something cool, open a PR on the [wiki repo](https://github.com/maziggy/bambuddy-wiki) to add it here.

!!! warning "Independent projects"
    Community add-ons are maintained by their respective authors, not by the Bambuddy team. Issues and feature requests for an add-on belong on its own repository.

---

## :material-gesture-tap-button: Hardware

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-radiobox-marked: Bambutton](bambutton.md)
ESP32-C3 wireless button that lets you mark the build plate clear at the printer with a single press, so the next queued job can be dispatched automatically. Includes an LED ring that signals when a plate needs clearing.

**Author:** [Edward Chamberlain](https://github.com/EdwardChamberlain) &middot; [Repository](https://github.com/EdwardChamberlain/bambutton)
</div>

</div>

---

## :material-robot-outline: AI &amp; Integrations

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-cog-outline: Bambuddy MCP Server](bambuddy-mcp.md)
Model Context Protocol server that exposes Bambuddy's full REST API as tools for AI assistants like Claude Desktop and Claude Code. Talk to your printers in plain language &mdash; check status, queue prints, control lights, view camera snapshots.

**Author:** [MrMebelMan](https://github.com/MrMebelMan) &middot; [Repository](https://github.com/MrMebelMan/bambuddy-mcp)
</div>

</div>

---

## Contributing an add-on

If you've built something that integrates with Bambuddy &mdash; hardware, browser extensions, mobile shortcuts, scripts, dashboards, anything &mdash; we'd love to list it here.

To submit:

1. Make sure the project has a clear README and a license.
2. Open a PR against the [wiki repository](https://github.com/maziggy/bambuddy-wiki) adding a page under `docs/community/` and a card on this hub page.
3. Keep it interoperability-focused &mdash; add-ons should use Bambuddy's public API, not internal endpoints.

Questions? Drop into the [Discord](https://discord.gg/aFS3ZfScHM) or [forum](https://forum.bambuddy.cool).
