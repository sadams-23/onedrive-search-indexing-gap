# Copilot Cannot Find Files That Exist on OneDrive
Microsoft Feedback Hub Submission | Stephanie Adams | May 4, 2026

## Summary

Copilot's file search fails to find documents that are confirmed to exist and be fully synced on OneDrive. A file named "Resume.docx" (16.9 KB) stored at the root level of a connected OneDrive account was invisible to all five search strategies attempted — keyword, exact filename, name variations, and both browse methods — despite being uploaded over 12 hours prior and despite 22 other files in the same account being returned normally. The search index is the single point of failure.

## Severity

High — The file search function returns false negatives for files that are fully synced and accessible, causing the primary document retrieval workflow to fail silently. Users are told their files do not exist when they do.

## Component

OneDrive search index / Copilot file search (search_files connector)

## Description

A file named "Resume.docx" (16.9 KB) stored at the root level of a personal OneDrive account was completely invisible to Copilot's file search. The OneDrive connection was active and functional — other files in the same root directory were returned without issue. Five distinct search strategies were used over the course of the session — keyword, exact filename, name variations, open browse by name, and open browse by modified date — all returning zero results. A second file ("Stephanie Adams - Resume") in the same account was also invisible, suggesting a pattern rather than a one-off anomaly.

## Reproducibility

100% during testing. The file was not returned by any of five different search strategies attempted across the session. The issue persisted for the entire duration of the session (approximately 2 hours).
The file had been uploaded to OneDrive no more than the night before the testing session, meaning the search index had at minimum overnight to catalog a single 16.9 KB file at the root level — and still failed to do so.

## Steps to Reproduce

1. Connect a personal OneDrive account to Copilot.
2. Store a file named "Resume.docx" at the root level of OneDrive. Confirm it is visible at onedrive.live.com.
3. Wait at minimum 12 hours to allow the OneDrive search index to catalog the file. (During the original test, the file had been uploaded the night before and was given overnight to index.)
4. Ask Copilot to search for the file using the keyword "resume."
5. Observe: file does not appear in results (other files do).
6. Repeat the search using: exact filename "Resume.docx," broad keyword variations ("CV," user's first and last name), a full open browse with no keywords sorted by name, and a full open browse sorted by modified date.
7. Observe: the file does not appear in any of the five search attempts, despite 22 other files being returned across these searches.

## Verification Performed

- Confirmed the file exists on the cloud (not just locally) by opening it through the OneDrive web interface in a browser session.
- Retrieved the file successfully using its direct resource ID (obtained from the browser URL), proving the file is stored and accessible — just not indexed for search.
- A second resume file in the same account was also invisible to search, suggesting the indexing gap affects multiple files.

## Expected Behavior

At minimum, a direct filename match should always return a file that exists on the cloud. More broadly, all synced files should be discoverable through keyword and browse searches within a few hours of upload.

## Actual Behavior

Files that are fully synced, accessible via the web interface, and retrievable by direct resource ID are not returned by any search query — keyword, exact filename, name variations, or browse method. The user is told the file does not exist when it does.

## Likely Root Cause

The OneDrive search API relies on a search index rather than querying the file system directly. When the index fails to catalog a file, there is no fallback mechanism. As a result, a single indexing failure makes a file permanently invisible to the user.

## Current Workaround

If a file cannot be found through search, it can be retrieved by navigating to the OneDrive web interface (onedrive.live.com), locating the file manually, extracting its direct resource ID from the browser URL, and fetching it by ID through the Copilot connector. This requires technical knowledge of resource IDs and browser-based troubleshooting, making it impractical for general users.

## Suggested Improvements

- When a search returns no results for a specific filename, implement a fallback that queries the file system directly rather than relying solely on the search index.
- Surface a user-facing indicator when search results may be incomplete due to indexing delays (e.g., "Some recently added files may not appear yet").
- Provide a mechanism for users to trigger a manual re-index of their OneDrive account through Copilot.
- Investigate whether folder structures are indexed for search — no folders were returned during any search query in this session.

## Impact

A user asked Copilot to find and format a resume stored on OneDrive. Copilot reported the file did not exist. Five different search attempts confirmed the same false negative. The file was only located through manual browser-based troubleshooting that most users would not know how to perform. For users who connect OneDrive specifically to work with their stored documents, this failure at the first step — finding the file — undermines the core value of the integration.


## Environment

| Field | Value |
| ------- | ------- |
| Platform | Copilot Tasks (web interface) |
| Connected Service | OneDrive (Personal, Microsoft account)|
| Date of Testing | April 29, 2026 |
| File | Resume.docx (16.9 KB, root level) |

