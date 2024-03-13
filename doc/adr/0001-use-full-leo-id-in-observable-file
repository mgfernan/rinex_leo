# Use full LEO id in observable file

## Status

The initial version of the observable file proposed the usage of a dictionary to 
solve the problem of the 2-digit  limit for the GNSS satellite id: LEO-PNT constellations
may be potentially numerous and therefore, albeit unlikely, there may be the possibility that more than
100 satellites for constellations such as Starlink are visible.

This dictionary was specified in the header of the RINEX file and using Event flags

## Context

The problem of the dictionary is that needs maintenance and therefore is prone to
error.

## Decision

It is proposed for the simpler solution of using the full LEO Id number (5 chars)
in the RINEX observation file

## Consequences

This decision has the following consequences:

- No dictionary is needed anymore and therefore the logic to read/write RINEX observable
files is simplified
- A specific format for the observable line will be required for LEO-PNT and therefore
the RINEX reader/writer will have to act differently (i.e. use another parser) depending
on the first letter of the line.
