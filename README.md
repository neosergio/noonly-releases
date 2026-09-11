# Noonly — downloads

Signed and notarised builds of **Noonly**, a native macOS app for keeping a professional
work memory in plain Markdown worklogs.

This repository holds releases only; it contains no source code.

![Noonly showing two days of a worklog: projects grouped in the sidebar, entries under each
day, and the composer at the bottom](screenshot.png)

Two days of work across several projects. The marks under an entry are threads: an orange ring
is work that was opened and is still open, a green check is work that was closed. That is the
difference between a worklog and a diary — the log knows what is still waiting for you.

## Download

**[Latest release](https://github.com/neosergio/noonly-releases/releases/latest)**

Requires macOS 14 or later. A universal binary: Apple Silicon and Intel.

Every build is signed with a Developer ID certificate and notarised by Apple, so it opens
without a Gatekeeper warning. The notarisation ticket is stapled to the app itself, not only
to the disk image, so the first launch works with no network connection.

## Verifying a download

```sh
shasum -a 256 Noonly-0.2.dmg     # compare against the checksum in the release notes
spctl --assess --type execute --verbose=2 /Applications/Noonly.app
```

A correct result reads `accepted` and `source=Notarized Developer ID`.

## The file format

Noonly writes into files you own, so what it writes is documented:
[WORKLOG_FORMAT.md](WORKLOG_FORMAT.md). A worklog is a folder of Markdown files, one per
project, each with a title and a `## Log` section holding dated entries. Nothing else is
required, and the files stay readable and editable without Noonly.

## Feedback

Bug reports and impressions are welcome in
[Issues](https://github.com/neosergio/noonly-releases/issues).
