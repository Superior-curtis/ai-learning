# FileEditTool: Editing Files in Depth

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tooling
> Understand how FileEditTool implements Claude Code's controlled incremental editing model.

---

## FileEditTool: Editing Files

FileEditTool is responsible for making targeted modifications to existing files. In Claude Code, its value has never been just "replacing a string": first it confirms the file has been read, then it confirms the file hasn't been modified by anyone else, then it confirms the current edit permission allows it, and only then does it generate a patch and write it back.

**FileEditTool embodies Claude Code's controlled editing model.**

***

### Dependent Submodules

It keeps all of these in view at once: diff, the Git perspective, permissions, file metadata, and patch generation.

***

### Edit Pipeline

The model decides to modify an existing file
⬇
Path and permission check
⬇
Confirm the file has been read and is not stale
⬇
Compare old\_string / new\_string → generate patch
⬇
Write back → update diff / diagnostics

***

### Conflict Detection

After Claude reads a file, if the user or a linter modifies it, the system blocks further edits based on the stale snapshot.

***

### Edit vs Write

✏️ FileEditTool
Existing files, targeted replacementemphasizes patches that preserve the original structure📝 FileWriteTool
New files or whole-file overwriteemphasizes writing complete content

***

### Summary in One Sentence

> The most valuable lesson from FileEditTool isn't "how to replace a string", but rather **how Claude Code turns file editing into an engineered operation that comes with read-before-edit, conflict detection, permission control, and patch results**.
