# Feature: One Click Table Exports

## Metadata
issue_number: `1`
adw_id: `1f259f43`
issue_json: `{"number":1,"title":"One Click Table Exports","body":"Using adw_plan_build_review add one click table exports and one click result export feature to get results as csv files. \n\nCreate two new endpoints to support these features. On exporting tables, one for exporting query results. \n\nPlace a download button directly to the left of the 'x' icon for available tables. \nPlace a download button directly to the left of the 'hide' button for query results.\n\nUse the appropriate download icon."}`

## Feature Description
This feature adds one-click CSV export functionality to the Natural Language SQL Interface application. Users will be able to export both available tables and query results as CSV files with a single click. The feature adds download buttons to the UI alongside existing controls and creates two new backend endpoints to generate and serve CSV files.

The export functionality enables users to:
- Download complete table data from the "Available Tables" section as CSV files
- Download query results as CSV files after executing natural language queries
- Obtain data in a portable format for use in spreadsheet applications or other data analysis tools

## User Story
As a user of the Natural Language SQL Interface
I want to export tables and query results as CSV files with one click
So that I can use the data in external tools like Excel, Google Sheets, or other data analysis applications without manual copy-pasting

## Problem Statement
Currently, users can view tables and query results in the browser interface, but there is no built-in way to extract this data for external use. Users who want to use the data in spreadsheet applications, perform additional analysis, or share results with colleagues must manually copy-paste data from the UI, which is time-consuming, error-prone, and doesn't preserve proper CSV formatting. This creates a significant barrier to using the application as part of a larger data analysis workflow.

## Solution Statement
Implement two new FastAPI endpoints that generate CSV files from table data and query results, and add download buttons to the client UI that trigger these endpoints. The backend will use Python's built-in `csv` module to convert SQLite query results into properly formatted CSV strings, set appropriate HTTP headers for file downloads, and return the data as downloadable responses. The frontend will add icon-based download buttons positioned next to existing controls (to the left of the 'x' icon for tables, and to the left of the 'Hide' button for query results) that trigger browser downloads when clicked.

## Relevant Files
Use these files to implement the feature:

- `app/server/server.py` - Contains FastAPI application and endpoint definitions. Will add two new GET endpoints: `/api/table/{table_name}/export` and `/api/query/export`
- `app/server/core/data_models.py` - Contains Pydantic models for API requests/responses. May need to add export-related models if required
- `app/server/core/sql_security.py` - Contains SQL injection protection utilities. Will use `validate_identifier()` and `execute_query_safely()` to safely query tables for export
- `app/server/core/sql_processor.py` - Contains SQL execution logic. May leverage existing query execution patterns for fetching table data
- `app/client/src/main.ts` - Contains client-side logic for displaying tables and results. Will add download button initialization and click handlers
- `app/client/src/api/client.ts` - Contains API client methods. Will add methods for export endpoints if needed (or use direct fetch)
- `app/client/src/types.d.ts` - Contains TypeScript type definitions. May add export-related types if required
- `app/client/src/style.css` - Contains styling for UI components. Will add styles for download buttons to match existing button patterns
- `app/client/index.html` - Contains HTML structure. No changes expected, but may verify structure for button placement

### New Files
- `.claude/commands/e2e/test_one_click_table_exports.md` - E2E test file to validate the export functionality works correctly

## Implementation Plan

### Phase 1: Foundation
Create the backend infrastructure for CSV export by implementing two new API endpoints. These endpoints will handle safe SQL execution to retrieve table/query data and convert results to CSV format. Focus on security by using the existing `sql_security` module for identifier validation and safe query execution. The endpoints will set proper HTTP headers (`Content-Type: text/csv` and `Content-Disposition: attachment`) to trigger browser downloads.

### Phase 2: Core Implementation
Build the CSV generation logic using Python's built-in `csv` module to convert SQLite query results (list of dictionaries) into properly formatted CSV strings. Implement error handling for cases where tables don't exist or queries fail. Ensure the CSV output includes column headers and properly escapes special characters, quotes, and newlines.

### Phase 3: Integration
Add download buttons to the client UI using appropriate icons (📥 download icon). Position buttons correctly: to the left of the 'x' icon in the "Available Tables" section, and to the left of the 'Hide' button in the query results section. Implement click handlers that trigger the export endpoints and initiate browser downloads. Style buttons to match the existing UI design patterns and ensure they're visually consistent with other interface elements.

## Step by Step Tasks

### Create E2E Test Specification
- Read `.claude/commands/test_e2e.md` to understand the E2E test format and execution process
- Read `.claude/commands/e2e/test_basic_query.md` to see an example of an E2E test file structure
- Create `.claude/commands/e2e/test_one_click_table_exports.md` with detailed test steps to validate:
  - Upload sample data to create a table
  - Verify download button appears next to 'x' icon in Available Tables section
  - Click download button for a table and verify CSV file is downloaded
  - Execute a natural language query to generate results
  - Verify download button appears next to 'Hide' button in query results section
  - Click download button for query results and verify CSV file is downloaded
  - Verify CSV files contain correct data with proper headers and formatting
  - Include success criteria and screenshot requirements

### Implement Backend Export Endpoints
- Open `app/server/server.py` and add two new endpoint functions
- Implement `GET /api/table/{table_name}/export` endpoint:
  - Use `@app.get("/api/table/{table_name}/export")` decorator
  - Validate `table_name` parameter using `validate_identifier()` from `sql_security`
  - Check table exists using `check_table_exists()` from `sql_security`
  - Execute `SELECT * FROM {table}` using `execute_query_safely()` with identifier params
  - Convert results to CSV format using Python's `csv` module (`csv.DictWriter`)
  - Set HTTP headers: `Content-Type: text/csv` and `Content-Disposition: attachment; filename="{table_name}.csv"`
  - Return CSV data as response using `fastapi.responses.Response`
  - Add error handling for missing tables (404) and SQL errors (500)
  - Add logging for successful exports and errors
- Implement `POST /api/query/export` endpoint:
  - Accept request body with `sql: str` and `filename: Optional[str]` fields
  - Validate SQL query using existing security checks (ensure it's a SELECT query)
  - Execute SQL using `execute_sql_safely()` from `sql_processor`
  - Convert results to CSV format using Python's `csv` module
  - Generate filename (use provided filename or default to `query_results_{timestamp}.csv`)
  - Set same HTTP headers as table export endpoint
  - Return CSV data as response
  - Add error handling and logging
- Add necessary imports: `from fastapi.responses import Response`, `import csv`, `from io import StringIO`
- Test endpoints manually to ensure they work correctly

### Update Backend Data Models (if needed)
- Open `app/server/core/data_models.py`
- Evaluate if new Pydantic models are needed:
  - If using POST for query export, add `QueryExportRequest` model with `sql: str` and `filename: Optional[str]` fields
  - If needed, add response models (though plain CSV response may not require Pydantic model)
- Skip this step if models aren't needed (e.g., using GET with path params only)

### Add Download Buttons to Available Tables Section
- Open `app/client/src/main.ts`
- Locate the `displayTables()` function (around line 254)
- In the table header creation section (around line 269-295), add download button creation:
  - Create button element: `const downloadButton = document.createElement('button')`
  - Set class: `downloadButton.className = 'download-table-button'`
  - Set icon content: `downloadButton.innerHTML = '📥'` (or use appropriate Unicode/emoji)
  - Set title: `downloadButton.title = 'Download table as CSV'`
  - Add click handler: `downloadButton.onclick = () => downloadTable(table.name)`
  - Insert button into `tableHeader` before `removeButton` so it appears to the left of the 'x' icon
- Create new `downloadTable(tableName: string)` function:
  - Use `window.location.href = \`/api/table/${tableName}/export\`` to trigger download
  - Or use `fetch()` with blob handling for more control
  - Add error handling with `displayError()` for failed downloads
- Test in browser to ensure button appears and positioning is correct

### Add Download Buttons to Query Results Section
- Open `app/client/src/main.ts`
- Locate the `displayResults()` function (around line 184)
- Find the results header section where the 'Hide' toggle button is rendered (around line 214)
- Modify the results header HTML structure to include a download button:
  - Store the current SQL and results data in a way that the download button can access it
  - Add download button to the `.results-header` div before the toggle button
  - Create button with class `download-results-button`, icon '📥', and title 'Download results as CSV'
- Create new `downloadQueryResults(sql: string, results: Record<string, any>[], columns: string[])` function:
  - Convert results to CSV format client-side OR
  - Send POST request to `/api/query/export` endpoint with SQL query
  - Trigger browser download using blob URL or direct endpoint call
  - Add error handling
- Update the results display to wire up the download button with the current query data
- Test in browser to ensure button appears next to 'Hide' button and downloads work

### Update Client API Module
- Open `app/client/src/api/client.ts`
- Evaluate if new API methods are needed:
  - For table export: may not need API method if using direct `window.location.href`
  - For query export: add `exportQuery(sql: string, filename?: string)` method if using API approach
- If adding methods, follow existing pattern with `apiRequest<T>()` helper
- Update TypeScript types in `src/types.d.ts` if new request/response types are needed
- Skip this step if not using centralized API methods for exports

### Style Download Buttons
- Open `app/client/src/style.css`
- Add CSS classes for download buttons:
  - `.download-table-button` - Style similar to `.remove-table-button` (around line 289)
  - `.download-results-button` - Style for results section download button
- Both buttons should:
  - Use `background: none` and `border: none` for icon-only buttons
  - Set appropriate `font-size` (e.g., 1.2rem) for icon visibility
  - Set `color` to match existing button colors (e.g., `var(--text-secondary)`)
  - Set `cursor: pointer` for interactivity indication
  - Add `transition: all 0.2s` for smooth hover effects
  - Add `:hover` state with background color change (e.g., `background: rgba(102, 126, 234, 0.1)`) and color change to `var(--primary-color)`
  - Set `padding` for clickable area (e.g., 0.5rem)
  - Set `border-radius` for hover effect (e.g., 4px)
- Ensure buttons are visually consistent with existing UI elements
- Test hover and click states in browser

### Update TypeScript Types
- Open `app/client/src/types.d.ts`
- Add any new TypeScript interfaces needed for export functionality:
  - If using API methods: `interface QueryExportRequest { sql: string; filename?: string; }`
  - Add to global types if needed
- Ensure all new client-side code has proper type annotations
- Skip this step if no new types are required

### Test Export Functionality Manually
- Start the application using `./scripts/start.sh`
- Upload sample data (users.json) via the UI
- Verify download button appears next to 'x' icon in Available Tables section
- Click download button and verify:
  - CSV file downloads with correct filename (e.g., `users.csv`)
  - File contains all table data with proper headers
  - Data is properly formatted with commas, quotes, and escaping
- Execute a natural language query (e.g., "Show me all users")
- Verify download button appears next to 'Hide' button in query results
- Click download button and verify:
  - CSV file downloads with appropriate filename
  - File contains query results with correct headers
  - Data matches what's displayed in the UI table
- Test edge cases:
  - Empty tables (should download CSV with headers only)
  - Large tables (should download without errors)
  - Tables with special characters in data (should properly escape)
  - Query results with null values (should handle gracefully)

### Run Validation Commands
- Execute all validation commands listed in the "Validation Commands" section below
- Ensure zero errors and zero regressions
- Fix any issues that arise before marking the feature as complete

## Testing Strategy

### Unit Tests
- Test CSV generation logic in isolation:
  - Create unit test for converting list of dictionaries to CSV string
  - Test with empty data (should return headers only)
  - Test with single row, multiple rows
  - Test with special characters (commas, quotes, newlines) in data
  - Test with null/None values in data
- Test endpoint validation:
  - Test table name validation (reject SQL injection attempts)
  - Test query validation (ensure only SELECT queries are allowed)
  - Test table existence checking
- Test error handling:
  - Test response when table doesn't exist (should return 404)
  - Test response when SQL execution fails (should return 500)
  - Test response when invalid table name is provided (should return 400)

### Integration Tests
- Test full export workflow:
  - Upload sample data, then export table via endpoint
  - Execute query, then export results via endpoint
  - Verify CSV content matches database content
  - Verify HTTP headers are set correctly
  - Verify filename in Content-Disposition header is correct

### Edge Cases
- **Empty tables**: Export should succeed with headers but no data rows
- **Tables with null values**: CSV should represent nulls as empty fields
- **Tables with special characters**: Data containing commas, quotes, newlines should be properly escaped in CSV
- **Tables with Unicode characters**: Should handle international characters and emojis correctly
- **Large tables**: Should handle tables with thousands of rows without memory issues or timeouts
- **Query results with complex data types**: Should handle dates, numbers, booleans, etc. correctly
- **Concurrent export requests**: Multiple users downloading simultaneously shouldn't cause issues
- **Invalid table names**: Should reject SQL injection attempts (e.g., `users; DROP TABLE users--`)
- **Malicious SQL queries**: Should reject DELETE, UPDATE, DROP, etc. in query export
- **Long filenames**: Should handle table names that create very long filenames

## Acceptance Criteria
- Download button appears to the left of 'x' icon in Available Tables section for each table
- Download button appears to the left of 'Hide' button in Query Results section
- Clicking download button for a table triggers immediate CSV file download
- Downloaded CSV file has filename matching pattern `{table_name}.csv`
- CSV file contains all rows from the table with proper column headers
- Clicking download button for query results triggers immediate CSV file download
- Downloaded query results CSV file has descriptive filename (e.g., `query_results_TIMESTAMP.csv` or based on query)
- CSV files are properly formatted with quoted fields, escaped special characters, and correct delimiters
- Export functionality works for tables with 1 row, 100 rows, and 1000+ rows
- Export functionality handles null values, special characters, and Unicode correctly
- Backend endpoints validate table names and SQL queries for security
- Backend endpoints return 404 for non-existent tables
- Backend endpoints return 400 for invalid table names or queries
- Backend endpoints return 500 for SQL execution errors with appropriate error messages
- UI buttons are styled consistently with existing interface elements
- Buttons have hover effects and visual feedback
- No regressions: all existing functionality (upload, query, delete table) continues to work
- All server tests pass: `cd app/server && uv run pytest`
- All frontend type checks pass: `cd app/client && bun tsc --noEmit`
- Frontend build succeeds: `cd app/client && bun run build`
- E2E test passes with screenshots demonstrating the feature works as expected

## Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

- Read `.claude/commands/test_e2e.md`, then read and execute the new E2E test file `.claude/commands/e2e/test_one_click_table_exports.md` to validate this functionality works end-to-end with browser automation
- `cd app/server && uv run pytest` - Run server tests to validate the feature works with zero regressions
- `cd app/client && bun tsc --noEmit` - Run frontend type checks to validate TypeScript code is correct
- `cd app/client && bun run build` - Run frontend build to validate the feature works with zero regressions
- Manual validation:
  - Start app with `./scripts/start.sh`
  - Upload sample data (users.json)
  - Click download button for 'users' table in Available Tables section
  - Verify `users.csv` downloads and contains correct data
  - Execute query "Show me all users"
  - Click download button in Query Results section
  - Verify CSV downloads and contains query results
  - Test with products.csv and events.jsonl sample data
  - Verify all existing features (upload, query, delete) still work

## Notes
- Use Python's built-in `csv` module for CSV generation - no additional dependencies required
- Consider using `csv.DictWriter` for converting list of dictionaries to CSV format
- HTTP headers for file download: `Content-Type: text/csv; charset=utf-8` and `Content-Disposition: attachment; filename="{filename}.csv"`
- For query results export, consider whether to use GET with SQL in query param (URL length limits) or POST with SQL in body (more flexible, recommended)
- Client-side download can be triggered via direct `window.location.href` assignment or by fetching blob and creating object URL - direct assignment is simpler for this use case
- Ensure CSV files use UTF-8 encoding to handle international characters
- Consider adding optional query parameter for export format (e.g., `?format=csv`) if future formats like JSON are planned, but CSV is sufficient for MVP
- The download icon '📥' is a Unicode emoji that should render consistently across browsers, but consider using an SVG icon library if more design control is needed
- Future enhancement: add option to export as JSON, Excel, or with custom delimiters (TSV)
- Future enhancement: add option to export only selected columns or filtered rows
- Future enhancement: add progress indicator for large table exports
