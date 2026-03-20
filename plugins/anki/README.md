# anki

## Description

Provides seamless integration with Anki via AnkiConnect, enabling the assistant to interact directly with your Anki collection.

### Capabilities

Use this plugin whenever the user needs to interact with Anki, including:

- Creating, updating, or deleting **notes and cards**
- Reading or modifying **decks and models**
- Searching and retrieving **existing notes/cards**
- Managing **media files**
- Triggering **sync operations**

### Usage

This skill is automatically invoked when a user requests any operation involving Anki or spaced repetition workflows.

Ensure that:
- Anki is running locally
- The AnkiConnect plugin is installed and enabled

### Notes

- Prefer non-destructive operations when possible
- Confirm before performing bulk edits or deletions
- Validate deck/model names before creating new content

## Installation

### Claude Code / Cowork

```bash
claude plugin marketplace add danny-doza/agent-skills
claude plugin install anki@doza-skills
```

### npx skills

```bash
npx skills add danny-doza/agent-skills --skill anki-connect
```
