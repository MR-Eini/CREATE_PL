# SWAT+ COCOA Build Workflow (Runbook)

This repository contains an R workflow to build a SWAT+ model using a COCOA-style setup. The pipeline creates/updates a SWATbuildR SQLite project, writes SWAT+ TxtInOut, applies required patches (landuse, nutrients, time), generates SWATfarmR management, and exports a portable `clean_setup/`.

---

## Repository layout (expected)

