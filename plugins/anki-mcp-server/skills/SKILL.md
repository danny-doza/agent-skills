---
name: anki
description: >
  Use this skill whenever Anki flashcards are involved in any way — creation, review, editing, deck
  management, or study sessions. Trigger on phrases like "make flashcards", "add to Anki", "create
  cards", "study with Anki", "review cards", "add a deck", or whenever a learning context implies
  spaced repetition. This skill uses the mcp__anki__* tools from the ankimcp/anki add-on
  (AnkiWeb ID 124672614) — NOT the older AnkiConnect protocol.
---

# Anki MCP Skill

Interface for the **ankimcp MCP server** (AnkiWeb add-on 124672614, `@ankimcp/anki`).
Tools are prefixed `mcp__anki__*` and communicate with Anki over AnkiConnect on port 8765.

---

## Tool Reference

| Tool | Purpose |
|---|---|
| `mcp__anki__sync` | Sync with AnkiWeb |
| `mcp__anki__list_decks` | List decks; `include_stats: true` for card counts |
| `mcp__anki__create_deck` | Create a deck; supports `Parent::Child` nesting |
| `mcp__anki__model_names` | List available note types |
| `mcp__anki__model_field_names` | Get field names for a specific model |
| `mcp__anki__model_styling` | Get CSS for a model |
| `mcp__anki__create_model` | Create a custom note type |
| `mcp__anki__add_note` | Add a single note |
| `mcp__anki__add_notes` | Batch-add up to 100 notes (preferred over add_note) |
| `mcp__anki__find_notes` | Search via Anki query syntax → returns note IDs |
| `mcp__anki__notes_info` | Get full note data (fields, tags, CSS) from IDs |
| `mcp__anki__update_note_fields` | Modify fields on an existing note |
| `mcp__anki__delete_notes` | Delete notes (requires `confirmDeletion: true`) |
| `mcp__anki__tag_management` | Add, remove, replace, or list tags |
| `mcp__anki__card_management` | Suspend, bury, move, flag, set due date |
| `mcp__anki__get_due_cards` | Get the next due card from a deck |
| `mcp__anki__present_card` | Show a card's question or answer |
| `mcp__anki__rate_card` | Submit a rating (1=Again, 2=Hard, 3=Good, 4=Easy) |
| `mcp__anki__store_media_file` | Upload media via base64, file path, or URL |
| `mcp__anki__get_media_files_names` | List media files by pattern |
| `mcp__anki__delete_media_file` | Delete a media file |

---

## Creating Notes

### Sequence

1. **Check models** — `model_names()`, then `model_field_names(model_name)` to confirm exact field names
2. **Check/create deck** — `list_decks()` or `create_deck()` (safe to call on existing deck)
3. **Add notes** — use `add_notes` for batches; each note's `fields` keys must exactly match the model

### Built-in Models

| Model | Fields |
|---|---|
| Basic | `Front`, `Back` |
| Basic (and reversed card) | `Front`, `Back` — generates two cards |
| Cloze | `Text` (use `{{c1::...}}`), `Extra` |

### add_notes — Call Shape

```json
{
  "deck_name": "My Deck",
  "model_name": "Basic",
  "notes": [
    { "fields": { "Front": "Question", "Back": "Answer" } },
    { "fields": { "Front": "Question 2", "Back": "Answer 2" } }
  ],
  "tags": ["topic", "subtopic"]
}
```

Cloze example:
```json
{
  "deck_name": "My Deck",
  "model_name": "Cloze",
  "notes": [
    { "fields": { "Text": "The speed of light is {{c1::299,792,458}} m/s.", "Extra": "In a vacuum." } }
  ]
}
```

---

## Review Session

```
1. sync()
2. get_due_cards(deck_name)         → card_id
3. present_card(card_id)            → show question
4. [user answers]
5. present_card(card_id, show_answer: true)
6. Suggest rating 1–4, wait for user confirmation
7. rate_card(card_id, rating)
8. Repeat from 2
```

Never call `rate_card` without explicit user confirmation.

---

## tag_management and card_management

Both tools take a `params` object keyed by action.

**tag_management:**
```json
{ "action": "add_tags",     "notes": [1234567890], "tags": "topic subtopic" }
{ "action": "remove_tags",  "notes": [1234567890], "tags": "old-tag" }
{ "action": "replace_tags", "notes": [1234567890], "tag_to_replace": "old", "new_tag": "new" }
{ "action": "get_tags" }
{ "action": "clear_unused_tags" }
```

**card_management:**
```json
{ "action": "suspend",      "cards": [1234567890] }
{ "action": "unsuspend",    "cards": [1234567890] }
{ "action": "bury",         "cards": [1234567890] }
{ "action": "change_deck",  "cards": [1234567890], "deck": "Target Deck" }
{ "action": "set_due_date", "cards": [1234567890], "days": "0" }
{ "action": "forget_cards", "cards": [1234567890] }
{ "action": "set_flag",     "cards": [1234567890], "flag": 1 }
```

---

## find_notes Query Syntax

```
deck:MyDeck              notes in a specific deck
tag:topic                notes with tag
is:due                   cards due for review
is:new                   cards never reviewed
added:7                  added in last 7 days
front:keyword            front field contains keyword
flag:1                   red-flagged cards
```

Combine with spaces (AND) or `OR`:
```
deck:"My Deck" tag:physics OR tag:chemistry
```

---

## Gotchas

- **Field name mismatch** — a note silently fails if field keys don't exactly match the model. Always verify with `model_field_names` first.
- **Duplicates** — `add_notes` with `allow_duplicate: false` (default) skips duplicates and reports them without failing the batch.
- **Update while browsing** — `update_note_fields` fails if the note is open in Anki's card browser. The user must close or navigate away first.
- **Deletion is permanent** — pass `confirmDeletion: true` explicitly; there is no undo.