# `raw/` — third-party source documents

This directory holds **immutable source material that this project did not author.** The wiki under `wiki/` is built by reading these documents; per `CLAUDE.md`, the LLM reads from `raw/` but never modifies a document's contents.

These documents come from many different publishers — standards bodies (e.g. the IETF), specification sites, vendor engineering blogs, papers, and other primary sources — and each is stored verbatim, as fetched. Authorship, copyright, and licensing therefore vary file by file and belong to each document's respective upstream source.

## Provenance

Each file carries its own provenance in a YAML frontmatter block at the top, containing at minimum a `source` property with the exact upstream URL and a `created` property with the snapshot date:

```yaml
---
source: https://example.org/some-article
created: 2026-08-16
---
```

A `download_url` property appears where the page being cited and the file actually retrieved are different URLs — a PDF behind a landing page, for example. Documents captured with a web clipper additionally carry whatever properties the tool emitted (`title`, `author`, `published`, `tags`, and so on), kept exactly as captured and not restyled to match the repo's own frontmatter conventions. **That block is the authoritative record of where a given document came from and who published it.** To check the origin of any file in this directory, read its frontmatter.

The frontmatter block is the only part of a file here that this project authors; everything below it is the upstream document, verbatim. Binary files cannot carry frontmatter, so a PDF's provenance lives in a markdown sidecar of the same slug — that sidecar records provenance only, and is not a substitute for reading the document itself.

## Licensing

Copyright in each document remains with its original author or publisher, and each is included here for reference under whatever terms govern its upstream source. Consult the `source` URL in a file's frontmatter to determine the applicable license for that document.

**The repository's own `LICENSE` (MIT, © Tony Offer) does not apply to this directory.** It covers the original work in this repo — the curated pages under `wiki/`, the operating schema in `CLAUDE.md`, and the project's own documentation. The contents of `raw/` are attributed to, and licensed by, their respective upstream sources as identified in each file's frontmatter.
