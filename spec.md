1. Overview
RehabFlow is a mobile-first, client-side web application designed to help users plan, track, and analyze their physical rehabilitation exercises. It allows users to schedule weekly routines, log sets with detailed metrics (reps, weight, heart rate), track the perceived difficulty (mood) of each set, and monitor recovery metrics like Left/Right leg strength symmetry.

Shutterstock

2. Technical Stack
Frontend: Vanilla HTML5, CSS3 (Custom Variables, Flexbox/Grid), Vanilla JavaScript (ES6+).

Architecture: Single Page Application (SPA) with DOM manipulation toggling .screen classes.

Storage: Browser localStorage (Key: rehabflow_weekplan_v1). No backend server is required.

Icons/Fonts: SVG icons (inline), 'Inter' (UI text) and 'Fraunces' (Headings) via Google Fonts.

3. Data Architecture (State Management)
The application state is maintained in a single JavaScript state object and synced to localStorage.

Core Data Models:
User: { name: String, leg: String ('left'|'right'|'both'), settings: { reminders: Boolean, rest: Boolean } }

Catalog (Exercises): { id: String, name: String, defaultType: String, emoji: String, color: String }

Plans By Day: Dictionary mapping a dayKey (e.g., "2023-10-25") to an array of planned items.

Plan Item: { id: String, exerciseId: String, type: String, targets: { sets, reps, weight, minutes, hr } }

Logs (Completed Sets): Array of logged sets.

Log Item: { ts: Number (timestamp), dayKey: String, sessionId: String, exerciseId: String, type: String, reps: Number, weight: Number, hr: Number, mood: String, moodScore: Number, leg: String, planItemId: String, setComment: String, exNote: String }

4. UI/UX & Screens
4.1. Onboarding Screen (#screen-onboard)
Trigger: First-time load (when no user data is found in localStorage).

Flow: 1. User selects their injured side (Left leg, Right leg, Both).
2. Explainer screen outlining core features (Mon-Sun planning, logging).
3. Completes onboarding and initializes the user profile.

4.2. Dashboard Tab (#screen-dashboard)
Purpose: View past and current activity at a glance.

Header: Displays a dynamic 5-day rolling date range (View: Month Day - Month Day).

5-Day Week Strip: Displays 5 days (2 days prior, current day, 2 days forward).

Interactions: Left/Right arrows shift the calendar exactly 2 days. Clicking a day sets it as the active day.

UI: Displays Date, Day of Week, "X sets" completed, and a calculated Average Mood Emoji for that day.

Day Panel: Shows a list of only completed sets ("Done") for the selected day. Includes a button to jump to the Training tab to edit the plan.

Statistics Panel:

Sessions (7d): Unique days with logged activity in the last 7 days.

Total Sets: All-time sets logged.

Streak: Consecutive days of logged activity up to today.

L/R Symmetry: Percentage ratio of Max Weight lifted by the weaker leg vs. stronger leg.

Quality: Average mood score (out of 4.0) over the last 7 days.

4.3. Training Tab (#screen-train)
Purpose: Plan future workouts and view specific exercise targets.

5-Day Week Strip: Same functionality as the Dashboard strip. Includes a "Today" button to snap the view back to the current date.

Day Editor:

Free Session: A button (▶ Free Session) to jump straight into an unplanned workout.

Planned Exercises: List of exercises planned for the day. Users can directly edit target Reps, Sets, Weight, or BPM inline. Includes a ▶ Start button to begin a session pre-filled with that exercise.

Add Exercise: Input field with a <datalist> dropdown. Users select exercise name and type (Reps, Weight, Cardio/HR) to add to the day's plan.

Logged Sets: Read-only list of completed sets for the selected day.

4.4. Active Session Screen (#screen-session)
Purpose: Real-time workout tracker.

Features:

Header: Live duration timer and "Finish" button.

Context: Leg toggle (Left, Both, Right) and Exercise Type toggle (Reps, Reps+Weight, Minutes+HR).

Inputs: Dynamic inputs based on Type (e.g., Target BPM only shows if Cardio/HR is selected).

Mood Rating: Mandatory selection of how the set felt (Great 😄, Good 🙂, OK 😐, Hard 😫).

Notes: Optional per-set note, and an overall session note (📝 Overall session note) applied to all sets upon finishing.

History: Live-updating list of sets completed during the current session.

4.5. Summary Screen (#screen-summary)
Purpose: Post-workout confirmation.

Metrics Shown: Total session duration, total sets completed, and average session quality (mood score).

Action: "Back to Training" button returns the user to the main flow.

4.6. Profile Tab (#screen-profile)
Purpose: Manage user settings and data.

Settings:

Edit Name and Injured Side.

Toggle mock settings (Reminders, Auto rest timer).

Danger Zone: "Reset data" clears localStorage and reloads the app.

5. Key Business Logic & Calculations
Mood Scoring & Averages:

Great = 4, Good = 3, OK = 2, Hard = 1.

The 5-day week strip takes all logged sets for a day, averages their numeric mood score, rounds to the nearest whole number, and displays the corresponding emoji.

Symmetry Calculation (getSymmetryHtml):

Filters all logs for type === 'weight' and leg is strictly 'left' or 'right'.

Finds the absolute maximum weight lifted for each specific exercise per leg.

Calculates the ratio (Min Weight / Max Weight) for each exercise where both legs have data.

Returns the average of these ratios as a percentage.

Streak Calculation (calcStreakDays):

Looks backwards from today (0 days ago, 1 day ago, etc.).

Counts consecutive days that have at least one log entry. Breaks the loop on the first empty day. (If today is empty but yesterday has data, it evaluates as 0, strictly requiring daily activity).
