# Patch: Change Query Results Download from Emoji to Text Button

## Metadata
adw_id: `1f259f43`
review_change_request: `For Query Results, make sure the download button is just to the left of our 'hide' button. Please make the Download button an actual button that says 'Download', not just an emoji`

## Issue Summary
**Original Spec:** specs/issue-1-adw-1f259f43-sdlc_planner-one-click-table-exports.md
**Issue:** The download button in the Query Results section is currently displaying as an emoji (📥) icon, but the requirement is for it to be an actual button with the text "Download"
**Solution:** Update the download button in the Query Results section to display the text "Download" instead of the emoji icon, while keeping the positioning to the left of the 'Hide' button

## Files to Modify
Use these files to implement the patch:

- `app/client/src/main.ts` - Update the download button innerHTML from emoji to text "Download"
- `app/client/src/style.css` - Update the `.download-results-button` styling to accommodate text instead of emoji icon

## Implementation Steps
IMPORTANT: Execute every step in order, top to bottom.

### Step 1: Update Query Results Download Button to Display Text
- Open `app/client/src/main.ts`
- Locate the `displayResults()` function around line 214 where the download button is created
- Change `downloadButton.innerHTML = '📥';` to `downloadButton.textContent = 'Download';`
- Update the title attribute if needed to remain consistent

### Step 2: Update CSS Styling for Text-Based Button
- Open `app/client/src/style.css`
- Locate the `.download-results-button` class definition
- Update the styling to work with text instead of emoji:
  - Change `font-size` from `1.2rem` to standard button font size (or remove it to inherit)
  - Update `width` and `height` to `auto` or appropriate dimensions for text button
  - Add `padding` suitable for text button (e.g., `0.5rem 1rem`)
  - Ensure the button has proper visual styling (background, border, etc.) to look like an actual button
  - Style it to be consistent with other text buttons in the UI (like the "Hide" button)

## Validation
Execute every command to validate the patch is complete with zero regressions.

- `cd app/client && bun tsc --noEmit` - Verify TypeScript compilation succeeds
- `cd app/client && bun run build` - Verify frontend build succeeds
- `cd app/server && uv run pytest` - Verify all server tests pass
- Manual validation:
  - Start app with `./scripts/start.sh`
  - Upload users.json sample data
  - Execute query "Show me all users"
  - Verify download button in Query Results section displays "Download" text (not emoji)
  - Verify button is positioned to the left of the "Hide" button
  - Verify clicking the button successfully downloads a CSV file
  - Verify button styling is consistent with other UI buttons
- Read `.claude/commands/test_e2e.md`, then read and execute `.claude/commands/e2e/test_one_click_table_exports.md` to validate the feature still works end-to-end

## Patch Scope
**Lines of code to change:** 2-10 lines (1 line in main.ts, several lines in style.css)
**Risk level:** low
**Testing required:** Frontend type check, build validation, manual UI testing, E2E test execution
