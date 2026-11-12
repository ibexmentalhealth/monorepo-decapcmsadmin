# Ibex content: how to add a full track
 
All programme content lives as Markdown files with YAML front matter. Decap CMS (`admin/config.yml`) is just an editor for those files; the app reads the files directly. There are no public web pages.
 
```
admin/config.yml        Decap CMS config (defines every field below)
content/tracks/         one file per track
content/weeks/          one file per week
content/days/           one file per day
content/assessments/    questionnaires (same question model as days)
content/assessment_result/  the copy on the assessment result screen (see the last section)
tools/validate.py       checks content against the config before you commit
```
 
## The model
 
```
Track  ->  Week (x4)  ->  Day (x7)
```
 
Each level points up to its parent by ID: a week has `track: track_x`, a day has `week: track_x_week_1`. The filename is always `<ID>.md`, and the ID must match the `track_id` / `week_id` / `day_id` field. Lowercase letters, digits and underscores only. **Never rename an ID after launch.**
 
## Order of work for a new track
 
Create things top-down, because children reference their parents.
 
1. **Track** (`content/tracks/track_x.md`)
2. **4 weeks** (`content/weeks/track_x_week_1.md` ... `_4.md`)
3. **Days** (28 for a full track; `content/days/track_x_w1_d1.md` ... `track_x_w4_d7.md`)
4. **Validate**, then commit.
### 1. Track
 
| Field | Notes |
|---|---|
| `track_id` | `track_x`. Referenced by every week. |
| `title`, `description` | Shown in the track list. |
| `display_order` | Position among tracks (1, 2, 3 ...). |
| body | Overview text (required in the CMS). |
 
### 2. Weeks
 
| Field | Notes |
|---|---|
| `week_id`, `track`, `week_number` | `track_x_week_2`, `track_x`, `2`. |
| `title`, `subtitle` | Required title; subtitle optional. |
| `epigraph_quote`, `epigraph_author` | Optional quote shown at the top. |
| `has_intro` | The intro screen is the Markdown body of the file ("what this week is about", days-at-a-glance table). Set `has_intro: false` for no intro. **The intro never counts toward week completion.** |
 
### 3. Days
 
`day_number` is 1 to 7. Most tracks run all seven days of a week; a track that does not simply leaves the missing day numbers out.
 
**Fields every day needs:** `day_id`, `week`, `day_number`, `day_type`, `title`, `estimated_minutes`, `completes_week`, plus content (`blocks`, or `morning` / `evening` for two-part practice days). Optional: `day_label` (the "ZONE 2" / "EXERCISE" style tag), `subtitle`, `estimated_minutes_max`, `time_label` (when the time is not a simple range), `format_summary`.
 
## What makes progression work
 
Days are always sequential on every track: each day opens once the previous day is complete. There is no per-track setting for this.
 
Progression depends on `day_type` and `completes_week`. The app decides when a day counts as done from its structure:
 
| Day type | Typical use | Content |
|---|---|---|
| `teaching` | Teaching, exercise, intervals | `blocks`, ending with an `exercise` block |
| `practice` (two-part) | Zone 2, real-world rep: morning prompt + evening check-in | `morning.blocks` and `evening.blocks` |
| `practice` (single) | Short action day | `blocks` |
| `reflection` | Week review | `blocks`; questions `required: true` |
 
**`completes_week`**: set `true` on **exactly one day per week**, the last day of that week. This is Day 7 for tracks that run every day, and an earlier day for tracks that do not. Completing it completes the week and the next week becomes available. Set `false` on every other day.
 
**`completion_screen`** (on the day that completes the week) controls what shows after completing:
- `celebration: week` on the last day of Weeks 1-3
- `celebration: track` on the last day of Week 4
- Optional `success_message`, body, and `followup_assessment` (offers an assessment, e.g. an exit measure)
## Content blocks
 
Day content is an ordered list of `blocks`:
 
| Block | Use |
|---|---|
| `text` | Prose. `style`: `normal`, `callout`, `note`, `scenario`, `example`, `science`, `excavating`, `anchoring`, `carry` ("To carry into your day"). Has an optional `heading`. |
| `section` | A phase divider (`open`, `deepen`, `build`, `other`) for tracks that use phases. |
| `expandable` | Collapsible content, e.g. "Research basis". Needs a `section_id`. |
| `recall` | Shows earlier answers by question ID (`from: [id, ...]`). `required: true` blocks the day if they are missing. |
| `exercise` | The questions. The app shows them one at a time. |
 
A teaching day should end with an `exercise` block, because that is what `exercise_submitted` waits on.
 
## Questions
 
Questions live inside `exercise` blocks.
 
| Field | Notes |
|---|---|
| `question_id` | **Globally unique, never renamed.** Responses are stored against it. Use a track prefix: `d_w2_d3_bring` = Track D, Week 2, Day 3. |
| `type` | `text`, `single_select`, `multi_select`, `toggle`, `dropdown`, `scale`, `date` |
| `prompt`, `help`, `required` | Question text, optional hint, whether it must be answered. |
| `group_label` | Heading shown above a run of related questions. |
| `show_if {question_id, equals}` | Conditional question. `equals: "a\|b"` matches either value. |
| `options` | `value` (lowercase, underscores) + `label`. `has_text_field: true` = "Other, please say". `is_exclusive: true` = picking it clears other choices (multi-select, e.g. "None of these"). |
| `scale_min`, `scale_max` | Plus either `scale_min_label` / `scale_max_label` or a full `scale_labels` list. |
| `builds_patterns` | `true` = the app uses this answer to find patterns across weeks. |
| `builds_profile` | `true` = the app uses this answer to build the user's profile. |
 
Rules that matter for the app:
- A `recall` block or `show_if` can only point at a question that exists. Point only at IDs from earlier in the track.
- Conditional questions should usually be `required: false`, since they are not shown to everyone.
- For "nothing happened" cases on practice days, ask a question (e.g. "Did something come up today?") and put the follow-up in a conditional question with `show_if`. The consequence is then ordinary content, editable per day.
- If you change a question's wording, keep its `question_id`. If the meaning changes, create a new ID.
## Checklist for a complete track
 
- [ ] One track file; `display_order` does not clash with another track.
- [ ] Four week files, `week_number` 1-4, each with `track` set.
- [ ] One day file per day, named `track_x_w{1-4}_d{1-7}.md` (28 for a track that runs every day); `day_id` matches the filename; `week` points at the right week.
- [ ] No duplicate `day_number` within a week.
- [ ] Exactly one `completes_week: true` per week (the last day); `false` elsewhere.
- [ ] Every teaching day ends with an `exercise` block.
- [ ] Last day of Weeks 1-3: `celebration: week`.
- [ ] Last day of Week 4: `celebration: track`.
- [ ] Every `question_id` carries the track prefix and is unique across all tracks.
- [ ] Every `recall` and `show_if` reference exists.
- [ ] `python3 -I tools/validate.py admin/config.yml .` prints `OK`.
## Tips
 
- **YAML quoting:** wrap any plain-text value containing `: ` or starting with a special character in quotes. For multi-paragraph prose use a block scalar (`body: |-`) and indent the text.
- **Editing locally:** with `local_backend: true`, run `npx decap-server` next to your repo and open `/admin`.
- **Dates and times:** `estimated_minutes` is the low end; put the high end in `estimated_minutes_max`. Use `time_label` for text like "5 minutes in-app, rest in the world".
- **Worked examples:** see `content/` for each track. Track D is a good reference for the usual structure; Track E shows a fully converted 28-day track.
## Assessment result copy
 
The text on the assessment result screen (hooks, states, personality profiles, life areas, track recommendations, closing line, disclaimer) lives in `content/assessment_result/`, one file per block, edited under "Assessment result copy" in the CMS.
 
- The scoring code decides which variant a user sees. It keys off the `code` values (`HOOK_01`, `STATE_02`, `AREA_03`, ...), so edit wording freely but **do not change codes**, and add or remove entries only together with a developer.
- **New track:** add an entry to `track_recommendations.md` (track, code like `TRACK_G`, text). Without it, a user routed to the new track sees no recommendation.