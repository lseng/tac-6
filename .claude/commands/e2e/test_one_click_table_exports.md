# E2E Test: One Click Table Exports

Test the one-click CSV export functionality for both available tables and query results in the Natural Language SQL Interface application.

## User Story

As a user
I want to export tables and query results as CSV files with one click
So that I can use the data in external tools like Excel, Google Sheets, or other data analysis applications without manual copy-pasting

## Test Steps

1. Navigate to the `Application URL`
2. Take a screenshot of the initial state
3. **Verify** the page title is "Natural Language SQL Interface"
4. **Verify** core UI elements are present:
   - Query input textbox
   - Query button
   - Upload Data button
   - Available Tables section

5. Upload sample data file (users.json) via the Upload Data button
6. Wait for the upload to complete
7. Take a screenshot of the Available Tables section after upload
8. **Verify** the 'users' table appears in the Available Tables section
9. **Verify** a download button (📥) appears to the left of the 'x' icon for the users table

10. Click the download button for the users table
11. Wait for download to complete (check browser download notifications or file system)
12. **Verify** a file named 'users.csv' was downloaded
13. Take a screenshot showing the download button interaction

14. Enter the query: "Show me all users"
15. Click the Query button
16. Wait for query results to appear
17. Take a screenshot of the query results
18. **Verify** the results table contains data
19. **Verify** a download button (📥) appears in the results header to the left of the 'Hide' button

20. Click the download button for query results
21. Wait for download to complete
22. **Verify** a CSV file for query results was downloaded (filename should contain 'query' or 'results')
23. Take a screenshot showing the results download button interaction

24. Click the Hide button to close results
25. Take a screenshot of the final state

## Success Criteria
- Upload functionality creates a table successfully
- Download button appears next to 'x' icon in Available Tables section
- Download button appears next to 'Hide' button in Query Results section
- Clicking table download button triggers CSV file download
- Downloaded table CSV file has correct filename (users.csv)
- Clicking results download button triggers CSV file download
- Downloaded results CSV file has appropriate filename
- All download buttons are styled consistently with the UI
- No JavaScript errors in console
- 6 screenshots are taken showing the complete workflow
