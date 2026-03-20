# AnkiMCP API Reference

Complete tool schemas from the **ankimcp MCP server** (AnkiWeb add-on `124672614`). All tools are prefixed `mcp__anki__*`. Read this when the SKILL.md doesn't cover the exact parameters you need.

---

## Table of Contents

1. [Sync](#sync)
2. [Decks](#decks)
3. [Models (Note Types)](#models-note-types)
4. [Notes — Create](#notes--create)
5. [Notes — Search and Read](#notes--search-and-read)
6. [Notes — Update and Delete](#notes--update-and-delete)
7. [Tags](#tags)
8. [Card Management](#card-management)
9. [Filtered Decks](#filtered-decks)
10. [Review Session](#review-session)
11. [FSRS Scheduler](#fsrs-scheduler)
12. [Media](#media)
13. [GUI Tools](#gui-tools)

---

## Sync

### `sync`

Sync local collection with AnkiWeb. Call at the start and end of every review session.

```json
{}
```

No parameters.

---

## Decks

### `list_decks`

List all decks. Pass `include_stats: true` for card counts (new/learn/review).

```json
{ "include_stats": true }
```

| Param | Type | Default | Purpose |
|---|---|---|---|
| `include_stats` | boolean | `false` | Include new/learn/review counts |

### `create_deck`

Create an empty deck. Supports `Parent::Child` nesting (max 2 levels). Safe to call on existing decks.

```json
{ "deck_name": "Parent::Child" }
```

| Param | Type | Required | Purpose |
|---|---|---|---|
| `deck_name` | string | yes | Deck name, `::` for nesting |

---

## Models (Note Types)

### `model_names`

List all available note type names.

```json
{}
```

### `model_field_names`

Get field names and descriptions for a model. Always call this before creating notes to confirm exact field names.

```json
{ "model_name": "Basic" }
```

| Param | Type | Required |
|---|---|---|
| `model_name` | string | yes |

### `model_styling`

Get the CSS for a model's card rendering.

```json
{ "model_name": "Basic" }
```

### `create_model`

Create a custom note type with fields, templates, and CSS.

```json
{
  "model_name": "Custom RTL",
  "in_order_fields": ["Front", "Back", "Notes"],
  "card_templates": [
    {
      "Name": "Card 1",
      "Front": "{{Front}}",
      "Back": "{{FrontSide}}<hr>{{Back}}"
    }
  ],
  "css": ".card { direction: rtl; font-size: 24px; }",
  "is_cloze": false
}
```

| Param | Type | Required | Purpose |
|---|---|---|---|
| `model_name` | string | yes | Name for the new model |
| `in_order_fields` | string[] | yes | Field names in order |
| `card_templates` | object[] | yes | Each has `Name`, `Front`, `Back` |
| `css` | string | no | Card styling CSS |
| `is_cloze` | boolean | no | `true` for cloze deletion models |

### `update_model_styling`

Replace the CSS for an existing model. Affects all cards using it.

```json
{
  "model_name": "Basic",
  "css": ".card { font-family: 'Georgia'; font-size: 20px; }"
}
```

| Param | Type | Required |
|---|---|---|
| `model_name` | string | yes |
| `css` | string | yes |

---

## Notes — Create

### `add_note`

Add a single note. Returns note ID on success.

```json
{
  "deck_name": "My Deck",
  "model_name": "Basic",
  "fields": { "Front": "Question", "Back": "Answer" },
  "tags": ["topic", "subtopic"],
  "allow_duplicate": false
}
```

| Param | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `deck_name` | string | yes | — | Target deck |
| `model_name` | string | yes | — | Note type |
| `fields` | object | yes | — | Keys must exactly match model fields |
| `tags` | string[] | no | `null` | Tags to apply |
| `allow_duplicate` | boolean | no | `false` | Allow duplicate content |

### `add_notes`

Batch-add up to 100 notes. Preferred over `add_note` for multiple cards. Atomic undo support. Partial success — individual failures don't block others.

```json
{
  "deck_name": "My Deck",
  "model_name": "Basic",
  "notes": [
    { "fields": { "Front": "Q1", "Back": "A1" } },
    { "fields": { "Front": "Q2", "Back": "A2" } }
  ],
  "tags": ["topic"],
  "allow_duplicate": false
}
```

| Param | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `deck_name` | string | yes | — | Target deck (shared by all notes) |
| `model_name` | string | yes | — | Note type (shared by all notes) |
| `notes` | object[] | yes | — | Array of `{fields: {...}}` objects |
| `tags` | string[] | no | `null` | Tags applied to all notes |
| `allow_duplicate` | boolean | no | `false` | Allow duplicates |

Cloze example:
```json
{
  "deck_name": "My Deck",
  "model_name": "Cloze",
  "notes": [
    { "fields": { "Text": "{{c1::Newton's second law}} states F = ma.", "Extra": "Classical mechanics" } }
  ]
}
```

---

## Notes — Search and Read

### `find_notes`

Search using Anki query syntax. Returns array of note IDs.

```json
{ "query": "deck:\"My Deck\" tag:physics", "limit": 100, "offset": 0 }
```

| Param | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `query` | string | yes | — | Anki search query |
| `limit` | number | no | `100` | Max results |
| `offset` | number | no | `0` | Skip first N results |

#### Query Syntax

```
deck:DeckName              notes in deck (quote if spaces: deck:"My Deck")
tag:tagname                notes with tag
is:due                     cards due for review
is:new                     cards never reviewed
is:suspended               suspended cards
added:N                    added in last N days
front:keyword              front field contains keyword
back:keyword               back field contains keyword
flag:N                     flagged cards (1=red, 2=orange, 3=green, 4=blue)
prop:due<=N                due within N days
card:N                     specific card ordinal
```

Combine with spaces (AND) or `OR`:
```
deck:"My Deck" tag:physics OR tag:chemistry
```

### `notes_info`

Get full note data from IDs — fields, tags, model info, CSS.

```json
{ "notes": [1234567890, 1234567891] }
```

| Param | Type | Required |
|---|---|---|
| `notes` | number[] | yes |

---

## Notes — Update and Delete

### `update_note_fields`

Modify fields on an existing note. The Anki browser must NOT have this note open.

```json
{ "id": 1234567890, "fields": { "Back": "Updated answer" } }
```

| Param | Type | Required | Purpose |
|---|---|---|---|
| `id` | number | yes | Note ID |
| `fields` | object | yes | Fields to update (partial OK) |

### `delete_notes`

Permanently delete notes and all associated cards. No undo.

```json
{ "notes": [1234567890], "confirmDeletion": true }
```

| Param | Type | Required | Purpose |
|---|---|---|---|
| `notes` | number[] | yes | Note IDs to delete |
| `confirmDeletion` | boolean | yes | Must be `true` |

---

## Tags

### `tag_management`

All tag operations use an `action` key inside a `params` object.

#### add_tags
```json
{ "params": { "action": "add_tags", "notes": [1234567890], "tags": "topic subtopic" } }
```

#### remove_tags
```json
{ "params": { "action": "remove_tags", "notes": [1234567890], "tags": "old-tag" } }
```

#### replace_tags
```json
{ "params": { "action": "replace_tags", "notes": [1234567890], "tag_to_replace": "old", "new_tag": "new" } }
```

#### get_tags
```json
{ "params": { "action": "get_tags" } }
```

#### clear_unused_tags
```json
{ "params": { "action": "clear_unused_tags" } }
```

Tags are space-separated strings, not arrays.

---

## Card Management

### `card_management`

All actions use a `params` object with an `action` key.

#### suspend / unsuspend
```json
{ "params": { "action": "suspend", "cards": [1234567890] } }
{ "params": { "action": "unsuspend", "cards": [1234567890] } }
```

#### bury / unbury
```json
{ "params": { "action": "bury", "cards": [1234567890] } }
{ "params": { "action": "unbury", "deck": "My Deck" } }
```
`unbury` takes a deck name and unburies ALL buried cards in that deck.

#### change_deck
```json
{ "params": { "action": "change_deck", "cards": [1234567890], "deck": "Target Deck" } }
```
Creates target deck if it doesn't exist.

#### set_due_date
```json
{ "params": { "action": "set_due_date", "cards": [1234567890], "days": "0" } }
```
`"0"` = due now, `"5"` = in 5 days, `"5-7"` = random range, `"5!"` = set due and reset interval.

#### set_flag
```json
{ "params": { "action": "set_flag", "cards": [1234567890], "flag": 1 } }
```
Flags: `0` = none/remove, `1` = red, `2` = orange, `3` = green, `4` = blue, `5–7` = custom.

#### forget_cards
```json
{ "params": { "action": "forget_cards", "cards": [1234567890] } }
```
Resets cards to new state. Options: `restore_position` (default true), `reset_counts` (default false).

#### reposition
```json
{ "params": { "action": "reposition", "cards": [1234567890], "position": 0 } }
```
Only works on NEW cards (queue=0). Non-new cards silently skipped.

---

## Filtered Decks

### `filtered_deck`

Manage filtered (cram) decks. All actions use a `params` object.

#### create_or_update
```json
{
  "params": {
    "action": "create_or_update",
    "deck_name": "Study: Physics",
    "search_terms": [
      { "search": "tag:domains::physics", "limit": 100, "order": 0 }
    ]
  }
}
```
Max 2 search terms per deck (Anki hard limit). Cards are borrowed, not duplicated.

#### rebuild
```json
{ "params": { "action": "rebuild", "deck_name": "Study: Physics" } }
```
Returns all cards to home decks, then re-pulls matching cards.

#### empty
```json
{ "params": { "action": "empty", "deck_name": "Study: Physics" } }
```
Returns all cards to home decks. Scheduling preserved.

#### delete
```json
{ "params": { "action": "delete", "deck_name": "Study: Physics" } }
```
Empties and removes the filtered deck. Cards go back to home decks.

---

## Review Session

### `get_due_cards`

Get the next due card from a deck in scheduler order. One card per call.

```json
{ "deck_name": "My Deck", "skip_images": false, "skip_audio": false }
```

| Param | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `deck_name` | string | yes | — | Deck to study |
| `skip_images` | boolean | no | `false` | Skip cards with images (buries them) |
| `skip_audio` | boolean | no | `false` | Skip cards with audio (buries them) |

Response includes `has_images` and `has_audio` flags.

### `present_card`

Show a card's question or answer.

```json
{ "card_id": 1234567890, "show_answer": false }
```

| Param | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `card_id` | number | yes | — | Card ID from `get_due_cards` |
| `show_answer` | boolean | no | `false` | `true` to reveal answer side |

### `rate_card`

Submit a rating. Never call without explicit user confirmation.

```json
{ "card_id": 1234567890, "rating": 3 }
```

| Param | Type | Required | Purpose |
|---|---|---|---|
| `card_id` | number | yes | Card ID |
| `rating` | number | yes | `1` = Again, `2` = Hard, `3` = Good, `4` = Easy |

### Review Workflow

```
1. sync()
2. get_due_cards(deck_name)         → card_id
3. present_card(card_id)            → show question
4. [user answers]
5. present_card(card_id, show_answer: true)
6. Suggest rating 1–4, wait for user confirmation
7. rate_card(card_id, rating)
8. Repeat from 2
9. sync() when done
```

---

## FSRS Scheduler

### `get_fsrs_params`

Get FSRS parameters for a deck preset. Empty `deck_name` returns all presets.

```json
{ "deck_name": "My Deck" }
```

### `set_fsrs_params`

Update FSRS parameters. At least one param must change.

```json
{
  "preset_name": "Default",
  "desired_retention": 0.90,
  "max_interval": 365,
  "fsrs_params": [0.4, 0.6, 2.4, 5.8]
}
```

| Param | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `preset_name` | string | yes | — | Deck config preset name |
| `desired_retention` | number | no | `-1` (unchanged) | Target retention `0.70–0.99` |
| `max_interval` | number | no | `-1` (unchanged) | Max days between reviews |
| `fsrs_params` | number[] | no | `null` | FSRS weight array |

### `optimize_fsrs_params`

Run FSRS optimizer on review history. Takes 5–30 seconds.

```json
{ "preset_name": "Default", "apply_results": false }
```

| Param | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `preset_name` | string | yes | — | Preset to optimize |
| `apply_results` | boolean | no | `false` | `false` = dry run, `true` = save |

### `get_card_memory_state`

Get FSRS memory state (stability, difficulty, retrievability) per card.

```json
{ "card_ids": [1234567890], "recompute": false }
```

| Param | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `card_ids` | number[] | yes | — | Cards to inspect |
| `recompute` | boolean | no | `false` | Recalculate from review log (slower) |

---

## Media

### `store_media_file`

Upload media to Anki's collection.media folder. Provide exactly one of `data`, `path`, or `url`.

```json
{ "filename": "image.png", "data": "<base64>" }
{ "filename": "audio.mp3", "path": "/path/to/file.mp3" }
{ "filename": "photo.jpg", "url": "https://example.com/photo.jpg" }
```

| Param | Type | Required | Purpose |
|---|---|---|---|
| `filename` | string | yes | Target filename in media folder |
| `data` | string | no | Base64-encoded file content |
| `path` | string | no | Local file path |
| `url` | string | no | URL to download from |

Reference in note fields as `<img src="image.png">` or `[sound:audio.mp3]`.

### `get_media_files_names`

List media files with optional pattern filtering.

```json
{ "pattern": "*.mp3" }
```

### `delete_media_file`

Permanently delete a media file. No undo.

```json
{ "filename": "old_image.png" }
```

---

## GUI Tools

These open Anki's native UI. Only use when the user explicitly requests GUI interaction. For programmatic workflows, use the non-GUI tools instead.

| Tool | Purpose | Key Params |
|---|---|---|
| `gui_add_cards` | Open Add Cards dialog | none |
| `gui_browse` | Open Card Browser with search | `query` (required) |
| `gui_deck_browser` | Open Deck Browser | none |
| `gui_edit_note` | Open note editor | `note_id` (required) |
| `gui_select_card` | Select card in open browser | `card_id` (required) |
| `gui_show_question` | Show question in review mode | none |
| `gui_show_answer` | Show answer in review mode | none |
| `gui_current_card` | Get current card in review | none |
| `gui_undo` | Undo last action | none |

---

## Sources

- [AnkiMCP Server](https://ankiweb.net/shared/info/124672614) (AnkiWeb Add-on)
- [Anki Manual — Searching](https://docs.ankiweb.net/searching.html)
- [FSRS Algorithm](https://docs.ankiweb.net/deck-options.html#fsrs)
