---
title: "Google Cloud Storage and CORS"
date: 2022-12-21T00:19:44-05:00
draft: false
---

Some time ago I made a form for players to report exploiters in a Roblox game. Files are uploaded
to Google Cloud Storage directly using signed links because of Cloudflare's 100MB request size
limit. R2 did not exist at the time either, so I was stuck with this as everything else was already
on GCP. With a fair amount of testing done I believed everything was fine. Then user complaints
started coming in.

There are a various error messages that can be displayed when trying to report a user, ranging from
the username not existing to the file being too large or unsupported. What users were seeing,
however, were not any of those. I had made some basic error handling for failed video uploads, but
it wasn't much since there is no inbuilt XML parser for JS. But it didn't work, because as I found
out, GCS does not return CORS headers on error responses. [Even worse is that this is intentional](
https://issuetracker.google.com/issues/113859351). This also means that I can't log said error
responses with Sentry, so actually trying to debug these issues is nearly impossible.

So, for now, any video upload-related errors simply return "Unknown Error" and I remain clueless as
to why this happens. Hopefully, Google will change this at some point so that I don't have to chase
down users and ask them to do things that are basically a foreign language to them. But until then,
oh well.