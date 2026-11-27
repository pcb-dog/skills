---
name: order-pcb
description: Quote and order printed circuit boards from pcb.dog. Exports Gerbers from a KiCad project (or takes a Gerber .zip), pre-checks them for manufacturing problems, compares USD prices and door-to-door delivery dates, and places the order once the person approves the summary. Use when someone wants PCBs made, a PCB price or lead time, or a manufacturability check of their Gerber files.
license: MIT
---

# Order PCBs from pcb.dog

pcb.dog makes 2, 4 and 6 layer FR-4 boards (10 × 10 to 400 × 500 mm, HASL or ENIG) and ships them door to door.
Prices are in USD and final; US and EU shipments are DDP, so duties are prepaid. No API key, account or wallet.

Use the `pcb-dog` MCP tools when they are connected. Otherwise call the HTTP API with curl; every step below names
both. Full reference: https://pcb.dog/llms.txt and https://pcb.dog/openapi.json

## 1. Get one Gerber .zip

- The person already has a Gerber .zip: use it.
- A KiCad project (KiCad 8 or later ships `kicad-cli`; on Windows it is in `C:\Program Files\KiCad\<version>\bin\`):

  ```sh
  kicad-cli pcb export gerbers --board-plot-params -o gerbers/ board.kicad_pcb
  kicad-cli pcb export drill -o gerbers/ board.kicad_pcb
  ```

  Then zip the files in `gerbers/` into one .zip.
- Any other tool: export Gerber RS-274X plus Excellon drill files and zip them. https://pcb.dog/en/gerber-guide

## 2. Pre-check the file

- With a shell: `curl -F file=@board.zip https://pcb.dog/api/analyze`. Keep the reply's `id`; it is the `file_id` for
  every later step and stays valid for 24 hours after its last use.
- MCP without a shell: `analyze_gerbers` with `file_url` (a public, direct link) or, for small files only,
  `file_base64`.

Tell the person the detected layer count and board size, and pass on any warnings or findings as they are.
If the layers or size are not what they expect, stop and sort that out before quoting.

## 3. Quote

`get_pcb_quote`, or `GET https://pcb.dog/api/quote?layers=4&width_mm=100&height_mm=100&qty=5`, with the layers and
size from the pre-check. Show each tier with its `usd` price and `delivery_by` date. Optional settings:
`thickness_mm`, `finish`, `mask`. If price matters, `find_cost_reductions` lists cheaper nearby choices.

## 4. Create the order

Ask the person for anything you do not have; never invent it:

- tier and quantity
- shipping address: `name`, `line1`, `city`, `postal_code`, `country` (two-letter code), plus `state` where it applies
- their e-mail address

Then `create_order`, or `POST https://pcb.dog/api/orders` with JSON
`{"file_id": "…", "tier": "economy", "qty": 5, "ship_to": {…}, "email": "…", "idempotency_key": "…"}`.
Nothing is charged. The reply is a summary with `order_id`, the USD total, the delivery date, a link to a preview of
the board and any warnings.

## 5. Get approval, then confirm

Show the person the summary and the preview link, and wait for a clear yes in this conversation.
Then `confirm_order`, or `POST https://pcb.dog/api/orders/{order_id}/confirm`.

Relay the reply as it is:

- An accepted order carries a payment link. Give it to the person: they pay, not you.
- If pcb.dog cannot take the order, the reply says why and carries a first-order coupon. Pass both on.

Confirming the same `order_id` again returns the same answer and never places a second order.

## Rules

- Never confirm an order the person has not approved.
- Never ask for, type or store card details; payment happens only through the link.
- Quote prices and dates exactly as the API returns them.
