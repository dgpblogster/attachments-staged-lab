# The three states

A file in the modern Attachments control is in one of three states, and only one of them is
in your data source.

| State | Where it lives | What a save does |
|---|---|---|
| **Saved** | The data source | Nothing to do |
| **Added, not saved** | The control's in-memory state | Writes it |
| **Marked for removal** | Still in the data source | Deletes it |

Adding a file **stages** it. It does not save it. Removing a saved file **marks** it. It does
not delete it. Nothing reaches the data source until `Patch` or `SubmitForm` runs.

## What that costs you

**`AttachmentsCount` excludes files marked for removal.** It reports what the record *would*
hold if you saved right now, not what it holds. Useful, as long as you know which one you are
reading.

**`IsUploading` is not a save confirmation.** It reflects whether files are being added to the
control's in-memory state. Microsoft says, in the same table entry, that it "isn't confirmation
that **Patch** or **SubmitForm** has saved the files." A Save button disabled on `IsUploading`
tells the user the app finished reading their file off disk. It says nothing about whether the
record was written.

**`OnError` fires for add failures, not save failures.** It runs when adding a file fails, such
as when the file exceeds a configured limit. Data-source save failures are a separate concern:
wrap the `Patch` in `IfError`.

**`OnAddFile` fires once per file, not once per batch.** Drop five files and the handler runs
five times. If it writes a row or calls a flow, that is a loop you did not intend. Keep it
idempotent and cheap.

**The limits are guidance, not enforcement.** Control limits do not override the data source's
file-size restrictions or file-type policies, and `AcceptedFileTypes` steers the file picker
rather than replacing validation. A `.pdf` extension has never been a promise about the bytes
behind it.

## Experiments, with expected results

Run these against the SharePoint shape in `controls/` plus
`formulas/save-sharepoint-attachments.power`. Each one isolates a single state transition.

| # | Do this | Expect |
|---|---|---|
| 1 | Add a file. Close the app without saving. Reopen the record. | File is **gone**. Staging is not saving. |
| 2 | Remove a saved file. Watch `AttachmentsCount`. Close without saving. | Count **dropped**, file is **still there**. The count described a future that never happened. |
| 3 | Upload a file with the same name as a saved one. Save. | **Two** files, the new one **suffixed**. Not a replacement. |
| 4 | Set `MaxAttachments` to 5, add 6 files. | Control takes **5** and reports the one it could not add. Check what was accepted *before* saving. |
| 5 | Add a file over `MaxAttachmentSize`, or an extension outside `AcceptedFileTypes`. | File **rejected**, error reported. `OnError` fires. |
| 6 | Add files, then select a different record in the gallery without saving. | Staged edits are **lost silently**. This is the one to design around. |

Experiment 6 is a design obligation rather than a defect. The guidance is to keep the selected
record unchanged while unsaved attachment edits exist, and to give the user a way to save or
discard before navigating.

## What the control does not do

- It does not enforce your file policy.
- It does not replace a saved file on a same-name upload.
- It does not confirm a save through `IsUploading`.
- It does not accept the whole `Attachments` table at a Dataverse file column.
