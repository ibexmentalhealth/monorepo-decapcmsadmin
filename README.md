# Ibex content: how to add a full track
 
All programme content lives as Markdown files with YAML front matter. Decap CMS (`admin/config.yml`) is just an editor for those files; the app reads the files directly. There are no public web pages.
 
```
admin/config.yml        Decap CMS config (defines every field below)
content/tracks/         one file per track
content/units/          one file per unit
content/days/           one file per day
content/questions/      one file per question (used by days and questionnaires)
content/questionnaires/ one file per questionnaire (an ordered list of questions)
content/assessments/    one file per assessment (the result after a questionnaire)
content/variants/       one file per variant text, with its scoring conditions
tools/validate.py       checks content against the config before you commit
```
 
## The model
 
```
Track  ->  Unit  ->  Day
```
 
Each level points up to its parent by ID: a unit has `track: track_x`, a day has `unit: track_x_unit_1`. The filename is always `<ID>.md`, and the ID must match the `track_id` / `unit_id` / `day_id` field. Lowercase letters, digits and underscores only. **Never rename an ID after launch.**
 
## Order of work for a new track
 
Create things top-down, because children reference their parents.
 
1. **Track** (`content/tracks/track_x.md`)
2. **Units** (`content/units/track_x_unit_1.md`, `track_x_unit_2.md`, ...)
3. **Days** (28 for a full track; `content/days/track_x_w1_d1.md` ... `track_x_w4_d7.md`)
4. **Validate**, then commit.
### 1. Track
 
| Field | Notes |
|---|---|
| `track_id` | `track_x`. Referenced by every unit. |
| `title`, `description` | Shown in the track list. |
| `display_order` | Position among tracks (1, 2, 3 ...). |
| body | Overview text (required in the CMS). |
| `routing` | Optional JSON text: the rules that make this track a candidate for a user, in the same condition format as variants (see Conditions). The backend evaluates it; empty = no rules. |
 
### 2. Units
 
| Field | Notes |
|---|---|
| `unit_id`, `track`, `unit_number` | `track_x_unit_2`, `track_x`, `2`. |
| `title`, `subtitle` | Required title; subtitle optional. |
| `epigraph_quote`, `epigraph_author` | Optional quote shown at the top. |
| `has_intro` | The intro screen is the Markdown body of the file ("what this unit is about", days-at-a-glance table). Set `has_intro: false` for no intro. |
 
### 3. Days
 
`day_number` sets the order within the unit. Days follow each other in order; the last day of a unit is followed by the first day of the next unit.
 
**Fields every day needs:** `day_id`, `unit`, `day_number`, `day_type`, `title`, `estimated_minutes`, `lengths`, plus `blocks`. Optional: `evidence`, `day_label` (the "ZONE 2" / "EXERCISE" style tag), `subtitle`, `estimated_minutes_max`, `time_label` (when the time is not a simple range), `format_summary`.
 
## Day blocks
 
A day is an ordered list of `blocks`:
 
| Block | Use |
|---|---|
| `text` | Prose. `style`: `normal`, `callout`, `note`, `scenario`, `example`, `science`, `excavating`, `anchoring`, `carry`. Optional `heading`. |
| `section` | A phase divider (`open`, `deepen`, `build`, `other`). |
| `expandable` | Collapsible content. Needs a `section_id`. |
| `recall` | Shows earlier answers: a `label` and a list of questions. |
| `check_in` | Attaches a check-in question to the day. It is asked later as a notification, outside the day order, and never blocks progress. |
| `question` | A question the user answers here, picked from the Questions collection. |
 
**`lengths`** says which versions of the track include the day: `short`, `medium`, `full` (tick one or more; default all three). A day in Medium and Full but not Short has `lengths: [medium, full]`. The app builds each person's Short, Medium or Full track from these tags.
 
**`evidence`** is an optional citation or research basis for the day. It is stored apart from the day text so the app can show it behind an icon.
 
There is no completion content on a day. A completion moment is an ordinary final day.
 
## Questions
 
Each question is its own file in `content/questions/`, written once and picked by ID in day `question` blocks, `recall` blocks and questionnaires. The filename is `<question_id>.md`.
 
| Field | Notes |
|---|---|
| `question_id` | **Globally unique, never renamed.** Answers are stored against it. Prefix with the track: `d_w2_d3_bring`. |
| `type` | `text`, `single_select`, `multi_select`, `toggle`, `dropdown`, `scale`, `date` |
| `prompt`, `help`, `placeholder` | Question text, optional hint, optional placeholder (text only). |
| `scale_min`, `scale_max` | Scale only, plus `scale_min_label` / `scale_max_label` or a full `scale_labels` list. |
| `options` | `value` + `label`, plus `has_text_field` ("Other"), `is_exclusive` (multi select) and `score` (only when the question feeds a metric). The answer is saved as the option's **`value`** (e.g. `a_bit_tense`), not its position or label. Labels can be reworded and options added, removed or reordered after launch; **never change a `value` once answers exist**. A toggle has exactly two. |
| `metric` | Feeds this metric on the user's profile (pick from the list). Scale, single select or dropdown only. |
| `modifier` | Sets this modifier on the profile (pick from the list). Toggle only. A question feeds a metric or a modifier, never both. |
| `follow_up` | A question asked right after this one (e.g. "What happened?"). Follow-up questions are **not** placed in day blocks or questionnaires; they are reached only through the question that owns them. |
| `notification` | Only for check-in questions: the `title` and `message` of the notification that asks it. |
| `follow_up_conditions` | Optional JSON text. The backend only asks the follow-up when these conditions are true; empty = always ask. |
 
If you change a question's wording, keep its `question_id`. If the meaning changes, create a new ID.
 
## Metrics and modifiers
 
Metrics (numbers on the user's profile, e.g. Stress) and modifiers (true/false flags, e.g. Disruption) are not content. They live in the database; the CMS only offers their IDs as a pick list on questions, which a developer keeps in sync (`metric` and `modifier` options in `admin/config.yml`).
 
## Check-ins
 
A check-in is a question asked later as a notification, for practice that happens in real life. It lives outside the day order and never blocks the plan.
 
1. Write the check-in as ordinary questions. For "Done / Not now / Didn't fit" then "how much did it help?": a single-select question with options `done`, `not_now`, `didnt_fit`, whose `follow_up` is a 0-10 scale question, with `follow_up_conditions` set so it is only asked after Done.
2. Give the first question a `notification` (title and message).
3. In the day where the practice starts, add a `check_in` block pointing at that question.
When the nudge is sent, at what time, and whether it repeats the next day are decided by the app, not the CMS.
 
## Questionnaires and assessments
 
- **Questionnaire** (`content/questionnaires/`): an opening text, an ordered list of `questions` (each one is a step / view), and optionally an `assessment` shown at the end.
- **Assessment** (`content/assessments/`): `blocks` shown top to bottom, each either fixed `text` (heading and body) or a `variant_set`.
- **Variant set**: a label, the variants in it, and **Show**: only the highest scoring variant, only the lowest, or all.
- **Variant** (`content/variants/`): a `name` (e.g. Hook), the `text`, and `conditions` as JSON.
### Conditions
 
A variant's `conditions` is plain JSON text. The CMS does not interpret it; the backend does all scoring from it. Formats:
 
```
{"metric_threshold": ["stress", ">", 3]}
{"modifier_is": ["disruption", true]}
{"and": [ ...conditions... ]}
{"or":  [ ...conditions... ]}
```
 
Several conditions go in a JSON list. Metric and modifier IDs must exist in the pick lists above; the validator checks the JSON parses and the IDs are known.
 
## Checklist for a complete track
 
- [ ] One track file; `display_order` does not clash with another track.
- [ ] One file per unit, `unit_number` in order, each with `track` set.
- [ ] One day file per day, named `track_x_u{n}_d{n}.md`; `day_id` matches the filename; `unit` points at the right unit.
- [ ] No duplicate `day_number` within a unit.
- [ ] Every question used in a day exists in `content/questions/` with a unique `question_id`.
- [ ] `python3 -I tools/validate.py admin/config.yml .` prints `OK`.
The validator also checks that every reference exists, that a question's type fits its metric or modifier, and that variant conditions are valid JSON with known IDs.
 
## Tips
 
- **YAML quoting:** wrap any plain-text value containing `: ` or starting with a special character in quotes. For multi-paragraph prose use a block scalar (`body: |-`) and indent the text.
- **Editing locally:** with `local_backend: true`, run `npx decap-server` next to your repo and open `/admin`.
- **Dates and times:** `estimated_minutes` is the low end; put the high end in `estimated_minutes_max`. Use `time_label` for text like "5 minutes in-app, rest in the world".