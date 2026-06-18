---
title: FairScan 2.0 released
url: https://lwn.net/Articles/1078242/
published: "2026-06-17T13:26:41Z"
feed: lwn
guid: https://lwn.net/Articles/1078242/
---

# FairScan 2.0 released

[Version
2.0](https://github.com/pynicolas/FairScan/releases/tag/v2.0.0) of the FairScan document-scanning app for Android has been released. The headline feature for this release is the [addition](https://github.com/pynicolas/FairScan/issues/27) of optical-character-recognition (OCR) support using [Tesseract](https://tesseract-ocr.github.io/) to produce PDFs with searchable text from scans. FairScan developer Pierre-Yves Nicolas has written a [detailed
blog](https://fairscan.org/blog/when-a-scan-becomes-a-searchable-pdf/) about adding the feature and explaining why it had not been added previously.

> That looks nice, so why didn't FairScan have it before? That's because FairScan wasn't ready for it: I wouldn't be comfortable if FairScan was giving you wrong text half of the time. To get good results from an OCR engine, you need to provide it a readable image. If it's hard to read for a human, it's certainly also hard to read for an OCR engine.
>
> Over the past year, I worked on different parts of FairScan's automatic processing to transform photos of documents into PDFs that are easy for humans to read:
> - document detection
> - perspective correction
> - shadow reduction
> - brightness and contrast enhancement
>
> All this work on image processing helped FairScan produce clean PDFs and can now also contribute to making text recognition effective.

FairScan is available via [Google
Play](https://play.google.com/store/apps/details?id=org.fairscan.app&pli=1) or [F-Droid](https://f-droid.org/en/packages/org.fairscan.app/).
