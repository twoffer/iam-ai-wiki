# `raw/` — third-party source documents

This directory holds **immutable source material that this project did not author.** The wiki under `wiki/` is built by reading these documents; per `CLAUDE.md`, the LLM reads from `raw/` but never modifies it.

These documents come from many different publishers — standards bodies (e.g. the IETF), specification sites, vendor engineering blogs, papers, and other primary sources — and each is stored verbatim, as fetched. Authorship, copyright, and licensing therefore vary file by file and belong to each document's respective upstream source.

## Provenance

Each file carries its own provenance in its header — a `Source:` URL identifying the exact upstream page, and a `Date pulled:` line giving the snapshot date. **Those headers are the authoritative record of where a given document came from and who published it.** To check the origin of any file in this directory, read its first few lines.

## Licensing

Copyright in each document remains with its original author or publisher, and each is included here for reference under whatever terms govern its upstream source. Consult the `Source:` URL in a file's header to determine the applicable license for that document.

**The repository's own `LICENSE` (MIT, © Tony Offer) does not apply to this directory.** It covers the original work in this repo — the curated pages under `wiki/`, the operating schema in `CLAUDE.md`, and the project's own documentation. The contents of `raw/` are attributed to, and licensed by, their respective upstream sources as identified in each file's header.
