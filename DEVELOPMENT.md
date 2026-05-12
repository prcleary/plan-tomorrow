# Development Context for Plan Tomorrow

## Project Overview

**Plan Tomorrow** is a single-page web application for daily planning with weighted randomization and drag-and-drop scheduling. Built entirely with vanilla JavaScript, semantic HTML, and CSS Grid - no frameworks or build tools required.

**Live App:** https://prcleary.github.io/plan-tomorrow/  
**Repository:** https://github.com/prcleary/plan-tomorrow  
**Development:** Created with Claude Sonnet 4.5 via GitHub Copilot in VS Code

## Architecture

### Single-File Application
- **index.html** - Complete application (HTML + CSS + JavaScript)
- **README.md** - User-facing documentation
- **LICENSE** - MIT license
- **DEVELOPMENT.md** - This file (developer context)

### Technology Stack
- Pure HTML5, CSS3, JavaScript (ES6)
- No frameworks, no build process, no dependencies
- GitHub Pages for static hosting
- Works offline after initial load

### State Management
All application state stored in a single `state` object:
```javascript
{
    startTime: '',           // ISO datetime string
    habits: [],              // Array of strings
    projects: [],            // Array of strings  
    tasks: [],               // Array of strings
    emails: [],              // Array of strings
    calendar: '',            // Calendar syntax string (e.g., "f8m2f4")
    randomisedItems: [],     // Array of {type, text, weight}
    calendarSlots: [],       // Array of {type, time, items[]}
    placedItems: {},         // Map of slotId -> items
    beforeItems: [],         // Array of {type, text} for Before section
    afterItems: []           // Array of {type, text} for After section
}
```

State is automatically persisted to `localStorage` under key `planTomorrowState` and restored on page load.

## Key Features & Implementation

### 1. Weighted Randomization Algorithm

**Location:** `getWeightedRandomSample()` function

**Weighting System:**
- **Habits:** Equal weight (1/n where n = number of habits)
- **Projects:** 0.9 (first) down to 0.5 (minimum), weighted by position
- **Tasks:** 0.95 (first) down to 0.05 (minimum), weighted by position  
- **Emails:** 0.95 (first) down to 0.05 (minimum), weighted by position

**Selection Method:**
- Cumulative probability distribution
- Random value 0-1 compared against cumulative weights
- Returns up to 50 items in random order (NOT sorted by weight)

**Visual Indicators:**
- Opacity: 0.6 (low priority) to 1.0 (high priority)
- Percentage badge: 0-100 showing relative probability
- Tooltips on hover explaining the weighting (disabled on mobile)

### 2. Calendar Syntax Parser

**Location:** `parseCalendar()` function

**Syntax:**
- `f` = free half-hour block
- `m` = meeting half-hour block
- Numbers multiply preceding character (multi-digit support)

**Examples:**
- `fmfm` = 2 hours (4 slots)
- `f16` = 8 hours (16 slots)
- `m4f12m2` = 2hr meeting + 6hr free + 1hr meeting

**Implementation:**
- Character-by-character parsing
- Lookahead for multi-digit numbers
- Validates and expands to array of 'free'/'meeting' strings

### 3. Auto-Project Placement

**Location:** `autoPlaceProjects()` function

**Algorithm:**
1. Find continuous free blocks of 2+ hours (4+ half-hour slots)
2. For each 2-hour chunk (4 slots), randomly select ONE project (weighted)
3. Projects can repeat across different chunks
4. Placement is random within available chunks

**Visual:**
- Projects appear as orange lozenges
- Auto-placed in calendar slots after generation
- Can be manually moved or removed

### 4. Drag-and-Drop System

**Three Interaction Methods:**

**Mouse (Desktop):**
- Uses native HTML5 Drag and Drop API
- `handleDragStart()`, `handleDrop()`, `handleDragOver()`
- Drop target highlighting with `.drag-over` class

**Touch (Mobile):**
- Custom touch event handlers
- `handleTouchStart()`, `handleTouchMove()`, `handleTouchEnd()`
- Visual clone follows finger during drag
- Swipe gesture detection with 10px threshold

**Keyboard (Accessibility):**
- Tab to navigate, Space to pick/place
- Arrow Up/Down to select destination slot
- Escape to cancel
- Visual focus indicators with outline

### 5. Before/After Sections

**Purpose:** Unlimited capacity sections for overflow items

**Implementation:**
- `.special-slot` CSS class with green dashed border
- No 4-item limit (unlike regular free slots)
- Separate drop handler: `handleSpecialDrop()`
- Included in markdown export

**Visual Distinction:**
- Green border (vs. gray for free slots)
- Bold green label text
- Min-height 70px (vs. 50px for regular slots)

### 6. Calendar Slots

**Slot Types:**
1. **Meeting slots:** Gray background, non-droppable, shows "Meetings schmeetings"
2. **Free slots:** Dark background, max 4 items, droppable
3. **Special slots (Before/After):** Green border, unlimited items

**Time Display:**
- 12-hour format with AM/PM
- Each slot represents 30 minutes
- Slots start from user-defined startTime

### 7. Markdown Export

**Location:** `exportData()` function

**Format:**
```markdown
# YYYY-MM-DD DayName note

## Before
- [ ] item1
- [ ] item2

## HH:MM AM/PM
- [ ] scheduled item

## HH:MM AM/PM - HH:MM AM/PM
Meetings schmeetings

## After
- [ ] item1
- [ ] item2
```

**Features:**
- Date heading with day name
- Consolidated meeting time ranges (e.g., "10:30 AM - 11:30 AM")
- Before section at top, After section at bottom
- Task checkbox format `- [ ]` for all items
- Copies to clipboard via `navigator.clipboard.writeText()`

### 8. Remove Button (X)

**Purpose:** Allow users to permanently remove items from Randomised list or Calendar without re-randomizing

**Implementation:**

**Visual Design:**
- Small circular button with × symbol (Unicode ×)
- Positioned on the right side of each lozenge
- Semi-transparent background (rgba(0,0,0,0.3))
- Red on hover (rgba(231, 76, 60, 0.8))
- 14px × 14px size, centered vertically

**CSS Classes:**
- `.remove-button` - Styling for the X button
- Absolute positioning within `.lozenge`
- Lozenge padding-right adjusted to 22px to make room

**Function:** `removeItem(id)`
- Handles removal from both randomised list and calendar
- For `random-` prefixed IDs: Removes from `state.randomisedItems` array
- For `placed-` or `project-` prefixed IDs: Removes from all calendar slots and Before/After sections
- Calls appropriate render function (`renderRandomised()` or `renderCalendar()`)
- Saves state to localStorage

**Behavior:**
- Click handler prevents event propagation (doesn't trigger drag)
- Item is permanently removed until next randomization
- No undo capability (user must re-run randomization to get items back)

**Code Locations:**
- CSS: Lines ~402-440
- Button creation in `createLozenge()`: Lines ~1188-1196
- `removeItem()` function: Lines ~1227-1265

## Critical Implementation Details

### Start Time Defaulting Issue (FIXED)

**Problem:** Start Time field not defaulting to tomorrow at 9am on page load or Clear Data.

**Root Cause:** 
- `form.reset()` was clearing the datetime-local input
- `setDefaultStartTime()` was only updating input field, not `state.startTime`
- Empty `state.startTime` was persisted to localStorage
- On reload, empty value overrode the default

**Solution (3 fixes applied):**
1. `setDefaultStartTime()` now updates BOTH input field AND `state.startTime`
2. `restoreInputs()` checks if `state.startTime` is empty and calls `setDefaultStartTime()`
3. `clearData()` calls `setDefaultStartTime()` BEFORE clearing other fields (not after form.reset())
4. `loadState()` saves corrected startTime back to localStorage after fixing

**Code Locations:**
- `setDefaultStartTime()` - Line ~750
- `restoreInputs()` - Line ~770
- `clearData()` - Line ~1690
- `loadState()` - Line ~710

**IMPORTANT:** Never use `form.reset()` before setting datetime-local values. The reset may clear the field asynchronously.

### Syntax Error in handleDragOver (FIXED)

**Problem:** Entire JavaScript failing to load due to syntax error on line 1370.

**Error:** `if (e.preventDefault) { || slot.classList.contains('special-slot')`

**Fix:** Moved special-slot check to proper location:
```javascript
if (e.preventDefault) {
    e.preventDefault();
}
const slot = e.currentTarget;
if (slot.classList.contains('free') || slot.classList.contains('special-slot')) {
    slot.classList.add('drag-over');
}
```

**Lesson:** Always validate syntax with `get_errors` tool before committing.

## File Structure

### HTML Structure
```
<!DOCTYPE html>
<html>
  <head>
    - Meta tags (charset, viewport, description, keywords)
    - Title
    - <style> (lines 26-614)
  </head>
  <body>
    - <header> with title and source link
    - <div class="container"> with three columns:
      1. Input column (form)
      2. Randomised column (generated items)
      3. Calendar column (time slots with Before/After)
    - <script> (lines 715-1930)
  </body>
</html>
```

### CSS Organization (with section headers)
1. Global Reset & Base Styles
2. Header Styles
3. Layout & Grid System
4. Column Styles
5. Form Styles
6. Buttons
7. Lozenges (Draggable Items)
8. Calendar Time Slots
9. Media Queries & Responsive Design

### JavaScript Structure
1. State management (single state object)
2. localStorage functions (load/save)
3. Utility functions (parseCalendar, formatTime, etc.)
4. Randomization (getWeightedRandomSample)
5. Main run() function
6. Auto-project placement
7. Rendering (renderRandomised, renderCalendar)
8. Lozenge creation with tooltips
9. Drag-and-drop handlers (mouse)
10. Touch event handlers (mobile)
11. Keyboard navigation handlers
12. Export/print functions
13. Event listeners setup
14. Initialization

## Development Workflow

### Local Development
1. Edit `index.html` directly
2. Test in browser (file:// protocol works)
3. Check for errors with F12 Developer Console
4. Validate with VSCode's built-in HTML/JS linting

### Git Workflow
```bash
git add .
git commit -m "Description of changes"
git push origin main
```

### GitHub Pages Deployment
- Automatic deployment on push to main branch
- Usually takes 2-3 minutes to deploy
- Access at: https://prcleary.github.io/plan-tomorrow/

### Testing Checklist
- [ ] Start Time defaults to tomorrow 9am on first load
- [ ] Clear Data resets Start Time to tomorrow 9am
- [ ] Page refresh preserves data (localStorage)
- [ ] Drag-and-drop works (mouse, touch, keyboard)
- [ ] Before/After sections accept unlimited items
- [ ] Regular slots limited to 4 items
- [ ] Markdown export includes Before/After
- [ ] Print layout shows only calendar column
- [ ] No console errors (F12)

## Browser Compatibility

**Tested Browsers:**
- Chrome/Edge (Chromium)
- Firefox
- Safari

**Required Features:**
- localStorage API
- HTML5 Drag and Drop API
- Touch Events API
- CSS Grid
- ES6 JavaScript (arrow functions, const/let, template literals)
- Clipboard API (navigator.clipboard)

**Mobile Considerations:**
- Tooltips disabled on touch devices (media query)
- Touch event handlers for drag-and-drop
- Responsive layout with CSS Grid
- Viewport meta tag for proper scaling

## Code Conventions

### Naming
- camelCase for variables and functions
- kebab-case for CSS classes
- UPPERCASE for constants (rare)
- Descriptive names (e.g., `handleDragStart`, not `hds`)

### Comments
- Function-level JSDoc-style comments
- Section headers in CSS
- Inline comments for complex logic
- TODO comments for future improvements

### Formatting
- 4-space indentation
- Single quotes for JavaScript strings
- Double quotes for HTML attributes
- No semicolons required but used consistently

### State Updates
- Always call `saveState()` after modifying state
- Never mutate state directly in event handlers
- Validate state before rendering

## Known Limitations

1. **No undo/redo:** Changes are immediately persisted
2. **Single day only:** No multi-day planning
3. **No recurring tasks:** Each day is independent
4. **No time estimates:** Everything is 30-minute blocks
5. **No collaboration:** Purely local, single-user
6. **No cloud sync:** localStorage only
7. **Browser-specific:** Data doesn't sync between browsers
8. **No mobile app:** Web-only, no native app

## Future Enhancement Ideas

- Add undo/redo functionality
- Multi-day planning view
- Recurring task templates
- Time estimates and automatic fitting
- Export to other formats (CSV, iCal)
- Import from other tools
- Customizable time block durations
- Color themes beyond dark mode
- Statistics/analytics on task completion
- Cloud backup option (optional)

## Performance Considerations

- **Rendering:** Virtual scrolling not needed (max 50 items + ~48 time slots)
- **State size:** Minimal (~10KB typical), localStorage has 5-10MB limit
- **Event listeners:** Properly cleaned up on element removal
- **Memory leaks:** None identified, but watch for detached DOM nodes
- **Animation:** CSS-only animations, no JavaScript animation loops

## Security Considerations

- **No server:** All data stays in browser
- **No authentication:** Purely local application
- **XSS:** Not applicable (no user-generated HTML)
- **CORS:** Not applicable (no external requests)
- **localStorage:** Limited to same-origin policy

## Accessibility Features

- Semantic HTML elements (`<header>`, `<form>`, `<button>`)
- Proper label associations (`for` attributes)
- Keyboard navigation (Tab, Space, Arrow keys, Escape)
- Focus indicators (outline on focus)
- ARIA attributes where appropriate
- Sufficient color contrast (dark mode)
- Screen reader compatible structure

## Common Pitfall Warnings

### ⚠️ DateTime-Local Input Quirks
- Don't use `form.reset()` then set value immediately
- Always update `state.startTime` when setting input value
- ISO format required: `YYYY-MM-DDTHH:mm`
- Be aware of timezone issues with `toISOString()` vs local time

### ⚠️ localStorage Edge Cases
- Check for empty strings vs null vs undefined
- Parse JSON with try-catch for corruption handling
- Remember Date objects don't serialize (convert to strings)
- Test with corrupted localStorage (manual edit in DevTools)

### ⚠️ Drag-and-Drop Gotchas
- Must call `preventDefault()` in dragover handler
- dataTransfer only available in dragstart/drop (not dragover)
- Touch events need separate implementation
- Event propagation can cause double-triggers

### ⚠️ CSS Specificity Issues
- Avoid `!important` unless absolutely necessary
- Use class selectors over ID selectors
- Media queries override earlier rules (mobile-first approach)
- Pseudo-selectors have same specificity as classes

## Debugging Tips

### Common Issues & Solutions

**"Nothing happens when I click Run"**
- Check browser console for JavaScript errors
- Verify form validation (required fields)
- Check if event listeners are attached

**"Drag-and-drop not working"**
- Check if draggedData is set correctly
- Verify drop target has dragover handler
- Look for event.stopPropagation() conflicts

**"localStorage not persisting"**
- Check if saveState() is being called
- Verify localStorage isn't disabled (private browsing)
- Look for JSON parsing errors

**"Layout broken on mobile"**
- Check CSS Grid media queries
- Verify viewport meta tag is present
- Test with browser DevTools mobile emulation

### Developer Console Commands

```javascript
// View current state
console.log(JSON.stringify(state, null, 2));

// Clear localStorage
localStorage.removeItem('planTomorrowState');

// Manually set state
state.startTime = '2026-05-13T09:00';
saveState();

// Check for corrupted data
try { JSON.parse(localStorage.getItem('planTomorrowState')); } 
catch(e) { console.error('Corrupted:', e); }
```

## Version History Notes

- Initial development completed in single session
- Multiple iterations on Start Time defaulting issue (regression introduced, now fixed)
- Before/After sections added after initial release
- Syntax error in handleDragOver fixed (line 1370)
- Documentation improvements and code organization (section headers)
- Roboto font integration via Google Fonts
- Timezone fix for Start Time (replaced UTC toISOString with local formatting)
- Remove button (X) added to all lozenges for permanent item deletion

## Contributing Guidelines

If extending this project:

1. **Maintain single-file architecture:** Don't split into multiple files
2. **No frameworks:** Keep it vanilla JavaScript
3. **Test cross-browser:** Chrome, Firefox, Safari minimum
4. **Update documentation:** Both README and this file
5. **Follow conventions:** Match existing code style
6. **Git messages:** Clear, descriptive commit messages
7. **Test thoroughly:** All interaction methods (mouse, touch, keyboard)

## Contact & Support

- GitHub Issues: https://github.com/prcleary/plan-tomorrow/issues
- Repository: https://github.com/prcleary/plan-tomorrow
- Developer: @prcleary

---

**Last Updated:** May 12, 2026  
**File Version:** 1.0  
**Application Version:** 1.0 (commit d9ce104)
