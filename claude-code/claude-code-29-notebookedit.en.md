# NotebookEditTool: Editing Notebooks

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tooling
> Understand how Claude Code edits Jupyter Notebooks at cell granularity.

---

## NotebookEditTool: Editing Notebooks

.ipynb files are essentially JSON, but semantically they are an ordered set of cells mixing code / markdown, carrying output, metadata, and language information. Claude Code built a dedicated NotebookEditTool for them.

***

### Input Schema

**The operation granularity is cell-level, not whole-file string-level.**

***

### Edit Pipeline

The model needs to modify the .ipynb
⬇
Validate the file extension and permissions
⬇
Read and parse the notebook JSON
⬇
Perform replace/insert/delete by cell_id
⬇
Preserve notebook structure → write back

***

### Summary in One Sentence

> Claude Code doesn't downgrade Notebooks to plain text; instead, for this "code + document" hybrid format it **built a dedicated controlled editing pipeline**, operating at cell granularity and preserving structural integrity.
