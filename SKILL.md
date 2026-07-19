---
name: devlog
description: A file-based single source of truth development approach. Trigger for `/devlog`, `devlog`, `use devlog to...`
---

# Devlog protocol 

## File as single source of truth -- log first, every single invocation

The user converses through `$target_doc` (default is `devlog.md`), not the terminal. Before doing anything else:

1. **Resolve `$target_doc` in the repo root.** 

If it doesn't exist, create it, then prepend one line on top: `* follow instructions from the installed agv5 skill's actions/devlog_protocol.md for protocols to operate this file.` Add an empty `STATUS` block and a round using the template below to hold the user's request.

If it exists, read the **STATUS block + the last ask/reply round** — that is sufficient to be current; the full history lives in `devlog.archive.md` and is optional reference, not required reading.

2. **Answer in the file, not in chat.** Append a Reply block (template below). The Reply is the real deliverable — everything you'd otherwise explain in the terminal goes there. Terminal output for a normal round is exactly one line: `$target_doc updated`. Do not restate or summarize the Reply again in terminal. Highlight all questions you need answers from the user, or try to batch questions at one location so it's easier for user to see and answer. Must provide suggested default answers to each question you ask.

3. **Context-saving check:** if the opening prompt points at a self-contained file (`continue working on @$target_doc`, `pick up @notes.md`) and the session still carries old turns, remind the user ONCE that `/clear` + re-reading from disk is cheaper. Skip if the session is fresh.

4. **Everything you author is in $output_language (default English)** — replies, docs, commit messages, comments — regardless of the ask's language. The one exception: the verbatim Ask quote stays in whatever language the user wrote.

5. After start running this protocol, when user says `continue`, `next` or similar words, it means they have modified $target_doc so re-read the content and continue working.

### Round template

```
---

# → Ask / A-NNN

+ <user's requests, copied VERBATIM — never paraphrased, trimmed, or cleaned up; it's their record of what they asked. Ignore throwaway lines like "continue">

# ← Reply / A-NNN
* _<YYYY-MM-DD HH:MM:SS> (<model-version/effort>)_
* _state: code <commit hashes this reply describes> (<N> tests, <M> scenarios)_

## <short heading for this round>

<full write-up: what you did, decided, or are asking back>

---

# → Ask / A-NNN+1

+ <placeholder>
```

Filling rules:

- **Ask ids are stable and sequential** (`A-001`, `A-002`, …). Never renumber. Replies may cite asks by id.

- Often the user has already typed the ask into the file before invoking you — leave their text untouched, stamp the `A-NNN` id if missing, and append the Reply under it.

- **Timestamp:** Asia/Taipei, obtained by shelling out (`TZ='Asia/Taipei' date '+%Y-%m-%d %H:%M:%S'`) — never guessed. Only the Reply line gets a timestamp.

- **Model version** (e.g. `fable-5/high`, `opus-4.8/xhigh`, `gpt-5.6-terra/medium): stamp truthfully; it's used for quality tracking.

Examples: `2026-07-18 04:55:14 (gpt-5.6-sol/medium)` ← Note: full model name and version, followed by effort, when using claude it might be: (Fable-5/high)

- **The `state:` line** names the code commits the reply describes plus the test/scenario counts at that moment, so every reply's claims are checkable against git later. (The devlog commit that records the reply lands after it — the next session finds it in `git log`.)

- One round covers one coherent unit of conversation; don't log routine tool noise.

### STATUS block — fold after every reply

`$target_doc` opens with a STATUS block using following template: 

```STATUS template
# STATUS
<content here>

---
```

Status contains current commit, test/scenario counts, what's proven, open items, next actions — **≤ 20 lines, rewritten (not appended to) at the end of every reply.** It is the projection of the whole history; a resuming agent reads STATUS + the last round and is current. Anything in older prose that disagrees with STATUS is stale by definition.

### Archive policy

- History is **append-only**: never rewrite or delete old rounds.

- When an era closes (a natural milestone/commit boundary), move its rounds **verbatim** to `$target_doc.archive.md` (default is `devlog.md`), keep their ask ids and state lines, and keep the era map at the archive's top current.

The live file holds only STATUS, references, and the current era's rounds — keep it under a few hundred lines.

- The $target_doc is the HUMAN dialogue record; machine truth lives in each run's event log (`events.jsonl`, rendered by `agv5/viewer.js`). The $target_doc persuades; the event log proves. Never confuse the two.

- Single writer: never run two sessions appending to the $target_doc concurrently.
