# monorepo-decapcmsadmin
# Ibex content: how to add a full track

All programme content lives as Markdown files with YAML front matter. Decap CMS (`admin/config.yml`) is just an editor for those files; the app reads the files directly. There are no public web pages.

```
admin/config.yml        Decap CMS config (defines every field below)
content/tracks/         one file per track
content/weeks/          one file per week
content/days/           one file per day
content/assessments/    questionnaires (same question model as days)
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
3. **28 days** (`content/days/track_x_w1_d1.md` ... `track_x_w4_d7.md`)
4. **Validate**, then commit.

### 1. Track

| Field | Notes |
|---|---|
| `track_id` | `track_x`. Referenced by every week. |
| `title`, `description` | Shown in the track list. |
| `display_order` | Position among tracks (1, 2, 3 ...). |
| `day_order` | `free` = days can be done in any order. `sequential` = one after the other. |
| `week_unlock` | `after_previous`, `after_previous_or_skip_ahead`, or `all_open`. Controls when Week N+1 opens. |
| `responses_locked_after_completion` | `true` = completed days are read-only. |
| body | Optional overview text. |

### 2. Weeks

| Field | Notes |
|---|---|
| `week_id`, `track`, `week_number` | `track_x_week_2`, `track_x`, `2`. |
| `title`, `subtitle` | Required title; subtitle optional. |
| `epigraph_quote`, `epigraph_author` | Optional quote shown at the top. |
| `has_intro`, `intro_skippable`, `intro_cta_label` | The intro screen is the Markdown body of the file ("what this week is about", days-at-a-glance table). Set `has_intro: false` for no intro. **The intro never counts toward week completion.** |

### 3. Days

Every week has exactly seven days, `day_number` 1 to 7.

**Fields every day needs:** `day_id`, `week`, `day_number`, `day_type`, `title`, `estimated_minutes`, `completion`, `completes_week`, plus content (`blocks`, or `morning` / `evening` for two-part practice days). Optional: `day_label` (the "ZONE 2" / "EXERCISE" style tag), `subtitle`, `estimated_minutes_max`, `time_label` (when the time is not a simple range), `format_summary`, `submit_label`.

## What makes progression work

Progression depends on three fields on each day. Get these right and the app unlocks things as intended.

**`day_type` and `completion`** decide when a day counts as done:

| Day type | Typical use | `completion` | Content |
|---|---|---|---|
| `teaching` | Teaching, exercise, intervals | `exercise_submitted` | `blocks`, ending with an `exercise` block |
| `practice` (two-part) | Zone 2, real-world rep: morning prompt + evening check-in | `evening_submitted` | `morning.blocks` and `evening.blocks` |
| `practice` (single) | Short action day | `exercise_submitted` | `blocks` |
| `reflection` | Day 7 week review | `all_questions_answered` | `blocks`; questions `required: true` |
| any | Nothing to submit | `manual` | A Complete button only |

**`completes_week`**: set `true` on **exactly one day per week**, the last day (Day 7). Completing it completes the week and unlocks the next one (per the track's `week_unlock`). Set `false` on every other day.

**`submit_label`** is the button text:
- Normal days: `Complete Session`
- Day 7: `Complete Week N and continue to Week N+1`
- Final day of the track: `Complete Track X`

**`completion_screen`** (on Day 7 only) controls what shows after completing:
- `celebration: week` on Day 7 of Weeks 1-3
- `celebration: track` on Day 7 of Week 4, with buttons such as `home` and `explore_tracks`
- Optional `success_message`, `unlock_note`, body, and `followup_assessment` (offers an assessment, e.g. an exit measure)

Optional day-level helpers:
- `nudge_if_incomplete {day_number, message}`: gentle prompt if an earlier day is not done. Never blocks.
- `nothing_came_up_prompt`: shown when a practice-day user reports nothing happened. The day still completes.

## Content blocks

Day content is an ordered list of `blocks`:

| Block | Use |
|---|---|
| `text` | Prose. `style`: `normal`, `callout`, `note`, `scenario`, `example`, `science`, `excavating`, `anchoring`, `carry` ("To carry into your day"). Has an optional `heading`. |
| `section` | A phase divider (`open`, `deepen`, `build`, `other`) for tracks that use phases. |
| `expandable` | Collapsible content, e.g. "Research basis". Needs a `section_id`. |
| `recall` | Shows earlier answers by question ID (`from: [id, ...]`). `required: true` blocks the day if they are missing. |
| `interactive` | App widgets: `emotion_wheel`, `body_map`, `window_of_tolerance`, `timer`, `choice_point`, `decision_tree`. |
| `exercise` | The questions. `presentation`: `single_page` or `stepper` (one at a time). `available_from: end_of_week` for later follow-ups. |

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
| `min_selections`, `max_selections` | Multi-select limits. |
| `min_chars`, `max_chars`, `input_size` | Text limits and box size (`small`, `medium`, `large`). |
| `scale_min`, `scale_max` | Plus either `scale_min_label` / `scale_max_label` or a full `scale_labels` list; `scale_style`: `buttons` or `slider`. |
| `pattern_tag` | Marks answers for cross-week tracking (`emotion`, `body_sensation`, `trigger`, `regulation_strategy`, `attempt_outcome`). |
| `profile_label` | Surfaces the answer in the user's profile under this heading. |

Rules that matter for the app:
- A `recall` block or `show_if` can only point at a question that exists. Point only at IDs from earlier in the track.
- Conditional questions should usually be `required: false`, since they are not shown to everyone.
- If you change a question's wording, keep its `question_id`. If the meaning changes, create a new ID.

## Checklist for a complete track

- [ ] One track file; `display_order` does not clash with another track.
- [ ] Four week files, `week_number` 1-4, each with `track` set.
- [ ] 28 day files named `track_x_w{1-4}_d{1-7}.md`; `day_id` matches the filename; `week` points at the right week.
- [ ] Each week has `day_number` 1-7 with no gaps or duplicates.
- [ ] Exactly one `completes_week: true` per week (Day 7); `false` elsewhere.
- [ ] `completion` matches the day type (table above).
- [ ] Every teaching day ends with an `exercise` block.
- [ ] Day 7 of Weeks 1-3: `celebration: week`, `Complete Week N and continue to Week N+1`.
- [ ] Day 7 of Week 4: `celebration: track`, `Complete Track X`.
- [ ] Every `question_id` carries the track prefix and is unique across all tracks.
- [ ] Every `recall` and `show_if` reference exists.
- [ ] `python3 -I tools/validate.py admin/config.yml content` prints `OK`.

## Tips

- **YAML quoting:** wrap any plain-text value containing `: ` or starting with a special character in quotes. For multi-paragraph prose use a block scalar (`body: |-`) and indent the text.
- **Editing locally:** with `local_backend: true`, run `npx decap-server` next to your repo and open `/admin`.
- **Dates and times:** `estimated_minutes` is the low end; put the high end in `estimated_minutes_max`. Use `time_label` for text like "5 minutes in-app, rest in the world".
- **Worked examples:** see `content/` for each track. Track D is a good reference for the usual structure; Track E shows a fully converted 28-day track.