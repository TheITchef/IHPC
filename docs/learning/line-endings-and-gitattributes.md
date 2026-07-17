# Line Endings & .gitattributes — what I learned

## The background concept (so future-me isn't lost)

Every line in a text file ends with an invisible marker — a "line ending" — that tells software where one line stops and the next begins. You never see it, but it's a real character (or two) stored in the file.

The catch: different operating systems mark line ends differently.

- **LF** ("line feed", one character, written `\n`) — used by Linux and macOS. Git prefers this internally.
- **CRLF** ("carriage return + line feed", two characters, written `\r\n`) — used by Windows.

The names come from typewriters: *carriage return* (`\r`) moved the carriage back to the left; *line feed* (`\n`) rolled the paper up one line. Windows kept both; Unix systems kept just the one. Same idea, different number of hidden characters.

**What it looks like.** Take two lines of text:

```
Hello
World
```

Stored with **LF** (Unix), the raw bytes are:

```
Hello\nWorld\n
```

Stored with **CRLF** (Windows), the same text is:

```
Hello\r\nWorld\r\n
```

Identical on screen — but every single line has an extra hidden `\r` character in the Windows version. To Git, that's a different file, byte for byte. Multiply across a whole document and Git sees "everything changed," even though you changed nothing.

## What happened

During a session, Git kept flagging files (the Kanban board, the Master Document) as "modified" even when I hadn't changed anything visible. `git diff` showed no content difference — just a warning:

```
LF will be replaced by CRLF the next time Git touches it
```

## What surprised me

The "change" was invisible. My Windows tools (Obsidian, my editor) were saving files with CRLF endings, but Git had stored them with LF. Same text on screen, different bytes underneath — so Git honestly reported the files as changed, because to Git they *were* different. I'd been seeing these phantom "modifications" all session without understanding they were purely a line-ending mismatch, not real edits.

## What I did

Added a `.gitattributes` file to the repo root. This is a plain-text config file (no extension, like `.gitignore`) that tells Git how to handle files. Mine tells Git to **normalise** line endings — store everything as LF inside the repository, and check files out with whatever ending my OS uses — so the phantom "modified" flags stop appearing.

I also marked binary files (`.pdf`, `.docx`, images) as `binary`, which tells Git to never attempt line-ending conversion on them. Converting a binary file's bytes would corrupt it — so this protects the companion documents living in `docs/learning/`.

## What I'd do differently

Add `.gitattributes` at the *start* of a repo, not partway through. I tolerated the line-ending noise for a whole session before fixing it at the source. It's a standard first-day file on any cross-platform project, and I now treat it as part of initial repo setup, not an afterthought.

## One-line takeaway

`.gitattributes` stops Git from seeing invisible Windows/Unix line-ending differences as real changes — add it on day one.