---
description: Design and scaffold accessible interface from PRD
---

# Scaffold Accessible Activity

The PRD (`docs/PROJECT.md`) calls for both an accessible and graphical interface to the activity. While it may end up being possible to fit both user experiences into a single view, start by creating an accessible version. This scaffolds the majority of the state management logic (Vuex/Pinia) that will drive both the accessible and graphical versions of the application.

## Process

1. **Observe Requirements** - Read `docs/PROJECT.md` to understand learning objectives, user flow, and interaction requirements

2. **Observe Static Content** - Find CSV files being loaded into the global state store; analyze their structure to understand the activity content

3. **Design Accessible UX** - For each learning objective, propose how screen reader users will:

   - Navigate and interact with content
   - Receive feedback (success, error, warnings)
   - Track their progress
   - Ask user for input on key UX decisions before proceeding

4. **Create State Management** - Scaffold the store module with:

   - State for navigating all data in the CSV
   - Actions/getters for navigating steps, answers, and feedback
   - Datastore persistence logic (`mapFromDatastore`/`mapToDatastore`)
   - Progress tracking with auto-save on key interactions

5. **Create Types** - Add enums and interfaces to `src/types/` that:

   - Map CSV column headers for type-safe access
   - Define step IDs, phases, or interaction types from the static content
   - Support the state management layer

6. **Create Helper Functions** - Create parsing and validation utilities in `src/helpers/` as needed to support the state management layer

7. **Create Components & View** - Build an accessible, clean, modern-looking interface:

   - Create reusable components where opportunities exist
   - Populate the activity view to achieve the full user flow
   - Ensure keyboard navigation and screen reader support
   - Use `aria-live` regions for reactive feedback (e.g., success/error messages after user answers, output changes the user should be alerted to even if not focused on that area) - only when necessary.
   - NOTE FOR PROJECTS USING NUXT UI: Many of those components already have `aria` attributes and other accessibility measures built in. Always defer to using those components first instead of adding custom `aria` regions. Many of the our projects use custom multimedia (interactive svg's, and other graphical components) - this is likely where you will need to add custom `aria` regions. Defer to the component documentation for the best practices on using those components.
   - NOTE FOR STEP-BASED PROJECTS: Screenreader and keyboard focus should be reset to the correct position once a step is completed. This is a common issue in slide-like interfaces where a user clicks next or previous and the contents of the main view are updated without resetting the focus to the top of the new contents.

8. **Document Status** - Update `docs/STATUS.md` based on guidelines in AGENTS.md

## Output

Ask clarifying questions during key decision points. Present the final plan to the user for confirmation before implementing.
