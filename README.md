# Backroom Inventory

Counts, adjustments, receiving and transfers for stores running **AEPOS (AdvEntPOS)**. Every scan is staged on
the PC and shown for review before anything is written to AEPOS. Part of **Backroom Suite**; independent software,
not made or supported by AdvEntPOS.

This repository only holds the installers. Most stores install it through the Backroom Suite hub
([BackroomSuite-Releases](https://github.com/Beeds666/BackroomSuite-Releases)), which also keeps it updated.

## Which download

Open **[Releases](../../releases)**:

| Release | Who it's for |
|---|---|
| **Latest** (not marked *Pre-release*) | Everyone. This is the one to install. |
| *Pre-release* | Test versions for PCs on the suite's test channel. Don't install these on a store PC unless you're testing. |

Download `AEPOSInventorySetup-x.y.z.exe`. Each release also has `AEPOSInventorySetup-x.y.z.exe.sha256`; the
in-app updater checks every download against it (you can too: `Get-FileHash .\AEPOSInventorySetup-x.y.z.exe`
in PowerShell must show the same value). The installers aren't code-signed yet, so Windows SmartScreen may
warn: **More info → Run anyway**.

## Requirements

- Windows 10 or 11, 64-bit. Nothing else to install (the app includes .NET).
- Network access from the PC to the store's AEPOS database server.
- Phone counting: phones on the same network as the PC, and Windows Firewall allowing the app on port **8787**
  (the Phone counting window checks this and has a **Fix firewall** button).

## Install

1. Run the Setup.exe (Windows asks for administrator permission). It installs to `C:\Program Files\Backroom Inventory`.
2. Open **Backroom Inventory** → **Store connections…** and add the store (or pick the one already set up in the
   Backroom Suite hub: every suite app shares the same list).
3. Pick the facility the store counts (Dutch Mill: 1 = In-Store, 2 = Store Room) and, optionally, the AEPOS user
   name to record on changes.
4. Optional: set a **manager PIN** (protects commit, undo and store connections).

Updates: the app checks for a newer version when it opens and offers **Update now**. Store connections, open
batches and history are kept.

## Database permissions (SQL Server)

| To | The app's login needs |
|---|---|
| Look up items, pull lists, reports | SELECT on the AEPOS item, stock, facility, batch and vendor tables (`ITM_*`, `STK_*`, `MST_*`, `VND_VENDOR`, `AUD_STOCKCHANGELOG`) |
| Commit and undo | INSERT/UPDATE on `STK_INVTBATCH`, `STK_INVTBATCHITEM`, `STK_INVTBATCHCOMMITLOG`, `STK_ITEMSTOCK` |
| Commit safety record | SELECT/INSERT/UPDATE on `AEI_COMMITOPS` (created on the first commit; if the login can't create tables, run `Sql\create-aei-commitops.sql` from the app folder as a database owner) |
| Count history for the planner | `AEI_COUNTLOG` (created automatically when allowed; counts still work without it) |
| Link an unknown barcode | INSERT on `ITM_ITEMUPC` |
| Backup check before commit | Reading `msdb` backup history and, to make a backup, BACKUP DATABASE (optional) |

## What works on each database

| | SQL Server (e.g. Dutch Mill) | PostgreSQL (Sypram cloud, e.g. Dale's) |
|---|---|---|
| Item look-up, search, pull lists, scanning, phone counting | Yes | Yes (assumes AEPOS's tables in lowercase; not yet tested on a real store) |
| Review with on-hand, cost and big-change check | Yes | No (on-hand can't be read yet) |
| **Commit** counts, adjustments, receiving | Yes | **No** |
| Transfers between facilities, undo, zero negatives | Yes | No |
| Low-stock, shrink report, count planner, item history, linking barcodes | Yes | No |
| Export a batch to CSV | Yes | Yes |

On PostgreSQL you can count and export, but nothing can be committed to AEPOS from this app yet.

## Before you commit

- **Back up the AEPOS database.** The app checks for a backup from the last 7 days and offers to make one.
- Commit shows every change (on hand now → new on hand, value at cost) and flags big changes. Nothing is written
  until you confirm.
- Large counts lock those items while they're written: sales of them wait. Best done after closing.

## If something goes wrong

- **"Nothing was changed in AEPOS"**: certain; the batch is open again with its scans. Fix the cause and commit again.
- **A batch shows "Committing"**: a commit was interrupted (crash, power, network). Don't re-enter it. Open it (or
  restart the app) with the database reachable: the app asks AEPOS whether the commit went in, then records it or
  reopens the batch. It never applies it twice.
- **"AEPOS WAS updated, but this PC couldn't save its record"**: AEPOS has the change; the app finishes the record
  automatically next time. Don't commit it again.
- **Undo** reverses a committed batch (same amounts in the other direction; sales since are kept).
- Data on the PC: `C:\ProgramData\AEPOSInventory` (`staging.db` holds open batches and history). Store connections:
  `%LocalAppData%\BackroomSuite\stores.json` (passwords encrypted for the Windows user).
- Support: in the Backroom Suite hub, **Support info** saves a file with versions, database checks and recent logs
  (no passwords) to send to whoever helps you.

## Release notes

Each release lists its changes, known issues and anything to do when upgrading.
