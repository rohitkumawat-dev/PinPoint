# PinPoint — MVP Product Specification
Version: 1.0
Status: Specification; implementation pending.
Working name: AtlasRush

## 1. Goal
Build a polished, responsive geography memory game.

The player has 10 minutes to type as many country names as
they can remember. Correctly guessed countries turn green
on an interactive world map. Unguessed countries remain grey
with clearly visible borders.

## 2. Scope
Include:
- One world-map country recall game.
- Start button and 10-minute countdown.
- Country answer input and Submit button.
- Live score and progress indicator.
- Green colouring for correct answers.
- Zoom, pan, and reset-map controls.
- Recently guessed country list.
- End-of-game results and Play Again.
- Refresh recovery.

Exclude for this version:
- Accounts, multiplayer, leaderboards, backend, database.
- Globe view, geography facts, capitals, flags, difficulty modes.
- AI features, hints, autocomplete, and paid map services.

## 3. Technology
- React + TypeScript + Vite.
- Tailwind CSS; CSS transitions for small animations.
- d3-geo for SVG map paths and an Equal Earth projection.
- d3-zoom for map navigation.
- Locally bundled, simplified map geometry.
- React useReducer for game state.
- localStorage for game persistence.
- Vitest and React Testing Library for logic/components.
- Playwright for essential browser flows.
- Use compatible stable dependencies and commit the lockfile.

React owns the SVG country elements.
D3 calculates geometry and navigation transforms.

## 4. Visual Direction
Style: clean, premium atlas with a focused game interface.

Colours:
- Page background: #F6F7F9
- Panels: #FFFFFF
- Main text: #172033
- Primary controls: #2563EB
- Map ocean/background: #EDF4F8
- Unguessed countries: #CBD2DC
- Guessed countries: #22A06B
- Country borders: #FFFFFF

Layout:
- Compact header with AtlasRush branding.
- Title: "How much of the world can you remember?"
- Short instructions before starting.
- Main game card dominated by the world map.
- Score, timer, and progress above the map.
- Prominent answer input directly below the map.
- Small recent-answer list below the input.

Use generous spacing, restrained shadows, rounded corners,
clear typography, and subtle 150–250 ms transitions.
Avoid excessive gradients, decorative panels, and animations.

Desktop: map receives most of the screen space.
Mobile: stack controls; preserve usable input and map height.
Do not shrink the entire desktop layout into a narrow screen.

## 5. Quiz Country Policy
Use a fixed 195-answer quiz roster:
193 UN member states plus the Holy See and Palestine.

Display the convention in the rules.
This is a gameplay convention, not a claim that every
territory or disputed entity has the same status.

Use stable internal IDs for answers.
Do not derive the roster or total from map feature count.
Non-roster areas remain neutral and do not affect scoring.
Exclude Antarctica from the quiz.

Each answer must map to its applicable geographic features.
Provide unobtrusive locator dots for tiny countries that
cannot be seen clearly at the default zoom.
Locator dots turn green when their country is guessed.

## 6. Gameplay
Initial state:
- Show all quiz countries grey with visible borders.
- Show 10:00 and 0/195.
- Keep answer input disabled until Start.

Start:
- Start the countdown and focus the answer input.

Submission:
- Submit using Enter or the Submit button.
- Use explicit submission, not automatic acceptance while
  typing; this prevents prefix conflicts such as
  "Guinea" and "Guinea-Bissau".
- Validate locally.
- Correct answer: colour all linked features green,
  increment score once, clear input, and retain focus.
- Duplicate: show "Already guessed"; do not change score.
- Unknown answer: show "Country not recognised";
  retain and select the input for easy correction.
- No point or time penalty.

Feedback:
- Show "India added!" or equivalent after a correct answer.
- Keep feedback near the input.
- Do not reveal unguessed country names on hover,
  focus, or map click during gameplay.
- Do not automatically move or zoom the map after answers.

End:
- End when the countdown reaches zero or all answers
  have been found.
- Disable further submissions.
- Keep guessed countries green and missed countries grey.
- Show score, percentage, guessed list, and missed list.
- Allow country names to be inspected after the game.
- Play Again resets the game to its initial state.

## 7. Answer Matching
Normalise both input and aliases:
- Trim leading/trailing whitespace.
- Convert to lowercase.
- Remove diacritics.
- Standardise apostrophes, punctuation, and repeated spaces.

Use exact matching against a curated alias lookup.
No fuzzy matching or silent typo acceptance in MVP.

Examples:
- USA, US, United States, United States of America
- UK, United Kingdom, Great Britain
- UAE, United Arab Emirates
- Ivory Coast, Côte d'Ivoire
- Czech Republic, Czechia
- Turkey, Türkiye
- Vatican City, Holy See
- DRC, DR Congo, Democratic Republic of the Congo

Do not treat England, Scotland, Wales, or Northern Ireland
as separate answers or aliases for the whole UK.

Ambiguous "Congo" must ask the player to specify
Republic of the Congo or Democratic Republic of the Congo.
The alias validator must detect conflicting aliases.

## 8. Timer and Persistence
Game states: idle, running, finished.

At Start:
- Save an absolute end timestamp: Date.now() + 600000.
- Derive remaining time from that timestamp.
- Do not implement the clock by subtracting one each tick.

The timer continues while the tab is hidden.
Refresh restores guesses and the original deadline.
An expired restored game opens in the finished state.

Before accepting every answer, check the current deadline.
Reject answers submitted at or after the deadline.
Never display negative time.

Persist:
- Schema version.
- Country dataset version.
- Game status.
- End timestamp.
- Guessed country IDs.

Validate stored data before restoring.
If storage is unavailable, gameplay must still work.
Discard incompatible or malformed saved games safely.

## 9. Data Model
Country:
- id: stable internal identifier
- name: display name
- aliases: string[]
- featureIds: string[]
- locator: optional [longitude, latitude]

Map feature:
- Stable feature ID
- Geometry
- Optional linked quiz country ID

Game:
- status: idle | running | finished
- endsAt: timestamp | null
- guessedIds: unique country IDs

Derive score from unique guessed IDs.
Derive total from the fixed quiz roster.
Keep geography and answer matching independent.

## 10. Suggested Structure
src/
  components/
    WorldMap.tsx
    GameHeader.tsx
    AnswerInput.tsx
    RecentGuesses.tsx
    ResultsPanel.tsx
  hooks/
    useQuizGame.ts
  data/
    countries.json
    world.geo.json
  lib/
    normalizeAnswer.ts
    matchAnswer.ts
    storage.ts
  styles/
    index.css
  App.tsx

## 11. Performance and Accessibility
- Bundle data locally; no runtime country-data API requests.
- Simplify map geometry without dropping needed countries.
- Memoise projected paths.
- Timer ticks must not recompute geographic paths.
- Keep borders visible during zoom.
- Provide zoom-in, zoom-out, and reset buttons.
- Support keyboard submission and visible focus states.
- Label the input and announce answer feedback politely.
- Do not announce every timer tick to screen readers.
- Support reduced-motion preferences.
- Show score and answer lists so colour is not the only cue.
- Keep touch targets at least 44px.
- Avoid horizontal page overflow on mobile.

## 12. Acceptance Criteria
1. Start begins a 600-second game.
2. "India" colours India green and adds exactly one point.
3. " india " is recognised after normalisation.
4. Repeating India does not add another point.
5. USA and United States refer to the same answer.
6. Unknown input does not alter map or score.
7. Unguessed countries stay grey with visible borders.
8. No unguessed names are exposed through map interactions.
9. Every roster answer has geometry or a visible locator.
10. Refresh restores the game without resetting the timer.
11. Hidden-tab time counts toward the deadline.
12. No answers are accepted after expiry.
13. Results show correct guessed and missed lists.
14. Play Again clears the previous game.
15. The game works with keyboard and mobile touch.
16. All alias conflicts and unmapped answers are caught
    by data-validation tests.

## 13. Build Order
1. Establish and validate the country roster and map mapping.
2. Render the responsive world map with real boundaries.
3. Implement answer matching and green country fills.
4. Add timer, scoring, persistence, and results.
5. Refine visual design, mobile layout, and transitions.
6. Run data, logic, and browser acceptance checks.

Deliver working source code and a short README containing
setup commands, build/test commands, data attribution,
quiz country policy, and known limitations.
