# Design Document: Expense & Budget Visualizer

## Overview

The Expense & Budget Visualizer is a fully client-side single-page application (SPA) built with plain HTML, CSS, and Vanilla JavaScript. It enables users to record personal expense transactions, view a running total balance, and visualize spending by category through a pie chart. All data is persisted in the browser's `localStorage` so sessions survive page refreshes and browser restarts.

The application has no build step, no server, and no JavaScript framework. It is opened directly as `index.html` in a browser. Chart rendering is delegated to [Chart.js 4.x](https://www.chartjs.org/docs/latest/) loaded via CDN.

### Key Design Decisions

- **No framework**: Vanilla JS with direct DOM manipulation keeps the dependency surface minimal and the app runnable without any toolchain.
- **Single JS file / Single CSS file**: Enforced by Requirement 6. All logic lives in `js/app.js`; all styles in `css/styles.css`.
- **Synchronous localStorage**: All reads and writes are synchronous, which simplifies the state model and satisfies the "before next user interaction" persistence requirement.
- **Atomic UI updates**: On every add/delete, the app recomputes the full state (balance, list, chart) from the in-memory transaction array and re-renders all three components together, preventing partial/inconsistent UI states.
- **Chart.js for pie chart**: Loaded via CDN `<script>` tag. The chart instance is created once on load and updated in-place using `chart.data` mutation + `chart.update()` to meet the 500 ms update requirement.

---

## Architecture

The application follows a simple **Model → View** pattern without a reactive framework:

```
┌─────────────────────────────────────────────────────────┐
│                        index.html                        │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  Form (HTML) │  │  List (HTML) │  │ Balance (HTML)│  │
│  └──────┬───────┘  └──────┬───────┘  └───────┬───────┘  │
│         │                 │                   │          │
│  ┌──────▼─────────────────▼───────────────────▼───────┐  │
│  │                    js/app.js                        │  │
│  │  ┌────────────┐  ┌──────────────┐  ┌────────────┐  │  │
│  │  │  Validator │  │ StateManager │  │  Renderer  │  │  │
│  │  └────────────┘  └──────┬───────┘  └─────┬──────┘  │  │
│  │                         │                │          │  │
│  │                  ┌──────▼───────┐  ┌─────▼──────┐  │  │
│  │                  │StorageService│  │ ChartCtrl  │  │  │
│  │                  └──────────────┘  └────────────┘  │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                           │
│  External: Chart.js 4.x (CDN)                            │
└─────────────────────────────────────────────────────────┘
```

### Data Flow

1. **App Init**: `StorageService.load()` → validate/parse → populate in-memory `transactions[]` → `Renderer.renderAll()`
2. **Add Transaction**: `Validator.validate(formData)` → `StorageService.save(tx)` → push to `transactions[]` → `Renderer.renderAll()`
3. **Delete Transaction**: `StorageService.remove(id)` → splice from `transactions[]` → `Renderer.renderAll()`
4. **Error path**: Any `StorageService` failure throws; the caller catches and calls `Renderer.showError(msg)` without mutating `transactions[]`

---

## Components and Interfaces

All components are plain JavaScript objects/functions defined in `js/app.js`. There are no classes required, but the design uses module-like namespacing via `const` objects.

### StorageService

Responsible for all `localStorage` interactions. Wraps every call in `try/catch`.

```js
const StorageService = {
  STORAGE_KEY: 'expense_transactions',

  // Returns Transaction[] or throws StorageError
  load(): Transaction[],

  // Persists full array; throws StorageError on quota/unavailable
  saveAll(transactions: Transaction[]): void,
};
```

- `load()` reads the JSON string, parses it, validates each record (name, amount, category present and valid types), discards corrupted records, and returns the clean array.
- `saveAll()` serializes the full array with `JSON.stringify` and calls `localStorage.setItem`. Throws a `StorageError` if the write fails (quota exceeded, private browsing restrictions, etc.).

### Validator

Pure functions that validate form input. Returns a result object rather than throwing.

```js
const Validator = {
  // Returns { valid: true, data: FormData } or { valid: false, errors: FieldErrors }
  validateForm(rawName: string, rawAmount: string, rawCategory: string): ValidationResult,

  // Returns true if amount string is a valid positive number ≤ 999999.99 with ≤ 2 decimal places
  isValidAmount(value: string): boolean,

  // Returns true if name is non-empty and ≤ 100 chars
  isValidName(value: string): boolean,

  // Returns true if category is one of 'Food' | 'Transport' | 'Fun'
  isValidCategory(value: string): boolean,
};
```

### StateManager

Holds the single source of truth: the in-memory `transactions` array. Coordinates between `StorageService` and `Renderer`.

```js
const StateManager = {
  transactions: Transaction[],  // in-memory array, reverse-chron order

  // Initializes from Storage; on error sets transactions=[] and flags error
  init(): void,

  // Validates, persists, updates array, triggers render; throws on storage failure
  addTransaction(formData: RawFormData): void,

  // Removes by id, persists, triggers render; throws on storage failure
  deleteTransaction(id: string): void,
};
```

### Renderer

Handles all DOM mutations. Reads from `StateManager.transactions` and updates the three UI regions plus the chart.

```js
const Renderer = {
  // Re-renders list, balance, and chart from current transactions[]
  renderAll(transactions: Transaction[]): void,

  // Updates the transaction list DOM
  renderList(transactions: Transaction[]): void,

  // Updates the balance display DOM
  renderBalance(transactions: Transaction[]): void,

  // Updates the Chart.js pie chart
  renderChart(transactions: Transaction[]): void,

  // Shows an inline error message in the form or a global error banner
  showError(context: 'form' | 'global', message: string): void,

  // Shows inline field validation errors
  showFieldErrors(errors: FieldErrors): void,

  // Clears all field errors
  clearFieldErrors(): void,
};
```

### ChartController

Manages the Chart.js instance lifecycle.

```js
const ChartController = {
  instance: Chart | null,

  // Creates the Chart.js pie chart on the <canvas> element
  init(canvasId: string): void,

  // Updates data in-place and calls chart.update()
  update(categoryTotals: CategoryTotals): void,

  // Shows placeholder text, hides canvas
  showPlaceholder(message: string): void,
};
```

### FormController

Handles form submission event, delegates to `Validator` and `StateManager`.

```js
const FormController = {
  // Attaches submit event listener to the form element
  init(formId: string): void,

  // Reads form fields, validates, calls StateManager.addTransaction
  handleSubmit(event: SubmitEvent): void,

  // Resets all form fields to default empty/unselected state
  resetForm(): void,
};
```

---

## Data Models

### Transaction

```js
/**
 * @typedef {Object} Transaction
 * @property {string}  id        - UUID v4 generated at creation time (crypto.randomUUID())
 * @property {string}  name      - Item name, 1–100 characters
 * @property {number}  amount    - Positive number, max 2 decimal places, range 0.01–999999.99
 * @property {string}  category  - One of 'Food' | 'Transport' | 'Fun'
 * @property {number}  timestamp - Unix ms timestamp (Date.now()) at creation
 */
```

### CategoryTotals

```js
/**
 * @typedef {Object} CategoryTotals
 * @property {number} Food
 * @property {number} Transport
 * @property {number} Fun
 */
```

### ValidationResult

```js
/**
 * @typedef {Object} ValidationResult
 * @property {boolean}     valid
 * @property {FormData}    [data]    - Present when valid === true
 * @property {FieldErrors} [errors]  - Present when valid === false
 */

/**
 * @typedef {Object} FieldErrors
 * @property {string} [name]     - Error message for name field
 * @property {string} [amount]   - Error message for amount field
 * @property {string} [category] - Error message for category field
 */
```

### StorageError

```js
/**
 * Custom error class for Storage failures.
 * @extends Error
 * @property {string} type - 'quota_exceeded' | 'unavailable' | 'parse_error'
 */
class StorageError extends Error {}
```

### Storage Schema

Transactions are stored as a JSON array under a single key:

```
localStorage key: "expense_transactions"
value: JSON.stringify(Transaction[])
```

Example:
```json
[
  {
    "id": "a1b2c3d4-...",
    "name": "Lunch",
    "amount": 12.50,
    "category": "Food",
    "timestamp": 1700000000000
  }
]
```

### Currency Formatting

The `formatCurrency(amount: number): string` utility function produces the display string:

- Positive: `$1,234.56`
- Zero: `$0.00`
- Negative: `-$50.00` (minus sign precedes the dollar sign)

Implementation uses `Math.abs()` + `toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })` with manual prefix assembly to ensure consistent cross-browser output.

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

