# Improving Copilot File Search Reliability

## Purpose

This repository documents a reproducible failure in OneDrive’s search indexing that prevents Copilot from discovering files that are fully synced and accessible in the user’s OneDrive account. The issue produces silent false negatives—Copilot reports that files do not exist even when they are present, cloud‑resident, and retrievable by direct resource ID.

The repo matters because search indexing gaps break the core Copilot workflow of finding and working with user documents. When Copilot cannot reliably locate files, users lose trust in the system and cannot complete basic tasks such as resume formatting, document retrieval, or file‑based automation.

The repository contains:
- A full technical report with reproduction steps, verification evidence, and environment details
- A structured analysis of the likely root cause
- Suggested engineering improvements
- A reproducible case study that can be used for debugging, regression testing, or internal escalation

## Summary of Findings

- OneDrive’s search index failed to ingest a cloud‑resident file that was fully synced, visible in the web interface, and retrievable via direct resource ID.  
- Copilot’s file‑search pipeline relies exclusively on this index and lacks a fallback path, causing the file to appear nonexistent despite being present.  
- Multiple search strategies—keyword queries, exact filename, name variations, and browse methods—all returned no results, confirming a consistent indexing failure rather than a query‑specific issue.  
- Folder structures were also not returned, indicating a second defect in folder‑level indexing that compounds the impact of file‑level misses.  
- This failure mode produces silent false negatives, which degrade user trust more severely than explicit errors and disrupt common workflows such as resume retrieval and document formatting.


**Reproducibility:** Confirmed on multiple attempts in a clean OneDrive environment.


## Why This Matters

This issue is not just a Copilot failure; it is a search‑indexing defect in OneDrive. Copilot relies entirely on the OneDrive search index, and when the index fails to ingest a file, Copilot has no fallback mechanism. This creates a single‑point‑of‑failure scenario where any indexing miss results in permanent invisibility.

Testing also suggests a second defect involving folder indexing, as no folder structures were returned. File indexing failures combined with folder indexing failures significantly increase the severity of the issue.

From a business perspective, resume files are among the most commonly searched documents. When Copilot cannot find them, all downstream workflows are blocked. This undermines Copilot’s value as a productivity assistant.

From a Human–AI interaction standpoint, silent false negatives erode user trust more quickly than visible errors. Users interpret the failure as AI incompetence, not an indexing gap. Copilot needs transparency when search results may be incomplete, and a fallback mechanism is essential for maintaining user trust.

## Potential Engineering Fixes

- **Provide clearer user feedback when indexing is incomplete:** Instead of returning “no results,” Copilot could surface a message explaining that OneDrive indexing may still be in progress. This would prevent users from assuming their files are missing and help maintain trust.
- **Add a fallback discovery method when search fails:** If the search index doesn’t return a file, Copilot could try a secondary method such as checking the folder directly. Even a simple fallback would reduce silent false negatives.
- **Offer a user‑initiated “refresh indexing” option:** Giving users a way to prompt OneDrive to re‑index a folder would create a recovery path when search results seem incorrect.
- **Improve visibility into indexing status:** A small indicator showing whether OneDrive indexing is complete would help users understand why search results might be incomplete.
- **Add telemetry to detect silent failures:** If a file is retrievable by ID but missing from search, the system could log that discrepancy. This would help engineering teams identify and fix indexing gaps earlier.

## Full Report

The complete technical case study is available here: [Full Technical Report](docs/full-report.md)

## How to Navigate This Repo

- README.md — High‑level overview of the issue, why it matters, and what this repository contains.
- docs/full-report.md — The complete technical case study, including reproduction steps, verification evidence, environment details, and root‑cause analysis.
  
## Recognition

A system notification acknowledged this investigation with the message “You helped Copilot,” reflecting that this work contributes to improving Copilot’s reliability and the user experience around AI‑assisted file search.

<p align="center"><img width="214" height="65" alt="Screenshot 2026-05-04 at 6 57 34 PM" src="https://github.com/user-attachments/assets/83044201-20b1-4da1-8ba6-4fb3bbce5247" />

## Example of the Issue in Action

This screenshot shows Copilot’s step‑by‑step troubleshooting during the investigation, including the moment it confirmed the file existed but was not discoverable through search.

<p align="center">
   <img width="405" height="410" alt="Screenshot 2026-05-04 at 6 52 19 PM" src="https://github.com/user-attachments/assets/45de3b22-7336-442b-b5ae-10e9c6836a64" />
</p>

*This interaction shows Copilot successfully retrieving the file through its direct resource ID while still failing to surface it through search, demonstrating the core OneDrive indexing defect documented in this repository.*
