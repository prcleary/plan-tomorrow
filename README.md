# Plan Tomorrow

A mobile-friendly single-page web application for planning your next day with intelligent task prioritization and randomization.

**Live App:** [https://prcleary.github.io/plan-tomorrow/](https://prcleary.github.io/plan-tomorrow/)

## Features

- **Smart Randomization**: Generates a weighted random sample of your tasks, emails, and habits based on priority
- **Auto-Project Placement**: Automatically schedules one randomly-selected project for each 2-hour chunk in continuous free blocks
- **Before/After Sections**: Unlimited capacity sections for items that don't fit in your scheduled day
- **Weight Indicators**: Visual opacity and percentage indicators show relative priority of items
- **Interactive Tooltips**: Hover to see probability weighting explanations for each item
- **Drag-and-Drop**: Intuitive mouse/touch interface for arranging items in your calendar
- **Touch/Swipe Support**: Full mobile support with swipe gestures for drag-and-drop
- **Keyboard Navigation**: Complete keyboard accessibility with Tab/Space/Arrow keys
- **Local Storage**: All data persists locally in your browser - works offline after first load
- **Dark Mode**: Easy on the eyes with a modern dark interface
- **Copy Markdown**: Export your schedule as formatted markdown with task checkboxes
- **Print Ready**: Print your completed calendar to PDF or paper

## How to Use

### 1. Set Your Start Time
- Default is 9:00 AM tomorrow
- Click the date/time picker to adjust if needed

### 2. Enter Your Items

**Habits** (optional)
- Daily recurring activities like reading, exercise, musical practice
- Enter one per line
- All habits receive equal weighting (1/n where n = number of habits)

**Projects** (optional, but one of Projects/Tasks/Emails required)
- Ongoing focused work requiring 2+ hour blocks
- Enter one per line, **ordered by importance** (most important first)
- Weighted from 0.9 (first) down to 0.5 (minimum)
- One project is randomly selected (weighted by priority) for each 2-hour chunk in continuous free blocks

**Tasks** (optional, but one of Projects/Tasks/Emails required)
- Items that take less than 30 minutes each
- Enter one per line, **ordered by importance** (most important first)
- Weighted from 0.95 (first) down to 0.05 (minimum)

**Emails** (optional, but one of Projects/Tasks/Emails required)
- Individual emails to reply to
- Enter one per line, **ordered by importance** (most important first)
- Weighted from 0.95 (first) down to 0.05 (minimum)

### 3. Define Your Calendar

Enter a string representing your day in half-hour blocks:

- `f` = free half-hour
- `m` = meeting half-hour
- Numbers multiply the preceding character (supports multi-digit numbers)

**Examples:**
- `fmfm` = free, meeting, free, meeting (2 hours total)
- `m4f2m` = 2-hour meeting, 1-hour free, half-hour meeting (3.5 hours total)
- `f8m2f4` = 4 hours free, 1 hour meeting, 2 hours free (7 hours total)
- `f16` = 8 hours free (full working day)
- `m4f12m2` = 2-hour meeting, 6 hours free, 1-hour meeting (9 hours total)

### 4. Generate Your Plan

Click **Run/Refresh** to:
1. Create a randomized list of habits, tasks, and emails (max 50 items)
2. Generate your calendar with time slots
3. Auto-place one randomly-selected project into each 2-hour chunk of continuous free time
4. Display weight indicators (small numbers) and opacity variations showing relative priority

### 5. Organize Your Day

**Using Mouse/Trackpad:**
- Drag items from the Randomised column to free time slots in the Calendar
- Hover over items to see probability weighting tooltips
- Click the **X button** on the right side of any item to remove it

**Using Touch/Mobile:**
- Long-press and swipe items to drag them to calendar slots
- Visual feedback shows where you're dropping
- Tap the **X button** on the right side of any item to remove it

**Using Keyboard:**
- Tab to navigate between items and slots
- Space to pick up/place an item
- Arrow Up/Down to select destination slot when item is picked up
- Escape to cancel

**General:**
- Each free slot can hold **up to 4 items**
- **Before** and **After** sections (green borders) accept **unlimited items** for overflow
- Drag items back to the Randomised column to remove them
- **Click the X button** on any item (in Randomised list or Calendar) to remove it permanently
  - Removed items will only reappear if you click **Run/Refresh** to regenerate the randomization
- Meeting slots show "Meetings schmeetings" (non-editable)
- Items are color-coded:
  - Purple = Habits
  - Orange = Projects
  - Blue = Tasks
  - Teal = Emails

### 6. Export or Print

**Copy Markdown:**
- Click **Copy Markdown** to copy your schedule to clipboard
- Format includes date heading, Before section, time slots, After section, and task checkboxes (`- [ ]`)
- Meeting slots are consolidated (e.g., "10:30 AM - 11:30 AM: Meetings schmeetings")
- Paste into Obsidian, Notion, or any markdown-compatible app

**Print Calendar:**
- Click **Print Calendar** to generate a print-ready version (formatted for A4)

### 7. Clear and Start Over

Click **Clear Data** to reset all inputs and start fresh.

## Technical Details

- **Pure HTML/CSS/JavaScript** - No frameworks or build steps required
- **localStorage API** - Full state persistence across browser sessions
- **Drag and Drop API** - Native browser drag-and-drop functionality
- **Responsive Design** - Works on desktop and mobile devices
- **GitHub Pages Compatible** - Static hosting, no server required

## Development

This application was developed using **Claude Sonnet 4.5** via **GitHub Copilot** in Visual Studio Code. The entire application, including UI/UX design, weighted randomization algorithm, drag-and-drop functionality, and localStorage persistence, was created through AI-assisted development.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Browser Compatibility

Works in all modern browsers that support:
- localStorage
- HTML5 Drag and Drop API
- CSS Grid
- ES6 JavaScript

Tested in Chrome, Edge, Firefox, and Safari.
