---
title: "Designing $unmerge: Reversing the FHIR Patient Merge"
url: "https://www.health-samurai.io/articles/designing-unmerge-fhir-patient?utm_source=rss&utm_medium=feed&utm_campaign=fhir_expert"
date: "2026-05-22"
author: "noreply@health-samurai.io (Ivan Shukshin)"
feed_url: "https://www.health-samurai.io/blog/rss.xml"
---
FHIR has no $unmerge operation. We built one that reverses a merge using Provenance + History API and lets the client decide what to do with data added between the merge and unmerge.
