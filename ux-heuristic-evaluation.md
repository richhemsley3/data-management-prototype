# UX Heuristic Evaluation Report
## DataShield Data Management Platform (15-Page Prototype)

**Evaluator:** Claude (UX Heuristic Evaluation)
**Date:** March 16, 2026
**Scope:** All 15 pages of the DataShield static HTML prototype
**Frameworks:** Nielsen's 10 Usability Heuristics, Laws of UX

---

## 1. Executive Summary

| Severity | Count |
|----------|-------|
| Critical | 4 |
| Major | 12 |
| Minor | 15 |
| **Total Issues** | **31** |

The DataShield prototype demonstrates strong foundational design work: a coherent design system with well-structured tokens, consistent shell layout across all 15 pages, thoughtful information architecture, and good use of progressive disclosure. The prototype is well above average for this stage of product development. The issues identified below are opportunities to elevate the design from good to excellent, particularly around destructive action safeguards, confirmation dialogs, keyboard navigation, and information density management.

---

## 2. Critical Issues

### C1. No Confirmation for Destructive Actions
**Heuristic:** H3 (User Control and Freedom), H5 (Error Prevention), H9 (Help Users Recover from Errors)
**Location:** `settings.html` (Delete button in bulk toolbar), `api-keys.html` (Revoke buttons), `users.html` (more_vert menus imply delete)
**Problem:** The "Delete" button in the source management bulk toolbar and the "Revoke" buttons on API keys have no confirmation dialog or undo mechanism. Revoking a production API key or deleting a data source are high-consequence, irreversible actions. In a data governance product, accidentally revoking `dsk_prod_...7x3f` (used 2 min ago, 1,234 queries/week) could cause production outages.
**Recommendation:** Add a confirmation modal for all destructive actions that states: (1) what will be affected, (2) the impact (e.g., "This will immediately invalidate all API calls using this key"), and (3) requires typing the resource name for high-severity deletions. Consider a brief undo window for bulk operations.

### C2. Duplicate / Competing Search Experiences on Home Dashboard
**Heuristic:** H4 (Consistency and Standards), H7 (Flexibility and Efficiency of Use)
**Law of UX:** Hick's Law (too many choices slow decision-making)
**Location:** `index.html` -- header search bar AND search hero in main content
**Problem:** The home page presents two visually prominent search inputs with nearly identical placeholder text ("Search datasets, tables, columns..." vs. "Search your entire data estate..."). Users must decide which to use, creating unnecessary cognitive load. Neither communicates how it differs from the other. This violates Hick's Law by splitting a single intent across two competing affordances.
**Recommendation:** Remove the search hero on the home page, or differentiate it clearly (e.g., make the hero a "quick search" with pre-populated suggestions, recently searched terms, or saved searches). The persistent header search should be the primary entry point for catalog search.

### C3. Security Score Displayed Redundantly with Conflicting Hierarchy
**Heuristic:** H8 (Aesthetic and Minimalist Design)
**Law of UX:** Miller's Law (information overload)
**Location:** `security.html` -- score hero card shows score as SVG gauge (73), large text (73), AND a trend indicator
**Problem:** The security score "73" is rendered three times in close proximity: inside the SVG circle, as a large text element next to it, and implied again in the trend. This wastes prime viewport space and dilutes the visual impact. The same pattern appears on `governance.html` (score 67). For security analysts scanning this page, the signal-to-noise ratio is suboptimal.
**Recommendation:** Choose one primary score representation (the SVG gauge with the number centered inside is the most effective). Remove the duplicate large text number. Use the freed space for contextual information like "73/100 -- Good (industry avg: 61)".

### C4. Connect Wizard Has No Escape Hatch During Long-Running Steps
**Heuristic:** H3 (User Control and Freedom), H1 (Visibility of System Status)
**Location:** `connect.html` -- Step 4 (Scan) shows 73% progress with 4m32s elapsed
**Problem:** During the scan step, the user sees a progress bar and elapsed time but there is no cancel/abort button for the scan itself. If a scan runs for 30+ minutes on a large Snowflake instance, the user is trapped. The "Cancel" button in the footer navigates away entirely (to settings.html) rather than canceling just the scan. There is also no way to run the scan in the background and return later.
**Recommendation:** Add a "Cancel Scan" button that stops the current scan without losing the connection. Add a "Run in Background" option that lets users navigate away and be notified when the scan completes. The footer Cancel should warn about unsaved progress.

---

## 3. Major Issues

### M1. Inconsistent Breadcrumb Patterns Across Settings Pages
**Heuristic:** H4 (Consistency and Standards), H6 (Recognition Rather Than Recall)
**Location:** `settings.html` breadcrumb links to `#` (broken), `organization.html` links to `settings.html`, `users.html` links to `#`, `api-keys.html` links to `#`, `audit-log.html` links to `#`
**Problem:** Settings sub-pages use inconsistent breadcrumb roots. Some link "Settings" to `#` (a dead link), while `organization.html` correctly links to `settings.html`. There is no dedicated settings landing page, making the breadcrumb hierarchy misleading. Users navigating via breadcrumbs from Users or API Keys hit a dead end.
**Recommendation:** Either create a settings landing page or use a consistent pattern where "Settings" in breadcrumbs links to the Sources page (as the first settings sub-page). Ensure all `#` hrefs are replaced with valid destinations.

### M2. Sidebar Navigation Label "Connectors" Links to a Wizard, Not a List
**Heuristic:** H2 (Match Between System and Real World), H6 (Recognition Rather Than Recall)
**Law of UX:** Jakob's Law (users expect navigation items to go to list/index pages)
**Location:** Sidebar: "Connectors" links to `connect.html` (a wizard)
**Problem:** Users expect clicking "Connectors" in the navigation to see a list of their existing connectors. Instead, they land on a "Connect a Source" wizard (a creation flow). The actual list of connected sources is under "Sources" (`settings.html`). This mapping is confusing: "Connectors" = wizard, "Sources" = list of connected sources. The distinction is unclear.
**Recommendation:** Rename "Connectors" to "Add Source" or move it into the Sources page as a primary action button (which already exists). Use the sidebar slot for a "Sources" entry that shows the list, with "Add Source" as a CTA within that page.

### M3. No Visible Loading/Empty States for Tab Content
**Heuristic:** H1 (Visibility of System Status)
**Location:** `classifications.html` -- "Sensitivity Levels", "PII Types", "Custom" tabs show placeholder empty states; `quality.html` tabs do not filter the table
**Problem:** When switching tabs on the Classifications page, two of the three secondary tabs show generic empty states ("Filter applied. Showing sensitivity level classifications only.") rather than filtered data. On the Quality page, tabs change visually but the table content does not change. Users cannot tell if the system is working or if there is genuinely no data.
**Recommendation:** Either populate all tab views with filtered data, or show a clear loading skeleton while "loading" and a meaningful empty state that includes a count ("0 custom classifications. Create your first custom classification.").

### M4. Tag Color Semantics Are Inconsistent Across Pages
**Heuristic:** H4 (Consistency and Standards)
**Law of UX:** Gestalt Principle of Similarity
**Location:** Multiple pages
**Problem:** Tag colors carry inconsistent meaning across the platform:
- "PII" is `sds-tag--error` (red) on `asset.html` and `classifications.html` (some rows), but `sds-tag--warning` (yellow) on the catalog and connect wizard pages.
- "PII" on `classifications.html` uses `sds-tag--error` for Email/Phone/SSN/Credit Card but `sds-tag--warning` for Full Name/Date of Birth -- even though all are PII.
- "Certified" is `sds-tag--purple` on `index.html` but `sds-tag--success` on `asset.html`.
- Domain tags use `sds-tag--neutral` on the home page but `sds-tag--info` on the catalog page.

Users rely on color to quickly scan and recognize patterns. Inconsistent color-meaning mapping forces users to read every label, defeating the purpose of color coding.
**Recommendation:** Create a definitive tag color mapping document: PII = always red, Certified = always purple, Domain = always one color, etc. Apply consistently across all 15 pages.

### M5. No Sort Indicators or Sortable Column Headers in Tables
**Heuristic:** H6 (Recognition Rather Than Recall), H7 (Flexibility and Efficiency of Use)
**Location:** All table instances across the platform (catalog, quality, classifications, glossary, users, settings, api-keys, audit-log)
**Problem:** None of the tables show sort direction indicators or offer column-header sorting. The catalog page has a "Sort by: Relevance" control but it is styled as plain text (easily missed) and does not indicate it is interactive. For a data management tool used by technical users, the inability to sort tables by any column (e.g., sort assets by risk score, sort users by last active, sort audit events by timestamp) is a significant efficiency gap.
**Recommendation:** Add sortable column headers with chevron indicators. Default to a sensible sort (e.g., most recent for audit log, highest risk for security). The sort state should persist within the session.

### M6. Pagination Lacks Per-Page Controls and Jump-to-Page
**Heuristic:** H7 (Flexibility and Efficiency of Use)
**Location:** `catalog.html`, `quality.html`, `classifications.html`, `glossary.html`, `audit-log.html`
**Problem:** Pagination only shows simple page numbers. There is no option to change rows per page (e.g., 10/25/50/100), no jump-to-page input, and no total page count shown. For a catalog with 1,847 assets, navigating page by page is inefficient. The audit log (1,847 events) is particularly affected.
**Recommendation:** Add a "rows per page" selector (at minimum 10/25/50), show the total page count, and for tables with 100+ pages, add a jump-to-page input.

### M7. Users Table Has a Pre-Checked Row with No Explanation
**Heuristic:** H5 (Error Prevention), H1 (Visibility of System Status)
**Location:** `users.html` -- Anika Kim's row has `checked` attribute on its checkbox
**Problem:** When the page loads, one user (Anika Kim) is pre-selected with no explanation of why. This creates confusion: is this user being edited? Is this a selection the user made previously? It also means the select-all checkbox shows an indeterminate state on load, which implies an active selection context.
**Recommendation:** Do not pre-select any rows on load. If the intent is to show "this is you," use a different visual indicator (e.g., a "You" badge or highlighted row) rather than a checkbox.

### M8. Asset Detail Page Has 7 Tabs Visible Simultaneously
**Heuristic:** H8 (Aesthetic and Minimalist Design)
**Law of UX:** Miller's Law (7 +/- 2 items), Hick's Law
**Location:** `asset.html` -- Overview, Schema, Access & Security, Lineage, Quality, Usage, Activity
**Problem:** Seven tabs compete for attention. While this is near Miller's limit, the tab bar becomes crowded especially with badge counts on three tabs (Schema: 42, Access & Security: 3, Quality: 98%). On smaller viewports, this tab row will overflow. The tabs are also not keyboard-navigable with arrow keys (only Tab key cycles through).
**Recommendation:** Consider grouping less-used tabs (Usage, Activity) under a "More" overflow, or use a scrollable tab bar with overflow indicators. Implement arrow-key navigation within the tab group per WAI-ARIA tab pattern.

### M9. Lineage Page Has No Breadcrumbs or Page Title in the Viewport
**Heuristic:** H1 (Visibility of System Status), H6 (Recognition Rather Than Recall)
**Location:** `lineage.html` -- full-bleed graph layout, no breadcrumbs, no h1 visible
**Problem:** The lineage page uses a full-bleed layout for the graph area. Unlike every other page, it has no breadcrumb, no page title (h1), and no subtitle. The only indication of location is the active sidebar item. When a user lands on this page from a deep link or browser back button, they have no contextual anchor.
**Recommendation:** Add a minimal toolbar header that includes the page title "Lineage" and optionally a breadcrumb. The existing toolbar has filter controls but no title.

### M10. Home Dashboard Quick Actions Cards Have Low Contrast Text
**Heuristic:** H8 (Aesthetic and Minimalist Design)
**Law of UX:** Aesthetic-Usability Effect
**Location:** `index.html` -- Quick Actions cards use `<a>` with `style="text-decoration:none"` wrapping card content
**Problem:** The Quick Actions cards suppress link underlines via inline style, which is fine for card patterns. However, the paragraph text inside these cards uses the default `card p` styling (`color: var(--sds-text-secondary)` which is `#54514D`). On the white card background (`#FFFFFF`), this meets WCAG AA at approximately 5.4:1 but the small font size (13px) combined with the muted color creates a low-contrast reading experience for the descriptive text. The card titles are stronger but the descriptions feel washed out.
**Recommendation:** Use `--sds-text-primary` for card body text, or increase the font size of card descriptions to at least 14px. Ensure all text meets WCAG AA at the rendered size.

### M11. Governance Page Has No Tab Navigation Despite Complex Content
**Heuristic:** H7 (Flexibility and Efficiency of Use)
**Location:** `governance.html` -- maturity hero + stat cards + AI readiness + priority tasks all in one scroll
**Problem:** The governance page is information-dense with at least four distinct content zones (score overview, category stats, AI-readiness grid, priority tasks) all presented as a single long scroll. Unlike peer pages (quality, classifications, glossary) which use tabs to segment content, governance forces the user to scroll to find priority tasks. The AI-Readiness section is particularly buried.
**Recommendation:** Consider adding tabs to separate "Overview" (score + stats) from "Tasks" and "AI Readiness." Alternatively, add an anchor navigation or sticky section headers.

### M12. Filter Sidebar on Catalog Has No Apply/Reset Pattern
**Heuristic:** H3 (User Control and Freedom), H5 (Error Prevention)
**Location:** `catalog.html` -- filter sidebar
**Problem:** The filter sidebar checkboxes are presented as immediate-apply controls (no explicit "Apply" button). While this is a valid pattern, the "Clear all" button is small and positioned after the active filter pill. If a user accidentally checks multiple filters, there is no "undo last filter change" option. More importantly, the active filter pill shows "Domain: Marketing" but the filter panel shows Marketing already checked -- yet the result count says "24 of 1,847." The relationship between filter state and results is unclear because there is no explicit "N filters applied" indicator.
**Recommendation:** Add a persistent filter count indicator (e.g., "3 filters applied") near the results header. Consider adding an "Apply Filters" button if the dataset is large enough that each filter change triggers expensive queries.

---

## 4. Minor Issues

### m1. Search Input in Header Lacks `aria-label` on Some Pages
**Heuristic:** H10 (Help and Documentation)
**Location:** `catalog.html` header search input is missing `aria-label` (present on most other pages)
**Problem:** Inconsistent ARIA labeling across the duplicated header markup.
**Recommendation:** Add `aria-label="Search"` to all header search inputs.

### m2. Notification Badge Count Is Static Across All Pages
**Heuristic:** H1 (Visibility of System Status)
**Location:** All pages -- notification badge always shows "3"
**Problem:** While understandable in a prototype, the static badge count means users cannot tell which page they were on when notifications arrived, and there is no notification panel/dropdown.
**Recommendation:** For prototype fidelity, consider showing a notification dropdown or panel on at least one page to demonstrate the interaction pattern.

### m3. Inconsistent Use of `<code>` vs. Monospace Font Styling
**Heuristic:** H4 (Consistency and Standards)
**Location:** `governance.html` uses inline `<code>` with explicit font-family/size styles; `api-keys.html` uses `<code>` with inline font-family; `audit-log.html` uses `.cell-mono` class
**Problem:** Three different approaches to displaying monospaced content (technical identifiers, code values). Some use `<code>` with inline styles, some use a custom class.
**Recommendation:** Standardize on a `.cell-mono` or `.code-inline` utility class in `shell.css`.

### m4. Tooltip Implementation Only on Home Page
**Heuristic:** H10 (Help and Documentation)
**Location:** `index.html` -- trust signal tags have `data-tooltip` attributes; `catalog.html` does not
**Problem:** The home page asset table uses CSS tooltips on trust signal tags (e.g., "Data refreshed recently" for "Fresh:2h"), but the catalog page's identical trust signal tags lack tooltips. New users who learn the meaning on the home page lose that scaffolding on the catalog.
**Recommendation:** Apply `data-tooltip` attributes consistently to all trust signal tags across all pages.

### m5. The "More Actions" Button Reuses Header Icon Button Styling
**Heuristic:** H4 (Consistency and Standards)
**Location:** `quality.html`, `classifications.html`, `glossary.html`, `users.html` -- `sds-header-icon-btn` class used for table row actions
**Problem:** The `sds-header-icon-btn` class is semantically named for header usage but is reused for table row action buttons. This works visually but creates confusion in the codebase and could lead to unintended style changes if the header button styling is modified.
**Recommendation:** Create a dedicated `.table-action-btn` or `.icon-btn` utility class separate from the header-specific styling.

### m6. Wizard Step Panels Lack `role="tabpanel"` Semantics
**Heuristic:** H10 (Help and Documentation)
**Location:** `connect.html` -- step panels use `aria-label` but no `role` attributes
**Problem:** The wizard steps use `aria-label` on panels but do not implement full WAI-ARIA wizard/tab semantics.
**Recommendation:** Add `role="tabpanel"` and `aria-labelledby` references to connect each panel to its stepper step.

### m7. Security Page "Remediate"/"Fix" Button Ambiguity
**Heuristic:** H2 (Match Between System and Real World)
**Location:** `security.html` -- first risk row has both "Remediate" (tertiary) and "Fix" (primary) buttons
**Problem:** The first row in the Top Risks table has two action buttons ("Remediate" and "Fix") while all other rows have only one. The distinction between "Remediate" and "Fix" is unclear -- they appear synonymous.
**Recommendation:** Use one consistent action label across all rows. If there are genuinely different actions, label them distinctly (e.g., "View Details" and "Auto-Fix").

### m8. Pagination Position Below Table Is Not Visually Connected
**Heuristic:** H4 (Consistency and Standards)
**Law of UX:** Gestalt Principle of Proximity
**Location:** `catalog.html`, `quality.html` -- pagination is inside `sds-table-container`; `users.html`, `settings.html` -- pagination is outside
**Problem:** Pagination is sometimes inside the table container (visually grouped) and sometimes outside (visually separated). This creates inconsistent proximity grouping.
**Recommendation:** Standardize pagination placement inside the `sds-table-container` on all pages.

### m9. Organization Page Notification Checkboxes Use Native Styling
**Heuristic:** H4 (Consistency and Standards)
**Location:** `organization.html` -- notification preference toggles use plain `<input type="checkbox">`
**Problem:** These are conceptually on/off toggles but are rendered as checkboxes. Modern SaaS products typically use toggle switches for binary settings to differentiate them from multi-select checkboxes used elsewhere (filters, table selections).
**Recommendation:** Replace with toggle switch components for binary settings, keeping checkboxes for multi-select contexts.

### m10. Empty State Content Is Generic
**Heuristic:** H10 (Help and Documentation)
**Location:** `classifications.html` -- Sensitivity Levels, PII Types, Custom tab empty states
**Problem:** Empty states say "Filter applied. Showing [type] classifications only." without actionable guidance.
**Recommendation:** Empty states should include a CTA (e.g., "Create your first custom classification") and optionally a brief explanation or illustration.

### m11. No Keyboard Shortcut Hints in the UI
**Heuristic:** H7 (Flexibility and Efficiency of Use)
**Law of UX:** Flexibility and Efficiency of Use (accelerators for expert users)
**Location:** Platform-wide
**Problem:** For a technical audience (data engineers, security analysts), keyboard shortcuts would significantly improve efficiency. Common SaaS products in this space offer `/` to focus search, `G then H` for home, etc. No shortcut hints appear anywhere.
**Recommendation:** Add keyboard shortcut hints to the search bar placeholder (e.g., "Search... (/)") and consider a keyboard shortcut cheat sheet accessible from the help button.

### m12. Audit Log Filter Dropdowns Use HTML Entities for Arrows
**Heuristic:** H4 (Consistency and Standards)
**Location:** `audit-log.html` -- filter buttons use `&#9662;` (down triangle) instead of Material Symbols icons
**Problem:** The dropdown indicator on filter buttons uses an HTML entity character (`&#9662;`) while the rest of the UI consistently uses Material Symbols Outlined icons (e.g., `expand_more`). This creates a visual inconsistency in icon weight and style.
**Recommendation:** Replace `&#9662;` with the `expand_more` Material Symbol icon for consistency.

### m13. Tab Badge Styling Varies Between Pages
**Heuristic:** H4 (Consistency and Standards)
**Location:** `asset.html` uses `.tab-badge`, `quality.html` uses `.sds-tab-badge`, `users.html` and `glossary.html` use `.tab-badge`
**Problem:** Two different class names (`.tab-badge` and `.sds-tab-badge`) are used for the same component. Only `.tab-badge` is styled in `shell.css` via `.sds-tab .tab-badge`. The `.sds-tab-badge` class on quality.html is unstyled.
**Recommendation:** Standardize on `.tab-badge` and ensure it is used consistently.

### m14. The "Help" Button in the Header Has No Associated Content
**Heuristic:** H10 (Help and Documentation)
**Location:** All pages -- help icon button in header
**Problem:** The help button is present on all 15 pages but has no tooltip, dropdown, or linked help page. For a complex data governance product, contextual help is critical.
**Recommendation:** Add a tooltip ("Help") and, for prototype purposes, at least demonstrate the help panel pattern on one page.

### m15. Home Page "Recent Assets" Table Lacks Pagination or "View All" Link
**Heuristic:** H7 (Flexibility and Efficiency of Use)
**Law of UX:** Goal-Gradient Effect (show progress toward viewing all)
**Location:** `index.html` -- Recent Assets table shows 5 rows with no indication of total count
**Problem:** The Recent Assets table shows exactly 5 items but does not indicate whether there are more, or provide a link to the full catalog filtered by recent activity.
**Recommendation:** Add a "View all recent assets" link below the table, or show "5 most recent" label to set expectations.

---

## 5. Strengths

### S1. Consistent Shell Layout Across All 15 Pages
Every page shares the same header, sidebar navigation, and main content structure. The sidebar correctly highlights the active page, and the navigation grouping (core features, governance, settings) is logical and well-labeled. This is excellent adherence to Jakob's Law and Gestalt's principle of common region.

### S2. Well-Designed Token System
The CSS design system (`shell.css`) uses a comprehensive set of semantic tokens for colors, spacing, typography, and status states. The warm-gray palette provides a professional, non-clinical aesthetic appropriate for enterprise software. The `--sds-*` namespace is clean and well-organized.

### S3. Trust Signals on Asset Tables Are Innovative
The combination of freshness, quality, certification, and security tags directly on table rows gives users a multi-dimensional trust assessment at a glance. The color-coded tags allow rapid scanning. This is a differentiating UX pattern for data catalogs.

### S4. Governance Maturity Model Is Well-Visualized
The governance page's stage progress bar (Search > Ownership > Classification > Compliance > Maturity) combined with the maturity score and gamified priority tasks creates a compelling improvement loop. The "+8 pts" impact indicators on tasks leverage the Goal-Gradient Effect and Zeigarnik Effect effectively.

### S5. Connect Wizard Is Thorough and Well-Structured
The 6-step wizard (Select Source > Authenticate > Discover > Scan > Review > Invite Team) follows a logical flow. The stepper visually shows progress (completed steps get checkmarks), the source tile grid is scannable, and the review step lets users verify AI-generated documentation before publishing. The wizard footer with Cancel/Back/Continue is well-positioned.

### S6. Security Posture Page Uses Progressive Disclosure Well
The security page layers information from high-level score to browse paths to specific risks, allowing users to start with an overview and drill into details. The browse path cards ("Unclassified Assets: 412", "Over-Permissioned: 89") are excellent entry points for different security workflows.

### S7. Asset Detail Page Is Comprehensive Without Overwhelming
The two-column layout on the asset detail overview (description + column summary on the left, metadata sidebar on the right) is a strong pattern. The 60/40 split prioritizes content while keeping metadata visible. The tab system, despite being crowded (see M8), successfully organizes seven distinct aspects of an asset.

### S8. Consistent ARIA Labeling
Most interactive elements have appropriate `aria-label` attributes. Tables have `aria-label`, buttons have descriptive labels, and the tab system uses `role="tablist"`, `role="tab"`, and `aria-selected`. The lineage graph nodes use `role="button"` and `tabindex="0"` for keyboard access.

### S9. Effective Use of Alert Banners for Timely Information
Pages like security, quality, settings, and governance use contextual alert banners to surface timely information (e.g., "1 source has breached its sync SLA", "3 new quality rule failures detected"). These banners use appropriate severity colors and include action links.

### S10. Warm, Non-Clinical Visual Design
The warm-gray color palette and Proxima Nova typography create a professional but approachable aesthetic that avoids the sterile feel common in enterprise data tools. This leverages the Aesthetic-Usability Effect -- users perceive better-looking interfaces as more usable.

---

## 6. Prioritized Top 10 Recommendations

| Priority | Recommendation | Severity | Effort | Impact |
|----------|---------------|----------|--------|--------|
| 1 | Add confirmation dialogs for all destructive actions (delete, revoke, remove) | Critical | Medium | High -- prevents data loss and production outages |
| 2 | Standardize tag color semantics across all pages (create a color-meaning map) | Major | Low | High -- reduces cognitive load on every page |
| 3 | Fix breadcrumb links on settings sub-pages (eliminate `#` hrefs) | Major | Low | Medium -- basic navigation reliability |
| 4 | Remove duplicate search input on home page or differentiate clearly | Critical | Low | Medium -- reduces decision paralysis |
| 5 | Add sortable column headers to all data tables | Major | Medium | High -- essential for technical users working with data |
| 6 | Add cancel/background-run options to the connect wizard scan step | Critical | Medium | High -- prevents user frustration during long operations |
| 7 | Resolve sidebar "Connectors" vs "Sources" naming confusion | Major | Low | Medium -- improves navigation predictability |
| 8 | Populate all tab content panels with real filtered data (remove placeholder empty states) | Major | Medium | Medium -- validates the filtering model |
| 9 | Add rows-per-page control and improved pagination for large datasets | Major | Low | Medium -- essential for catalog and audit log |
| 10 | Add keyboard shortcut support (at minimum `/` for search focus) | Minor | Low | Medium -- efficiency gain for expert users |

---

## Appendix: Evaluation Coverage Matrix

| Page | Nielsen Heuristics Evaluated | Laws of UX Applied |
|------|-----------------------------|--------------------|
| index.html | H1, H4, H7, H8 | Hick's Law, Serial Position Effect |
| catalog.html | H3, H5, H6, H7 | Gestalt (Proximity), Miller's Law |
| asset.html | H1, H6, H7, H8 | Miller's Law, Hick's Law |
| security.html | H2, H4, H8 | Von Restorff Effect, Cognitive Load |
| lineage.html | H1, H6, H7 | Fitts's Law, Gestalt (Common Region) |
| quality.html | H1, H4, H7 | Gestalt (Similarity) |
| classifications.html | H1, H4, H10 | Gestalt (Similarity) |
| glossary.html | H4, H6, H7 | Jakob's Law |
| governance.html | H7, H8 | Goal-Gradient, Zeigarnik Effect |
| connect.html | H1, H3, H5 | Doherty Threshold |
| settings.html | H3, H5, H9 | Tesler's Law |
| organization.html | H4 | Gestalt (Common Region) |
| users.html | H1, H4, H5 | Gestalt (Proximity) |
| api-keys.html | H3, H5, H9, H10 | Von Restorff Effect |
| audit-log.html | H4, H7 | Serial Position Effect, Cognitive Load |
