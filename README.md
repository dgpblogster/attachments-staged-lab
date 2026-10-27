# Attachments staged lab

A lab for the **modern Attachments control** in canvas apps (preview), built around the one
fact that explains every surprise it hands you: a file in that control is either saved, added
but not saved, or marked for removal, and only one of those is in your data source.

Companion to [Staged, Not Saved](https://www.theworkbench.blog) on
[The Workbench](https://www.theworkbench.blog).

> **Preview feature, and it writes files.** Point this at a developer environment, never
> production. Enable modern controls and themes in the app first.

> **Not yet run.** The control definition and both formulas are Microsoft's documented
> examples, reproduced here so they are in one place with the traps annotated. The experiment
> table in [STATES.md](STATES.md) is a test plan, not a report. If you run it and something
> behaves differently, say so in the blog comments and this repo gets corrected.

## What is here

| Path | What it is |
|---|---|
| [`STATES.md`](STATES.md) | **Start here.** The three states, what each costs you, and six experiments with expected results. |
| [`controls/Attachments1.pa.yaml`](controls/Attachments1.pa.yaml) | The control definition for the SharePoint attachment-column shape: 5 files, 10 MB each, PDF and Word only, disabled when no record is selected. |
| [`formulas/save-sharepoint-attachments.power`](formulas/save-sharepoint-attachments.power) | Save button for a SharePoint attachment column. The `Attachments` output goes straight to `Patch`. |
| [`formulas/save-dataverse-file-column.power`](formulas/save-dataverse-file-column.power) | Save button for a Dataverse **file column**, which takes one file as a `FileName` / `Value` record rather than the whole table. |

The control definition and the two formulas come from Microsoft's own documentation for this
control. What is mine is the annotation: the comments marking where each one will bite, and the
experiment table.

## The short version

The classic Attachments control only worked inside a form. This one works outside one, and its
`Attachments` output passes straight to `Patch`. That is the capability. The price is that
staging and saving are now two separate things you have to keep straight, and three of the
control's own outputs will mislead you about which has happened:

- `AttachmentsCount` excludes files marked for removal.
- `IsUploading` reports in-memory activity, **not** a completed save.
- `OnError` fires when *adding* a file fails, never when saving does.

Details and the full list in [STATES.md](STATES.md).

## Using it

1. Enable modern controls and themes in a canvas app in a developer environment.
2. Connect a SharePoint list with attachments enabled, and add a gallery named
   `RequestsGallery` over it.
3. Add the control from `controls/Attachments1.pa.yaml`, or insert a modern Attachments
   control named `Attachments1` and set the properties it lists.
4. Add a **Save attachments** button and paste
   `formulas/save-sharepoint-attachments.power` into its `OnSelect`.
5. Work through the experiments in [STATES.md](STATES.md) in order.

For the Dataverse file column shape, set the control's `Items` to `Blank()`,
`MaxAttachments` to `1`, and use the other formula instead.

## Contributing

This repository is a publishing surface for the blog rather than a community project, so it
does not accept pull requests. If something here does not match what you observe, the comments
on the related blog post are the right place, and that is where the next version of this comes
from.

## License

MIT. See [LICENSE](LICENSE).
