# Noonly — downloads

Signed and notarised builds of **Noonly**, a native macOS app for keeping a professional
work memory in plain Markdown worklogs.

This repository holds releases only; it contains no source code.

## Download

**[Latest release](https://github.com/neosergio/noonly-releases/releases/latest)**

Requires macOS 14 or later. A universal binary: Apple Silicon and Intel.

Every build is signed with a Developer ID certificate and notarised by Apple, so it opens
without a Gatekeeper warning. The notarisation ticket is stapled to the app itself, not only
to the disk image, so the first launch works with no network connection.

## Verifying a download

```sh
shasum -a 256 Noonly-0.1.dmg     # compare against the checksum in the release notes
spctl --assess --type execute --verbose=2 /Applications/Noonly.app
```

A correct result reads `accepted` and `source=Notarized Developer ID`.

## Feedback

Bug reports and impressions are welcome in
[Issues](https://github.com/neosergio/noonly-releases/issues).
