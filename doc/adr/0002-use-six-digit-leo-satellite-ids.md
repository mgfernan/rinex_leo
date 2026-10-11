# Use six-digit LEO satellite IDs

## Status

Accepted

## Context

The RINEX-LEO proposal uses NORAD catalog numbers as LEO satellite IDs.
NORAD catalog numbers have exceeded the five-digit range, so new objects
require six digits. The existing five-digit RINEX-LEO satellite-ID field cannot
represent these IDs.

## Decision

Use a six-digit, zero-padded satellite ID for LEO satellites in both navigation
and observation records. Declare this incompatible change as RINEX-LEO extension
version 2.00.

## Consequences

- RINEX-LEO navigation and observation readers and writers must use a six-digit
  LEO satellite-ID field.
- Existing RINEX-LEO extension version 1 files retain their five-digit layout
  and are not interpreted as version 2 files.
- MEO GNSS satellite identifiers retain their existing two-digit format.
