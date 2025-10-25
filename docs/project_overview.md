# Tournament Bracket Challenge – Project Brief

## 1. Vision & Context
Build a modern web application that lets friends compare single-elimination tournament picks (e.g., tennis Grand Slams). Each participant completes their bracket via drag-and-drop before the tournament begins. Picks remain hidden from others until the bracket is officially locked. The system must allow administrators to configure scoring rules, manage tournaments, and optionally notify players about milestones and results.

## 2. Goals
- Support medium-sized single-elimination tournaments (up to 128 entrants) with flexible seeding to cover tennis Grand Slams.
- Provide an intuitive drag-and-drop bracket builder for desktop users at launch, with an upgrade path to mobile-friendly interactions.
- Allow admins to configure tournaments, define custom scoring rules, and control when brackets lock.
- Keep opponent picks hidden until the tournament starts or all players have submitted, whichever comes first; admins can trigger an early lock.
- Enable authenticated players to submit and, if permitted, update picks while the tournament remains unlocked.
- Supply optional email notifications (e.g., reminders before lock, results after rounds).
- Modernize the stack to a current Django release suited to low-cost/free hosting.

### Non-Goals (Phase 1)
- Real-time match updates or live data feeds.
- Support for double-elimination or round-robin formats.
- Native mobile applications (responsive web will come later).
- Comprehensive social features (e.g., global leaderboards, chat).

## 3. Stakeholders & Roles
- **Admins**: Create tournaments, configure scoring, invite players, manage locks, review results.
- **Players**: Authenticate, build bracket predictions, view friends’ brackets once unlocked, receive notifications.
- **Spectators (future)**: Read-only viewers of public tournaments (not in scope for MVP).

## 4. High-Level Requirements

### Functional
1. **Tournament Management**
   - Admins can create tournaments with metadata (name, sport, start date/time, description, participant seeds, bracket size).
   - Admins upload or seed entrants and assign them to bracket positions.
   - Admins manage bracket lock status: automatic at scheduled start or manual override.
2. **Bracket Picks**
- Authenticated players join tournaments and populate their personal bracket via drag-and-drop.
- Picks hidden from other players until lock; admins can view at any time.
- Optional rule to allow edits after lock (default disallowed) configurable per tournament.
- Predictive backfilling: advancing a player in a later round automatically fills their prior matches with the corresponding wins so brackets stay logically consistent.
3. **Scoring**
   - Admin-configurable point schema (per round weighting, upset bonuses, champion bonus, etc.).
- System computes scores after each round based on actual results entered by admins.
- Backfill validator ensures automatic picks respect bracket structure and alerts players if manual adjustments are required.
4. **Results Management**
   - Admins record actual match outcomes.
   - System updates standings, allowing players to compare scores.
5. **Notifications**
   - Optional email reminders: before lock, when brackets unlock for viewing, final results.
6. **Authentication & Authorization**
   - User registration/login (email + password, potential social login later).
   - Admin role assignment via Django admin or custom interface.
7. **Content Security**
   - Picks visible only to respective owners and admins pre-lock.

### Non-Functional
- Target modern Django (>= 4.2 LTS) with Python 3.11.
- Deployable on low-cost/free hosting (e.g., Railway, Render, Fly.io) with PostgreSQL or SQLite for prototypes.
- Responsive design prioritized for desktop, with roadmap to mobile support.
- Accessibility-aware drag-and-drop interactions.
- Logging, metrics, and error reporting suitable for small-scale ops.
- Development process emphasizes writing automated tests before feature implementation (test-driven mindset).

## 5. Example Scoring Structure
- Round of 128: 1 point per correct pick
- Round of 64: 2 points per correct pick
- Round of 32: 4 points per correct pick
- Round of 16: 8 points per correct pick
- Quarterfinals: 12 points per correct pick
- Semifinals: 20 points per correct pick
- Final: 32 points per correct pick
- Upset bonus: +3 points if a lower-seeded player beats a higher seed
- Perfect champion pick bonus: +5 points

## 6. User Stories (MVP)
1. **As an admin**, I can create a new tournament with a defined start time so players know when picks lock.
2. **As an admin**, I can import or input tournament seeds and bracket structure to seed the bracket.
3. **As an admin**, I can configure scoring rules using round multipliers and optional bonuses.
4. **As a player**, I can register, join a tournament, and complete my bracket via drag-and-drop before the lock deadline.
5. **As a player**, I cannot see friends’ picks until the tournament locks, ensuring fairness.
6. **As a player**, I receive an email reminder 24 hours before bracket lock (if notifications are enabled).
7. **As an admin**, I can record match outcomes and see updated leaderboard standings.

## 7. Architecture Draft

### Frontend
- React-based single-page client or Django templates enhanced with HTMX/Alpine (evaluate effort vs. time).
- Drag-and-drop implemented with a library (e.g., React Beautiful DnD) or HTML5 drag events.
- Progressive enhancement path to support touch devices in later phases.

### Backend
- Django 4.2 project with RESTful API (Django REST Framework) powering the bracket UI.
- PostgreSQL database on managed free tier (e.g., Neon, Supabase) for relational integrity.
- Celery + Redis (optional) for background tasks (email notifications); for MVP, rely on Django Q or async tasks supported by hosting.
- Use Django’s built-in admin for tournament configuration.
- Engineering workflow favors tests-first delivery (unit/integration tests) to enforce scoring logic, bracket validation, and API behavior before implementing features.

### Hosting & DevOps
- Containerized deployment (Docker) for parity between development and production.
- Target free-tier PaaS (Railway/Render) with scheduled background worker if notifications require asynchronous processing.
- GitHub Actions for CI (linting, tests).
- Environment variables for secrets; rely on managed email service (SendGrid, Mailgun free tier).

## 8. Data Model Sketch
- `User`: extends Django’s `AbstractUser`, adds roles.
- `Tournament`: metadata, start time, lock status, settings flags.
- `Entrant`: ties tournament to player/seed.
- `BracketNode`: structure for matches with references to previous nodes and winners.
- `Pick`: player’s selection per `BracketNode`.
- `ScoringRule`: configuration for base points/bonuses.
- `MatchResult`: actual outcomes recorded by admins.
- `NotificationPreference`: per-user toggles.

## 9. API & Feature Backlog (Initial)
1. Tournament CRUD for admins (REST + admin UI).
2. Endpoint to fetch bracket layout and entrants.
3. Endpoint to submit/update player picks before lock.
4. Endpoint to reveal opponents’ brackets post-lock.
5. Endpoint to record results and compute scores.
6. Notification scheduler (cron job or background worker).

## 10. UI/UX Considerations
- Bracket builder shows two sides of the bracket (top/bottom) similar to tennis draw.
- Drag participants to advance them; include keyboard-accessible controls.
- Dashboard summarizing tournaments player joined, lock countdown, and current standings.
- Admin console (within Django admin) for scoring rules with validation previews.
- Bracket builder provides instant visual feedback when auto-backfilling occurs and allows undoing propagated changes.

## 11. Security & Privacy
- Enforce HTTPS in production.
- CSRF protection for form submissions (Django default).
- Rate limiting on login/register endpoints (via DRF throttling or middleware).
- RBAC: Admin privileges restricted to trusted accounts.
- Audit trail for match updates and scoring adjustments.
- Comprehensive automated test suite runs in CI before merges to maintain quality.

## 12. Open Questions & Future Work
- Precise scoring customization UI (how granular should bonuses be?).
- Integration with third-party sports data feeds vs. manual result entry.
- Social sharing features or league grouping.
- Mobile-first redesign timeline.
- Support for team-based picks (doubles) or multi-tournament seasons.
- Gamification ideas such as streak achievements or side challenges.
- Integrations with calendar/reminder services for match start times.

## 13. Next Steps
1. Validate requirements and scoring flexibility with stakeholders.
2. Decide on frontend strategy (React SPA vs. Django templates + JS enhancements).
3. Prototype data model and bracket generation logic.
4. Define notification strategy and hosting provider.
5. Design automated test strategy (unit, integration, end-to-end) to precede feature development.
6. Plan migration path from legacy App Engine stack to modern deployment.

