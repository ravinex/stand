# Canvas App Best Practices — Do & Don’t Guide

## 1. Architecture & Data Design
### ✅ Do
- Model your data before building screens.
- Prefer **Dataverse** for relational, secure, audited applications.
- Index columns used for filtering/sorting.
- Load only required data per screen.
- Cache static reference data using `Concurrent()` on App start.

### ❌ Don’t
- Don’t use non-delegable functions on large datasets.
- Don’t load entire tables into collections unless required.
- Don’t reload reference data on every screen.

## 2. Performance & Delegation
### ✅ Do
- Keep screens lightweight (<300 controls).
- Use **components**, **containers**, and **named formulas**.
- Enable `DelayOutput = true` on search inputs.
- Enable `DelayItemLoading = true` on galleries.
- Use delegable functions (`Filter`, `SortByColumns`, `StartsWith`).
- Use `Concurrent()` for parallel loading.

### ❌ Don’t
- Don’t use `ForAll` over remote tables for reads.
- Don’t use `AddColumns` + `LookUp` on large tables.
- Don’t embed large media files inside the app.

## 3. UX, UI & Accessibility
### ✅ Do
- Build responsive screens using containers & relative sizing.
- Centralize theme tokens: colors, spacing, radius, fonts.
- Set accessible labels, tab order, screen reader text.
- Provide loading, empty, and error states.

### ❌ Don’t
- Don’t hard-code pixel sizes or hex codes everywhere.
- Don’t rely only on color to communicate status.
- Don’t overuse pop-ups for simple warnings.

## 4. Coding Standards (Power Fx)
### ✅ Do
- Use `With()` for cleaner, scoped formulas.
- Use `UpdateContext()` for screen state (local vars).
- Use `Set()` sparingly for global variables.
- Alias records using `ThisItem As rec`.
- Centralize constants like URLs, version numbers, theme tokens.

### ❌ Don’t
- Don’t mix business logic inside UI control properties.
- Don’t duplicate formulas across screens.
- Don’t bury logic in many small controls — consolidate.

## 5. Naming Conventions

| Component | Prefix | Example |
|----------|--------|---------|
| Screen | `scr` | `scrHome` |
| Gallery | `gal` | `galOrders` |
| Button | `btn` | `btnSubmit` |
| Text Input | `txt` | `txtSearch` |
| Label | `lbl` | `lblHeader` |
| Icon | `ico` | `icoRefresh` |
| Component | `cmp` | `cmpHeader` |
| Global Variable | `gbl` | `gblCurrentUser` |
| Local Variable | `loc` | `locIsDirty` |
| Collection | `col` | `colRefStatus` |

## 6. Forms & Data Operations
### ✅ Do
- Validate input before Patch.
- Use `Patch()` for partial updates.
- Use `IfError()` and `Errors()` for error handling.
- Log save and failure events.

## 7. Security & Compliance
### ❌ Don’t
- Don’t assume Patch/SubmitForm always succeeds.
- Don’t block saves for minor warnings.
