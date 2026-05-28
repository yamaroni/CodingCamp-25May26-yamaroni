# Requirements Document

## Introduction

The Expense & Budget Visualizer is a client-side web application that allows users to track personal expenses by category, view a running total balance, and visualize spending distribution through a pie chart. The app runs entirely in the browser using HTML, CSS, and Vanilla JavaScript, with all data persisted via the browser's Local Storage API. No backend server or build toolchain is required.

## Glossary

- **App**: The Expense & Budget Visualizer web application.
- **Transaction**: A single expense entry consisting of an item name, a monetary amount, and a category.
- **Category**: One of three predefined spending groups — Food, Transport, or Fun.
- **Transaction_List**: The scrollable UI component that displays all recorded transactions.
- **Balance_Display**: The UI component at the top of the page that shows the current total of all transaction amounts.
- **Chart**: The pie chart component that visualizes spending distribution by category.
- **Form**: The input form used to create a new transaction.
- **Validator**: The client-side logic that checks form inputs before a transaction is recorded.
- **Storage**: The browser Local Storage API used to persist transaction data.
- **Chart_Library**: An external charting library (e.g., Chart.js) used to render the pie chart.

---

## Requirements

### Requirement 1: Transaction Input Form

**User Story:** As a user, I want to fill in a form with an item name, amount, and category so that I can record a new expense transaction.

#### Acceptance Criteria

1. THE Form SHALL contain an item name text field (maximum 100 characters), a numeric amount field (accepting values from 0.01 to 999,999.99 with up to 2 decimal places), and a category selector with options: Food, Transport, and Fun, where no category is pre-selected by default.
2. WHEN the user submits the Form with all fields filled and a valid positive numeric amount within the accepted range, THE App SHALL add the transaction to the Transaction_List and persist it to Storage.
3. WHEN the user submits the Form with one or more empty fields, THE Validator SHALL display an inline error message adjacent to each empty field indicating it is required and SHALL NOT add a transaction.
4. WHEN the user submits the Form with a non-positive, non-numeric, out-of-range, or more-than-2-decimal-place value in the amount field, THE Validator SHALL display an inline error message on the amount field and SHALL NOT add a transaction.
5. WHEN a transaction is successfully added, THE Form SHALL reset all fields to their default empty/unselected state.
6. IF persisting the transaction to Storage fails (e.g., Storage quota exceeded or Storage is unavailable), THE App SHALL display an error message to the user and SHALL NOT add the transaction to the Transaction_List.

---

### Requirement 2: Transaction List

**User Story:** As a user, I want to see a scrollable list of all my recorded transactions so that I can review my spending history.

#### Acceptance Criteria

1. THE Transaction_List SHALL display all persisted transactions in reverse-chronological order (most recent first), each showing the item name, amount formatted as a signed currency value with a currency symbol and 2 decimal places (e.g., -$12.50), and category.
2. WHILE the number of transactions exceeds the visible area of the Transaction_List, THE Transaction_List SHALL be scrollable to reveal all entries.
3. WHEN the App loads in the browser, THE Transaction_List SHALL render all transactions previously saved in Storage.
4. WHEN the user clicks the delete control on a transaction entry, THE App SHALL immediately and synchronously remove that transaction from the Transaction_List and from Storage.
5. WHEN the App loads and Storage contains no transactions, THE Transaction_List SHALL display an empty state message indicating no transactions have been recorded.
6. IF reading from Storage fails on load, THE Transaction_List SHALL display an error state message and render no transaction entries.

---

### Requirement 3: Total Balance Display

**User Story:** As a user, I want to see my total spending balance at the top of the page so that I always know how much I have spent in total.

#### Acceptance Criteria

1. THE Balance_Display SHALL show the sum of all transaction amounts formatted as a currency value with a currency symbol prefix, 2 decimal places, and comma thousands separators (e.g., $1,234.56).
2. WHEN a new transaction is added, THE Balance_Display SHALL update to reflect the new total within 1 second and without requiring a page reload.
3. WHEN a transaction is deleted, THE Balance_Display SHALL update to reflect the reduced total within 1 second and without requiring a page reload.
4. WHEN the App loads with no transactions in Storage, THE Balance_Display SHALL show a total of $0.00.
5. IF the sum of all transaction amounts is negative, THE Balance_Display SHALL display the value with a leading minus sign (e.g., -$50.00).

---

### Requirement 4: Spending Distribution Chart

**User Story:** As a user, I want to see a pie chart of my spending by category so that I can understand where my money is going.

#### Acceptance Criteria

1. THE Chart SHALL render as a pie chart using Chart_Library, with each slice representing one of the three categories: Food, Transport, and Fun. Only categories with a total spending amount greater than 0 SHALL be rendered as slices.
2. WHEN a new transaction is added, THE Chart SHALL update to reflect the new spending distribution within 500 milliseconds and SHALL NOT require or trigger a page reload under any circumstances.
3. WHEN a transaction is deleted, THE Chart SHALL update to reflect the revised spending distribution within 500 milliseconds and SHALL NOT require or trigger a page reload under any circumstances.
4. WHEN the App loads and Storage contains no transactions, THE Chart SHALL render in a placeholder state displaying a text message indicating no spending data is available.
5. IF transactions exist in Storage but a data loading error occurs, THEN THE Chart SHALL also render in the placeholder state described in criterion 4.
6. THE Chart SHALL display a legend identifying each category by name and a distinct color not shared by any other category.

---

### Requirement 5: Data Persistence

**User Story:** As a user, I want my transactions to be saved between browser sessions so that I do not lose my data when I close or refresh the page.

#### Acceptance Criteria

1. WHEN a transaction is added, THE Storage SHALL persist the complete transaction data (name, amount, category) before the next user interaction is accepted.
2. WHEN a transaction is deleted, THE Storage SHALL remove the corresponding transaction data before the next user interaction is accepted.
3. WHEN the App loads and Storage contains one or more valid transactions, THE App SHALL read all transactions from Storage and restore the Transaction_List, Balance_Display, and Chart to the state they were in at the end of the previous session.
4. WHEN the App loads and Storage contains no transactions, THE App SHALL initialize an empty state with an empty Transaction_List, a Balance_Display showing $0.00, and the Chart in its placeholder state.
5. IF reading from Storage fails due to data that is unparseable or contains records missing required fields (name, amount, or category), THEN THE App SHALL discard the corrupted data, initialize an empty state as described in criterion 4, and allow the user to start fresh.

---

### Requirement 6: Technical and Structural Constraints

**User Story:** As a developer, I want the codebase to follow defined structural rules so that the project remains clean, maintainable, and dependency-free on the server side.

#### Acceptance Criteria

1. THE App SHALL be implemented using only HTML, CSS, and Vanilla JavaScript with no JavaScript frameworks (e.g., React, Vue, Angular).
2. THE App SHALL contain exactly one CSS file located inside the `css/` directory.
3. THE App SHALL contain exactly one JavaScript file located inside the `js/` directory.
4. THE App SHALL require no backend server and SHALL be runnable by opening `index.html` directly in a browser or as a browser extension without any local server or build step.
5. WHERE Chart_Library is used, THE App SHALL load it via a CDN `<script>` tag present in the HTML file, and the library SHALL load without any local installation step or package manager.
6. THE App SHALL function correctly in stable versions of Chrome, Firefox, Edge, and Safari released within the 12 months prior to testing, meaning: no uncaught JavaScript errors are thrown, all UI elements render visibly, and all interactive features (form submission, delete, chart update) respond to user input.

---

### Requirement 7: Performance and Visual Design

**User Story:** As a user, I want the app to load quickly and look clean so that using it feels effortless and pleasant.

#### Acceptance Criteria

1. WHEN the App is opened on a connection of at least 25 Mbps download speed, THE App SHALL render the full initial UI within 2 seconds.
2. WHEN the user adds or deletes a transaction, THE App SHALL update the Transaction_List, Balance_Display, and Chart within 100 milliseconds.
3. THE App SHALL apply a consistent visual hierarchy where heading font sizes are at least 1.25× the body font size, and all interactive controls have a visible focus indicator of at least 2px.
4. THE App SHALL use readable font sizes of at least 14px for body text and at least 18px for headings.
5. THE App SHALL provide sufficient color contrast between text and background to meet WCAG 2.1 AA contrast ratio requirements (minimum 4.5:1 for normal text).
6. IF a transaction add or delete operation fails (e.g., Storage error), THE App SHALL display a visible error message to the user and SHALL NOT update the Transaction_List, Balance_Display, or Chart with partial or inconsistent data.
