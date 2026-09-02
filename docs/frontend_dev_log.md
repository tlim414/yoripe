---
layout: default
title: "Frontend Dev Log"
permalink: /frontend-dev-log
---

# Frontend Dev Log
This page serves as a logbook for the frontend changes


## Standardize units from image extraction response data
### August 30
Implemented unit normalization
- Created a mapping dictionary (UNIT_ALIASES) to consolidate alternate unit representations into supported frontend options
- Added a utility function to standardize ingredient units returned by the extraction endpoint before they reach the UI, mitigating edge cases where extracted recipes use non-standard or unsupported unit variants.
```text
export const UNIT_ALIASES: Record<string, UnitType> = {
  // Cups
  cup: UNITS.CUPS,
  cups: UNITS.CUPS,
  ...
```


## Add Image Uploading to Backend
### August 28
- Integrated file uploader: Implemented `react-dropzone` to allow image selection via file browser or drag-and-drop.
- Image preview & explicit trigger: Displayed an immediate preview of the selected image alongside its file name. Opted for an explicit "Extract" button rather than auto-submitting on upload, avoiding unnecessary API usage and traffic if a user selects the wrong file.
- Flexible fraction parsing for amounts: Updated the amount input field to accept string representations instead of strict numeric types:
- Updated to regex validation on input change to support integers, decimals, mixed fractions, and slash-separated fractions (e.g., 3, 2.75, 1 1/3, 3/5).
- Updated the "Save" button's disabled state validation to block submission when amount formats are invalid.

## MVP
### July21 - August 14



