# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Bucket List Manager** - A vanilla JavaScript web application for managing personal goals/bucket lists. Features include CRUD operations, real-time statistics, filtering, and persistent storage via LocalStorage.

This is a **static frontend-only project** with no build tools, tests, or backend dependencies.

## Architecture

The application follows a **2-layer separation**:

### Data Layer: `js/storage.js`
- **BucketStorage** object: Pure data management using LocalStorage
- Responsibilities: CRUD operations, statistics calculation, filtering logic
- Key methods: `load()`, `save()`, `addItem()`, `updateItem()`, `deleteItem()`, `toggleComplete()`, `getStats()`, `getFilteredList()`
- No DOM manipulation - fully testable in isolation
- Error handling: All methods wrap localStorage operations in try-catch blocks

### Presentation Layer: `js/app.js`
- **BucketListApp** class: Manages UI rendering and user interactions
- Responsibilities: DOM caching, event handling, modal management, HTML generation
- Pattern: Caches DOM elements in constructor for performance, re-renders entire list on state changes
- Important: Uses `escapeHtml()` to prevent XSS vulnerabilities
- Modal management: `openEditModal()` and `closeEditModal()` control edit form visibility

## Key Architectural Patterns

1. **Data persistence**: All changes flow through `BucketStorage.save()` which persists to LocalStorage. No data exists in memory between page loads.

2. **Re-render on change**: After any data modification, `BucketListApp.render()` is called to update the entire UI. This is simple but not optimized for large datasets.

3. **Event delegation**: Events are bound directly to specific elements (filter buttons, form submit). Modal close can also be triggered by clicking the backdrop.

4. **Date handling**: Uses ISO strings in storage (`toISOString()`), displays using Korean locale (`toLocaleDateString('ko-KR')`).

5. **Filter state**: Current filter is tracked in `app.currentFilter` and affects what `render()` displays via `getFilteredList()`.

## Data Structure

Each bucket item stored in LocalStorage:
```javascript
{
  id: "timestamp",           // Used for key lookup
  title: "string",
  completed: boolean,
  createdAt: "ISO-8601",
  completedAt: "ISO-8601 or null"
}
```

All items stored as array in single LocalStorage key: `"bucketList"`

## Running the Application

**Option 1: Direct file open**
- Double-click `index.html` in file explorer

**Option 2: Python HTTP server**
```bash
python -m http.server 8000
# Visit http://localhost:8000
```

**Option 3: VS Code Live Server**
- Right-click `index.html` → "Open with Live Server"

No build step required. All CSS is Tailwind CDN + custom styles.css.

## File Structure Guide

```
index.html          - Single page with form, list container, modal, statistics
css/styles.css      - Custom animations, responsive adjustments, dark mode support
js/storage.js       - Data management layer (stateless, single object)
js/app.js           - UI layer (stateful class instance called 'app')
```

## UI/Styling

- **CSS Framework**: Tailwind CSS (CDN) + custom CSS in `styles.css`
- **Animations**: `slideIn` (new items), `fadeIn` (modals/empty state), `scaleIn` (modal content)
- **Responsive breakpoints**: 640px (mobile), 768px (tablet), 1024px (desktop)
- **Dark mode**: Supported via `@media (prefers-color-scheme: dark)` in styles.css
- **Color scheme**: Blue (#3b82f6) primary, Green (#10b981) success, Red (#ef4444) error

## Common Development Tasks

**Adding a new feature:**
1. Determine if it's data logic (add to `BucketStorage`) or UI logic (add to `BucketListApp`)
2. Maintain the separation: Storage has no DOM references, App doesn't modify data directly
3. Call `this.render()` after any data changes to update the UI

**Modifying the UI:**
- Update `createBucketItemHTML()` to change item appearance
- Update HTML in `index.html` for layout changes
- Tailwind classes are preferred over custom CSS for rapid iteration

**Adding data persistence features:**
- All changes must flow through `BucketStorage.save()` to persist
- Remember: After modifying data, call `app.render()` to refresh the view

**Handling user input:**
- Form validation happens in event handlers (empty string checks)
- Use `alert()` for simple errors (current pattern)
- Modal state is managed through `editingId` and modal class manipulation

## Security Notes

- **XSS prevention**: The `escapeHtml()` method is critical. Use it when displaying user input in HTML.
- **Currently used in**: Item title display and in modal editing
- **Risk areas**: If adding new user input fields, ensure they're escaped before being inserted into DOM via `innerHTML`

## Browser Compatibility

- Modern browsers only (Chrome, Firefox, Safari, Edge)
- Requires: ES6 support, LocalStorage API, CSS Grid/Flexbox
- No IE11 support

## Important Implementation Details

- **ID generation**: Uses `Date.now().toString()` - not UUID, sufficient for this use case
- **Focus management**: Input focus restored after adding item (`bucketInput.focus()`)
- **Modal backdrop**: Clicking outside modal (on backdrop) closes it via event target check
- **Empty state**: Toggle `#emptyState` visibility based on whether list has items
- **Active filter styling**: Filter buttons have `.active` class toggled on click

## Code Style Notes

- Korean comments throughout
- Object-style module for storage (not class-based)
- Class-based approach for app logic
- Inline onclick handlers for item actions (generated in `createBucketItemHTML()`)
- No external dependencies beyond Tailwind CDN
