# Implementation Plan: Expense & Budget Visualizer

## Overview

Implement a fully client-side single-page web app using plain HTML, CSS, and Vanilla JavaScript. The app records expense transactions, displays a running balance, and visualizes spending by category via a Chart.js pie chart. All data is persisted in `localStorage`. The implementation follows the component architecture defined in the design: `StorageService`, `Validator`, `StateManager`, `Renderer`, `ChartController`, and `FormController` — all defined in a single `js/app.js` file.

---

## Tasks

- [x] 1. Set up project structure and HTML skeleton
  - Create `index.html` with semantic HTML structure: header with balance display, main section with form and transaction list, chart canvas, and a global error banner element
  - Add Chart.js 4.x CDN `<script>` tag to `index.html`
  - Create `css/styles.css` with base reset, typography (body ≥ 14px, headings ≥ 18px), layout, and focus indicator styles (≥ 2px outline)
  - Create `js/app.js` as an empty module scaffold with placeholder `const` objects for all six components
  - Ensure `index.html` links `css/styles.css` and `js/app.js`
  - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5, 7.3, 7.4_

- [-] 2. Implement `StorageService` and data models
  - [-] 2.1 Implement `StorageService` with `load()` and `saveAll()` methods
    - `load()`: reads `localStorage["expense_transactions"]`, parses JSON, validates each record (name string, amount positive number, category in allowed set), discards corrupted records, returns clean `Transaction[]`
    - `saveAll(transactions)`: serializes array with `JSON.stringify` and writes to `localStorage`; wraps both operations in `try/catch` and throws `StorageError` with appropriate `type` (`'quota_exceeded'`, `'unavailable'`, `'parse_error'`) on failure
    - Implement `StorageError` custom error class extending `Error` with a `type` property
    - _Requirements: 5.1, 5.2, 5.5, 1.6_



- [ ] 3. Implement `Validator`
  - [~] 3.1 Implement `Validator` with `validateForm()`, `isValidAmount()`, `isValidName()`, and `isValidCategory()` methods
    - `isValidName(value)`: returns `true` if non-empty string and ≤ 100 chars
    - `isValidAmount(value)`: returns `true` if parseable as a positive number, ≤ 999999.99, and has ≤ 2 decimal places
    - `isValidCategory(value)`: returns `true` if value is exactly `'Food'`, `'Transport'`, or `'Fun'`
    - `validateForm(rawName, rawAmount, rawCategory)`: calls the three validators and returns `{ valid: true, data: {...} }` or `{ valid: false, errors: { name?, amount?, category? } }`
    - _Requirements: 1.1, 1.3, 1.4_



- [ ] 4. Implement `ChartController`
  - [~] 4.1 Implement `ChartController` with `init()`, `update()`, and `showPlaceholder()` methods
    - `init(canvasId)`: creates a Chart.js pie chart instance on the specified canvas with three dataset slots (Food, Transport, Fun) and distinct colors; stores instance in `ChartController.instance`
    - `update(categoryTotals)`: mutates `chart.data.datasets[0].data` and `chart.data.labels` in-place (omitting categories with 0 total), then calls `chart.update()` — must complete within 500 ms
    - `showPlaceholder(message)`: hides the canvas element and shows a placeholder text element with the given message
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.6_

 

- [ ] 5. Implement `Renderer`
  - [~] 5.1 Implement `Renderer` with `renderAll()`, `renderList()`, `renderBalance()`, `renderChart()`, `showError()`, `showFieldErrors()`, and `clearFieldErrors()` methods
    - `renderList(transactions)`: clears the list DOM node and rebuilds it; renders transactions in reverse-chronological order; each entry shows name, amount (via `formatCurrency`), category, and a delete button with a `data-id` attribute; shows empty-state message when array is empty
    - `renderBalance(transactions)`: sums all amounts and updates the balance display element using `formatCurrency`; handles negative totals with leading minus sign
    - `renderChart(transactions)`: computes `CategoryTotals` from the array and calls `ChartController.update()`; calls `ChartController.showPlaceholder()` when array is empty
    - `renderAll(transactions)`: calls `renderList`, `renderBalance`, and `renderChart` in sequence
    - `showError(context, message)`: for `'form'` shows inline form error; for `'global'` shows the global error banner
    - `showFieldErrors(errors)`: inserts inline error messages adjacent to each invalid field
    - `clearFieldErrors()`: removes all inline field error messages
    - Implement `formatCurrency(amount)` utility: positive → `$1,234.56`, zero → `$0.00`, negative → `-$50.00`
    - _Requirements: 2.1, 2.2, 2.5, 3.1, 3.4, 3.5, 4.1, 4.4, 7.6_

 

- [ ] 6. Implement `StateManager`
  - [~] 6.1 Implement `StateManager` with `init()`, `addTransaction()`, and `deleteTransaction()` methods
    - `init()`: calls `StorageService.load()`, populates `StateManager.transactions`; on `StorageError` sets `transactions = []` and calls `Renderer.showError('global', ...)`
    - `addTransaction(formData)`: generates a UUID via `crypto.randomUUID()`, constructs a `Transaction` object with `Date.now()` timestamp, calls `StorageService.saveAll([...transactions, newTx])`, pushes to `transactions[]`, calls `Renderer.renderAll(transactions)`; on `StorageError` calls `Renderer.showError('global', ...)` without mutating `transactions[]`
    - `deleteTransaction(id)`: splices the matching transaction from `transactions[]`, calls `StorageService.saveAll(transactions)`, calls `Renderer.renderAll(transactions)`; on `StorageError` calls `Renderer.showError('global', ...)` and restores the spliced item
    - _Requirements: 1.2, 1.6, 2.3, 2.4, 5.1, 5.2, 5.3, 7.6_

 

- [ ] 7. Checkpoint — Core logic complete
  - Ensure all unit tests pass (if written). Verify `StorageService`, `Validator`, `StateManager`, `Renderer`, and `ChartController` are wired together correctly in `js/app.js`. Ask the user if any questions arise before proceeding.

- [ ] 8. Implement `FormController` and wire up event listeners
  - [~] 8.1 Implement `FormController` with `init()`, `handleSubmit()`, and `resetForm()` methods
    - `init(formId)`: attaches a `submit` event listener to the form element
    - `handleSubmit(event)`: calls `event.preventDefault()`, calls `Renderer.clearFieldErrors()`, reads raw field values, calls `Validator.validateForm()`; on invalid result calls `Renderer.showFieldErrors(errors)`; on valid result calls `StateManager.addTransaction(data)` then `resetForm()`
    - `resetForm()`: resets all form fields to default empty/unselected state
    - _Requirements: 1.2, 1.3, 1.4, 1.5_

  - [~] 8.2 Wire delete button event delegation on the transaction list
    - Attach a single `click` event listener to the transaction list container using event delegation
    - On click of a delete button, read `data-id` and call `StateManager.deleteTransaction(id)`
    - _Requirements: 2.4_

  - [~] 8.3 Write the app entry point
    - At the bottom of `js/app.js`, add a `DOMContentLoaded` listener that calls `ChartController.init('chart-canvas')`, `StateManager.init()`, and `FormController.init('expense-form')` in that order
    - _Requirements: 2.3, 5.3, 5.4_



- [ ] 9. Apply visual styles in `css/styles.css`
  - [~] 9.1 Implement layout and component styles
    - Style the balance display as a prominent heading (≥ 18px), the form with labeled fields and visible focus indicators (≥ 2px outline), the transaction list as a scrollable container, and the chart canvas area
    - Apply consistent color palette with WCAG 2.1 AA contrast ratios (≥ 4.5:1 for normal text)
    - Style the empty-state message, error banner, and inline field error messages
    - Style the delete button on each transaction entry
    - _Requirements: 7.3, 7.4, 7.5, 2.2_

  - [~] 9.2 Implement responsive and cross-browser compatibility
    - Ensure layout renders correctly in Chrome, Firefox, Edge, and Safari (no vendor-prefixed properties that break modern stable versions)
    - Verify no uncaught JS errors are thrown on load in all four browsers (use standard APIs: `localStorage`, `crypto.randomUUID`, `Chart.js` CDN)
    - _Requirements: 6.6_

- [ ] 10. Final checkpoint — Full integration
  - Open `index.html` directly in a browser (no server). Verify: form submission adds a transaction and updates list, balance, and chart; delete removes a transaction and updates all three; data persists after page refresh; empty state shows correctly on first load; storage error path shows error banner without corrupting state. Ensure all automated tests pass. Ask the user if any questions arise.

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Each task references specific requirements for traceability
- Checkpoints (tasks 7 and 10) ensure incremental validation before moving forward
- The design has no Correctness Properties section, so property-based tests are not included — unit tests cover validation logic and utility functions
- All six components (`StorageService`, `Validator`, `StateManager`, `Renderer`, `ChartController`, `FormController`) must live in the single `js/app.js` file per Requirement 6.3
- `crypto.randomUUID()` is available in all modern browsers (Chrome 92+, Firefox 95+, Edge 92+, Safari 15.4+) — no polyfill needed

---

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1"] },
    { "id": 1, "tasks": ["2.1", "3.1", "4.1"] },
    { "id": 2, "tasks": ["2.2", "3.2", "4.2", "5.1"] },
    { "id": 3, "tasks": ["5.2", "6.1"] },
    { "id": 4, "tasks": ["6.2", "8.1", "8.2", "8.3"] },
    { "id": 5, "tasks": ["8.4", "9.1"] },
    { "id": 6, "tasks": ["9.2"] }
  ]
}
```
