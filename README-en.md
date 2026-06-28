# Metadata OPSEC: How Not to Leak Your Identity When Sharing Files

*A guide to the data hidden inside the files you send — and how to keep it from giving you away.*

**Version:** 1.0 · **Last reviewed:** June 2026

> **What this guide is.** A practical, threat-model-driven guide to the metadata that rides along inside ordinary files — photos, PDFs, documents, logs, screenshots — and that can reveal your location, your device, your software, your real name, and your habits to anyone you share those files with. The recurring idea: **a file is not just its visible content; it is also a quiet record of how, when, where, and with what it was made.**
>
> **This guide is tool-agnostic.** It teaches the *decision framework* — what to look for, why it matters, and what no tool can fix. The tools are implementations of that framework, and they change over time; this guide points to established offline tools (exiftool, mat2, and others; see §6) as today's means. The framework is permanent; the tooling is not.

> **A note on scope.** This guide is defensive: it exists to help people avoid leaking their own identity. Removing metadata from your own files before sharing them is good hygiene, not evasion. It is not legal advice, and it does not cover stripping metadata to defeat a legitimate investigation — that is a different thing with different consequences.

---

## Table of contents

1. [What metadata is](#1-what-metadata-is)
2. [Why EXIF can leak your location](#2-why-exif-can-leak-your-location)
3. [Risks in images, PDFs, DOCX, logs, screenshots, video, and audio](#3-risks-in-images-pdfs-docx-logs-screenshots-video-and-audio)
4. [Which metadata is dangerous — by threat model](#4-which-metadata-is-dangerous--by-threat-model)
5. [How to review before sharing](#5-how-to-review-before-sharing)
6. [How to clean without damaging the original](#6-how-to-clean-without-damaging-the-original)
7. [What a tool cannot clean](#7-what-a-tool-cannot-clean)
8. [Checklist before sending files](#8-checklist-before-sending-files)
9. [Worked cases](#9-worked-cases)

---

## 1. What metadata is

Metadata is *data about data*: information a file carries about itself, separate from the content you see when you open it. A photo shows you an image; its metadata records the camera, the lens, the exposure, the date, the editing software, and — often — the exact GPS coordinates where it was taken. A document shows you text; its metadata may record the author's name, the organization, the template, the revision history, and the username of whoever last saved it.

This information exists for legitimate reasons. Cameras embed settings so photo software can organize and correct images. Office suites track authorship for collaboration. The problem is not that metadata exists; it is that **it travels with the file by default, invisibly, and most people never look at it before they share.**

Two distinctions worth holding onto:

**Embedded vs. sidecar.** Most dangerous metadata is *embedded* — written inside the file itself, so it follows the file everywhere. Some is *sidecar* (a separate file alongside the original); that is easier to notice and drop. This guide focuses on embedded metadata, because that is the part that surprises people.

**Content vs. container.** Stripping metadata changes the container, not the content. A photo with its GPS removed is still the same photo. This matters for §6: cleaning well means removing the hidden record without damaging the thing you actually meant to share.

---

## 2. Why EXIF can leak your location

EXIF (Exchangeable Image File Format) is the metadata standard embedded in most photos taken by phones and cameras. It is the single most common way people leak their physical location without realizing it, so it deserves its own section.

When a phone with location services takes a photo, it can write the **GPS coordinates** — latitude, longitude, sometimes altitude — directly into the image's EXIF. Those coordinates are often precise to within a few meters. A single photo, shared once, can reveal:

- **Where you live**, if it was taken at home.
- **Where you work, sleep, or regularly are**, if you share photos over time — the pattern of coordinates is more revealing than any single one.
- **Where a "private" location is** — a source's safehouse, a shelter, a meeting point — even when the visible image is innocuous.

Beyond GPS, EXIF typically also carries the **camera make and model** (and on phones, effectively the device model), the **original date and time** the photo was taken, the **software** used to edit it, and sometimes **author or copyright fields** containing a real name. None of these locate you the way GPS does, but together they build a fingerprint: two "anonymous" photos sharing the same device model, editing software, and time pattern can be linked to the same person.

The trap is that **none of this is visible when you look at the photo**, and most sharing feels casual. Social platforms usually strip EXIF on upload (though not always, and not predictably) — but **direct sharing does not**: a photo sent over email, a messaging app that preserves originals, a file drop, or a cloud link very often carries the full EXIF intact. The moment you bypass a platform's re-encoding, you are back to shipping raw coordinates.

> A useful instinct: treat every photo you took yourself as if it has your location stamped on it, until you have confirmed otherwise. That is the safe default, and it is usually true.

---

## 3. Risks in images, PDFs, DOCX, logs, screenshots, video, and audio

Different file types leak different things. Knowing the shape of each risk is what lets you check the right places instead of trusting a single tool blindly.

**Images (JPEG, PNG, HEIC, TIFF).** The EXIF story above (GPS, device, timestamps, software, author). PNG screenshots are usually GPS-free but can carry text chunks with software and timestamps. HEIC (modern iPhone photos) carries rich EXIF including location.

**PDFs.** PDFs have a document information dictionary and often XMP metadata: **author, creator, producer** (the software that generated it), creation and modification dates, and sometimes the title or keywords. A PDF "exported from" a named template or account can carry that name. Worse, PDFs can embed **fonts, attachments, and layers**, and — critically — a PDF that looks like a clean export may still contain **earlier content that was visually covered but not removed**. Be blunt about this: **drawing a black rectangle is not redaction.** If the text can still be selected, searched, copied, extracted with a tool like `pdftotext`, or recovered from layers and annotations, it was *not* redacted — it was merely hidden. True redaction removes the underlying data and then flattens the file (see §6 and §7).

**DOCX (and modern Office formats).** A DOCX is actually a ZIP archive of XML files. Inside, `docProps/core.xml` and `docProps/app.xml` record **author, last-modified-by, company, revision number, total editing time**, and timestamps. The "last modified by" field in particular has burned many people: it is the username of whoever last saved the file, and it is rarely what they intended to publish. But the key point is broader: **Office files are containers, and cleaning document properties is not enough.** Hidden data hides in many places — comments, tracked changes, annotations, headers and footers, hidden text, custom XML, embedded images, and speaker notes in presentations. Inspect all of those, not only `docProps`. (Microsoft's built-in Document Inspector finds and removes most of these categories; run it on a *copy*, because some removed data cannot be restored.)

**Logs.** Logs are metadata-as-content: they routinely contain **usernames, internal hostnames, IP addresses, file paths (which include your home-directory username), API tokens, session identifiers, and timestamps** that reveal time zone and activity patterns. A consultant who shares a raw log to demonstrate a problem can hand over the client's internal network topology, their own machine's username, and live credentials in one paste. Logs deserve more suspicion than any other file type here, because the sensitive data is woven into the text, not tucked in a header.

**Screenshots.** Screenshots feel safe because "it's just what was on my screen" — but that is exactly the risk. A screenshot can capture **the surrounding UI**: other browser tabs, notification banners with sender names and message previews, the system clock and time zone, the username in a window title or file path, open file names in a taskbar, other people's names in a chat sidebar. The image content *is* the leak. And modern phone screenshots carry their own EXIF-like metadata (timestamp, device) on top.

**Video and audio.** Video files can carry creation time, device and software fields, GPS/location metadata (depending on the source), embedded thumbnails, subtitles, chapters, and container-level metadata. Audio can expose recorder/software fields, timestamps, and — in the content itself — voices, background sounds, and location clues. Stripping container metadata or re-encoding helps with the *headers*, but it does not remove identifying information carried in the audio or video *content*: a recognizable voice, a landmark in frame, a reflection, or ambient sound that places the recording. As with images, the header and the content are two separate problems.

---

## 4. Which metadata is dangerous — by threat model

Not all metadata is equally dangerous, and "strip everything always" is a blunt instinct that wastes effort and can damage files. The disciplined approach is to rank by **what the metadata reveals** and **who you are protecting against** — the same threat-modeling logic as the rest of this repository (see the *Operational Security Primer*).

A rough hierarchy, by impact:

**HIGH — locates or names you directly.**
- GPS coordinates (physical location).
- Real name in author/copyright/last-modified-by fields.
- Credentials, tokens, or session identifiers in logs.
These should be removed before almost any sharing. They don't merely correlate — they identify.

**MEDIUM — fingerprints or correlates you.**
- Device make/model and any serial-like fields.
- Editing software and versions.
- Internal hostnames, IP addresses, file paths (which leak usernames).
These rarely identify you alone, but they link separate files to the same origin, which is how compartmentalization fails.

**LOW — minor, usually harmless.**
- Capture timestamps and time zone (though time zone narrows your location, and timing patterns correlate — so "low" is not "zero").
- Orientation, resolution, color profile.

**UNKNOWN — the most important category in security.**
- A format the tool did not parse, could not fully enumerate, or does not support.
- **"Not detected" does not mean "clean."** Treat an unparsed or unsupported file as *not proven clean*, not as safe. This is the category that catches people who ran a tool, saw nothing, and shipped the file. When in doubt, re-encode or convert to a format with no metadata (or plain text), rather than trust an empty scan.

The threat model decides the threshold:

- For an **ordinary user** sharing holiday photos, removing GPS is the high-value move; the rest rarely matters.
- For a **journalist or activist**, the threshold drops hard: device fingerprints and timestamps that link files, and any field naming a source, all become HIGH, because the adversary is actively trying to correlate.
- For a **consultant**, logs and document author fields dominate: the risk is leaking the client's internals or your own identity across deliverables.

The point of ranking is not to relax — it is to spend your attention where the damage is, and to avoid the false security of "I ran a tool once" without knowing what mattered.

---

## 5. How to review before sharing

Reviewing means *looking before you send*, deliberately, as a habit — not trusting that "it's probably fine." A workable routine, in order of effort:

**Build the instinct first.** Before any tool, internalize the default from §2: assume your own photos carry location, assume your own documents carry your name, assume your own logs carry credentials. Reviewing is confirming or removing those assumptions, not discovering them from scratch.

**Inspect, don't guess.** Use a tool (exiftool, mat2's `--show`, or another trusted metadata viewer) to *see* what a file actually contains before deciding. The goal of a scan is to turn "probably fine" into a concrete list: this file has GPS, that device model, this author name. You cannot make a good decision about what to remove until you can see what is there.

**Check the file type's known weak spots (§3).** For an image, look for GPS and author. For a DOCX, look at `docProps` for author and last-modified-by, and check for tracked changes and comments. For a PDF, look at producer/author and ask whether anything was visually redacted but not removed. For a log, read it as text and hunt for usernames, paths, IPs, and tokens. For a screenshot, look at the whole frame, not just the part you meant to capture.

**Review the visible content too, not only the headers.** This is the step tools cannot do for you. A scan finds EXIF GPS; it does not notice that the photo itself shows your street sign, that the screenshot includes a notification with a real name, or that the "redacted" PDF still has selectable text under the black box. Metadata review and content review are two passes, and you need both.

**Decide against your threshold (§4), then act.** Knowing what is there and who you're protecting against, decide what must go. Then clean — carefully (§6).

---

## 6. How to clean without damaging the original

Cleaning metadata is the moment most likely to go wrong, because the act of rewriting a file can corrupt it, strip something you needed, or — worse — give you false confidence that it worked when it didn't. The discipline here comes down to one principle:

> **Safe workflow — the procedure, in order:**
> 1. **Make a copy.** Never work on the original.
> 2. **Inspect the copy** — see what metadata is actually there.
> 3. **Review the visible content** — the pixels/text, not just the headers.
> 4. **Clean the copy.**
> 5. **Re-scan the cleaned copy** — confirm the metadata is gone.
> 6. **Verify manually** — judgment the tool can't apply.
> 7. **Share only the cleaned copy.**
> 8. **Keep or destroy the original** according to your threat model and obligations. If the file is evidence, a business record, client material, or part of a legal or compliance process, **preserve the original** and work only on a redacted copy — destroying it may itself be a problem (see the scope note at the top of this guide).

The single principle behind all eight steps:

> **Never modify the original by default.** Always produce a *cleaned copy*, keep the original offline, and share only the copy. A tool that edits in place is one accidental flag away from destroying your only version.

Principles for cleaning safely:

**Work on a copy; preserve the original.** Keep the untouched original somewhere you control and never share it. Operate on a duplicate. This is non-negotiable: if cleaning corrupts the file or removes too much, you have lost nothing.

**Prefer removal over editing.** For most files, the safe operation is to *strip* the metadata block entirely (e.g., remove the EXIF segment from a JPEG) rather than blank out individual fields, which can leave residue. The right approach reconstructs the file without the metadata block rather than overwriting fields in place.

**Re-encode when you need certainty — but verify.** For images where you want to be sure nothing hidden survives, re-encoding (decoding the pixels and saving a fresh file) can discard the original metadata — *if your re-encoding flow does not copy metadata into the output*. Many tools will preserve or re-attach metadata when asked, and complex formats can keep container or stream metadata through a re-encode. So treat re-encoding as a strong option, not an automatic guarantee, and re-scan the result. For screenshots and photos shared with strangers, the trade (slight quality loss for a clean output you then verify) is usually worth it.

**Verify after cleaning — do not assume.** Re-scan the cleaned copy. The most common failure is believing a tool worked when a field survived, or when a second metadata block (e.g., XMP alongside EXIF) was missed. Cleaning without verifying is how people ship "cleaned" files that still carry GPS.

**Flatten only after removing — flattening alone is not redaction.** For PDFs and documents where earlier content or tracked changes may persist, exporting/printing to a fresh flattened PDF, or copying the final text into a clean document, removes buried history. But flattening *helps only after the sensitive content has actually been removed*: flattening a black box over live text can still leave the text recoverable, depending on the workflow. Always verify by trying to select, search, copy/paste, and `pdftotext`-extract the supposedly-removed content — and check in a second viewer. PDF is treacherous; confirm, don't assume.

**Beware in-place and batch operations.** The two highest-risk actions are editing the original directly and cleaning a whole folder at once. If you do either, do it on copies, and verify a sample. A well-designed tool requires an explicit flag and confirmation before it will touch an original.

### Tools that implement these workflows

These workflows are tool-agnostic, but they have to run on *something*. Established, offline, open-source tools cover most needs today. None is perfect; each has limits, which is exactly why the decision framework above matters more than the tool.

- **exiftool** — the long-standing, cross-platform (Windows/Mac/Linux) Swiss-army tool for *reading, writing, and erasing* metadata across an exceptionally wide format range. Best for *inspection* (seeing what's there) and surgical edits. When writing, it preserves the original by default, renaming it with `_original` appended (use `-overwrite_original` to suppress that). Still: work on your own copies and verify the result before deleting any backup.
- **mat2** (Metadata Anonymisation Toolkit 2) — a defensive *removal* tool that, by design, writes a `.cleaned` copy and never touches your original, and only shows the metadata it considers revealing. Excellent for one-shot "strip and go." Note its maintenance status: the upstream 0xacab repository is archived and read-only, so it is best treated as stable and finished rather than actively maintained — verify your distro's package, any active forks, and overall maintenance status before relying on it for high-risk workflows, and verify outputs rather than assuming completeness. (A GUI front-end, Metadata Cleaner, uses mat2 under the hood for those who avoid the terminal.)
- **PDF tools** (`pdfinfo`, `pdftotext`, `qpdf`, and viewers with real redaction support) — for inspection, extraction checks, structural inspection, and verification. `pdftotext` is how you *check* whether supposedly-removed text is still there; `qpdf` inspects and restructures. Do not treat any of these as automatic redaction tools.
- **ffmpeg** — for re-encoding or remuxing images, audio, and video to strip or avoid carrying metadata when configured correctly; one of the most practical routes for common video-metadata workflows, but verify afterward (it does not touch content, burned-in subtitles, or every stream/chapter/attachment).
- **Office / LibreOffice** — to export a clean copy of a document and to remove tracked changes, comments, and `docProps` author fields.
- **ripgrep / scripts / manual review** — for logs, where the sensitive data is *content*, not a header, and must be read and scrubbed by a human (see §7).

A reference matrix of where each risk lives, what addresses it today, and how much you can trust the result:

| File type | Tools today | Confidence | Does NOT solve |
|---|---|---|---|
| JPEG / HEIC | exiftool, mat2, re-encode | High | visible clues in frame, sensor noise |
| PNG / screenshots | exiftool, mat2, crop/re-encode | High (header), Human (content) | everything visible in the capture |
| PDF | qpdf, pdftotext, flattening | Medium | bad redaction, layers, attachments |
| DOCX | Office Document Inspector, LibreOffice | Medium | embedded objects, buried revisions |
| Logs | ripgrep / scripts / manual | Human review required | context-sensitive secrets |
| Video / audio | ffmpeg, exiftool | Medium | voices, surroundings, sound clues, streams/chapters/subtitles |

The last two columns carry the real lesson: not all formats are equally cleanable, and every tool leaves something it does not solve. A header strip on a JPEG is reliable for the *header*; a PDF or DOCX hides history that field-level cleaning misses; a log is *content*, so only a human reading it is trustworthy. **Tool ≠ safety.**

**Minimal command examples.** These are starting points, not turnkey recipes — exact flags depend on the format, and destructive operations vary. **Always run them on a copy and re-scan the result.**

```sh
# INSPECT (read-only, safe):
exiftool file.jpg                       # show all metadata
mat2 --show file.pdf                    # show what mat2 considers revealing
pdftotext file.pdf -                    # dump PDF text to stdout (is "redacted" text still there?)

# CLEAN (writes; work on copies):
exiftool -all= file.jpg                 # strip all metadata (keeps _original backup). May also drop color profiles etc.; verify the visual output
mat2 file.pdf                           # writes file.cleaned.pdf, original untouched
ffmpeg -i in.mp4 -map_metadata -1 -c copy out.mp4   # strip common container metadata; verify streams/chapters/subtitles separately

# VERIFY (read-only): re-run the INSPECT commands on the cleaned copy.
```

> **A hard rule on online cleaners.** Online "remove metadata" websites are acceptable **only for low-sensitivity files**. Uploading a file to a web cleaner may strip its local metadata, but it hands the *entire file* to a third party — content included. For anything personal, legal, journalistic, work-related, or involving a source, **use local/offline tools**. Stripping a GPS tag by giving the whole photo to an unknown server is not a win.

A caution that the tools themselves state plainly: a clean *scan* does not mean a *safe* file. mat2's own documentation warns that even when its `--show` finds nothing, the file may still carry metadata — there is no reliable way to enumerate every field in complex formats. So clean defensively even when a scan looks empty, and always verify the result. This humility is the correct posture, and it leads directly into the next section.

---

## 7. What a tool cannot clean

This is the section that separates honest guidance from a false sense of security. A metadata tool removes *metadata*. It does not remove sensitive information that lives in the **content** of the file, and it does not understand what is sensitive to *you*. The limits are real and must be named:

**Sensitive content in the image itself.** No tool stripping EXIF will notice that the photo shows your house number, a street sign, a reflection of your face in a window, a name badge, or a screen in the background. The pixels are the content; the tool only touches the header.

**Visually "redacted" but recoverable data.** A black rectangle drawn over text in a PDF, or a blur that can be reversed, is not redaction — the underlying text or pixels are often still there and selectable or recoverable. A metadata cleaner does not fix this. True redaction means removing the data, then flattening.

**Hidden or covered content in documents.** Text hidden behind images, white-on-white text, content in earlier revisions, cropped-but-embedded portions of images (some formats keep the cropped-away pixels), and speaker notes in presentations. Field-level metadata cleaning leaves all of this.

**The act of sharing itself.** The metadata of *transmission* — your IP address when you upload, the email headers, the account you sent it from, the timestamp of sending — is outside the file entirely. Cleaning the file does nothing about the channel. (This is where this guide hands off to network OPSEC.)

**Correlation across files.** Even perfectly cleaned files can be linked by their *content* style, the time you send them, or the account you use. A tool cleans one file; it does not manage your behavior across many.

**Sensor noise — a fingerprint metadata cleaners do not address.** Many digital imaging sensors leave device-specific noise patterns (PRNU — Photo Response Non-Uniformity) that can *sometimes* be used for source-camera identification: forensic analysis may link images to the specific camera that took them, even after all metadata is stripped, because the pattern lives in the pixel data, not the header. How reliably depends on compression, the camera's processing pipeline, how many images are available, and the analyst's capability — it is a real technique, not magic, and not guaranteed. Metadata cleaners do not address it at all. For most threat models it doesn't matter; for a source whose photos must not be tied to their device, it is a reason to use a dedicated capture device whose outputs are not reused across identities (a dedicated camera still builds a pattern if every shot maps back to the same identity), or to accept that re-encoding does not make an image truly origin-anonymous.

**What is sensitive is your judgment, not the tool's.** A tool can flag GPS as HIGH because GPS is always locating. It cannot know that a particular ordinary-looking timestamp places you somewhere you said you weren't. The final review is human.

The honest framing: a metadata tool is necessary and valuable, and it is **not sufficient**. It helps inspect and remove many common forms of hidden metadata in supported formats; the visible content, the buried history, the transmission channel, and your own judgment are yours.

---

## 8. Checklist before sending files

A short, repeatable pass. Adapt the threshold to your threat model (§4).

**Before sending any file:**
- [ ] Do I actually need to send *this* file, or can I send less (a crop, a summary, a fresh export)?
- [ ] Have I scanned it and *seen* what metadata it contains — not assumed?
- [ ] Is there GPS, a real name, or credentials/tokens? (If yes → remove before sending.)
- [ ] Am I working on a **copy**, with the original preserved untouched?
- [ ] Did I **re-scan the cleaned copy** to confirm the metadata is actually gone?

**For images / screenshots:**
- [ ] GPS removed (or re-encoded to be sure)?
- [ ] Does the visible image show anything identifying (signs, faces, screens, badges)?
- [ ] For screenshots: did I check the *whole frame* — tabs, notifications, clock, usernames, sidebars?

**For PDFs / documents:**
- [ ] Author / last-modified-by / company fields checked and cleared?
- [ ] Tracked changes and comments removed?
- [ ] Anything "redacted" actually removed and flattened, not just covered?

**For logs:**
- [ ] Usernames, file paths, internal hostnames, IPs scrubbed?
- [ ] Tokens, keys, session IDs removed?
- [ ] Timestamps reviewed (time zone / activity pattern acceptable to reveal)?

**Always:**
- [ ] Am I about to upload this to an online cleaner? If the file is sensitive, **stop** — use local/offline tools.
- [ ] Original kept offline; only the cleaned copy shared.
- [ ] If the tool reported nothing, did I treat the file as *not proven clean* rather than safe (UNKNOWN, §4)?

---

## 9. Worked cases

A set of short, illustrative scenarios showing how the threshold shifts with the threat model. They are examples, not prescriptions.

### A journalist sending photos

A reporter photographs a document at a source's location and sends the images to an editor. The visible content is just the document — but the **EXIF GPS would reveal the source's location**, and the device model and timestamps could link these photos to the reporter's other work. Threshold: HIGH on everything. The correct move is to re-encode the images with a workflow that does not copy metadata into the output, verify no GPS survives, check that nothing in the *frame* (a window view, a reflection, a visible address) identifies the place, and send only the cleaned copies. Here the metadata is not a minor hygiene issue; a single coordinate could endanger a person.

### An activist sharing a PDF

An organizer prepares a PDF flyer in an office suite and shares it publicly. The PDF's **producer and author fields carry their real name**, and the document was built from an earlier draft. Threshold: HIGH on identity fields, and a real risk of buried history. The move: clear the author/producer metadata, and — because content may persist from earlier revisions — flatten the document (export to a fresh PDF) rather than trust field-level cleaning. Then confirm the name is gone from the new file. The visible flyer is meant to be public; the author's identity is not.

### A consultant sending logs

A security consultant pastes a log excerpt into a report to demonstrate a finding. The log contains the **client's internal hostnames and IPs, the consultant's own machine username in file paths, and a live session token**. Threshold: HIGH — and the danger is in the *content*, not a header, so no metadata tool alone solves it. The move: read the log as text, scrub usernames, paths, internal hostnames, IPs, and any credentials, replace them with clearly-marked placeholders, and keep only what demonstrates the point. This is the case that most reliably burns technical people, precisely because they trust a "clean" file without reading it.

### An ordinary user sharing screenshots

Someone screenshots a chat to share a funny message with a friend. The screenshot also captures **a notification banner with another contact's name and message preview, the system clock, and a browser tab title revealing what they were doing**. Threshold: mostly LOW, but the incidental capture is the leak. The move: crop to just the relevant part before sharing, glance at the whole frame for names and previews, and — if sharing beyond the friend — strip the screenshot's own timestamp metadata. Low stakes, but the habit of *looking at the whole frame* is what prevents the occasional high-stakes accident.

### A developer filing an open-source bug report

Someone attaches logs and screenshots to a public GitHub issue to get help with a bug. The paste leaks, in one go, **their home-directory username (visible in every file path), the contents of a `.env` file, a live API token, an internal hostname, and the name of a private repository** — none of which they meant to publish, all now in a public, indexed, permanently-archived issue. Threshold: HIGH, and it is the *content* of logs and the *frame* of screenshots that leak, not headers. The move: before posting, read every line; replace usernames, paths, hostnames, repo names, and emails with clearly-marked placeholders; remove any token, key, or `.env` content (and rotate any credential that was already exposed); and crop screenshots to the error, checking the whole frame. Public bug trackers are forever — treat anything you paste there as published.

### A job-seeker sharing a résumé PDF

Someone exports their CV to PDF and uploads it to job boards and emails it to recruiters. The PDF carries, in its metadata, **the author and company fields (sometimes a current employer's name), the template's origin, an old username, and the document's modification history** — and if it was built from a previous version, possibly text from roles or salary notes they thought they had deleted. Threshold: MEDIUM on the identity fields, with a real risk of buried draft content. The move: clear the author/company/producer fields, then export a fresh PDF (or print to PDF) after removing hidden data to drop revision history, and re-scan to confirm; then read the visible document once more for anything carried over from an earlier draft. A résumé is meant to be shared — which is exactly why its hidden fields deserve a look.

---

## References and verification

This guide intentionally separates decision-making from tooling. Tool behavior changes over time; verify commands and supported formats against upstream documentation before relying on them for high-risk workflows. The references below are upstream or primary sources for the tools and guidance mentioned, as of writing.

- **ExifTool** — official site and documentation: `https://exiftool.org/` (a SourceForge mirror exists if the main site is unavailable).
- **mat2** — official repository and documentation: `https://0xacab.org/jvoisin/mat2` (developed with the support of the Tails project).
- **Microsoft Document Inspector** — "Remove hidden data and personal information by inspecting documents, presentations, or workbooks," on Microsoft's official support site (`support.microsoft.com`).
- **FFmpeg** — official documentation: `https://ffmpeg.org/documentation.html`.
- **PDF redaction** — consult your PDF software's official documentation for its *redaction* feature specifically (e.g., Adobe Acrobat's redaction tools), and note the standing guidance from sources such as the NSA on sanitizing documents: true redaction removes the underlying content; it is not the same as drawing over it. Verify by attempting to extract the supposedly-removed content (see §6).

Commands here were written as examples, not guarantees; test them against your installed versions. Always treat upstream documentation as the source of truth over any guide, including this one. If a command or behavior described here does not match what your installed version does, trust your version and the official docs.

---

> **The thread, one more time.** Metadata leaks identity because it travels invisibly and by default. Tools like exiftool and mat2 help inspect and remove many common forms of hidden metadata. But the visible content, the buried history, the sensor noise in the pixels, the transmission channel, and the judgment of what is sensitive to *you* remain human work. Clean the file, yes. But review it like the person whose identity depends on it — because sometimes it does.

---

*This guide is tool-agnostic: it teaches the decision framework, and points to established offline tools as today's implementations. It applies the threat-modeling framework from the Operational Security Primer to one concrete, everyday risk. Defensive in nature; not legal advice.*