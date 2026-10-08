# Satellite ID naming

As part of the discussion during the IGS 2024 Workshop splinter session on RINEX
format, a critical point that was discussed was the naming of the Satellite ID
that could be common to all *EX files (RINEX, ORBEX, ...). A possible format
for the satellite ID that could cover more than 26 constellations as well
as more than 10000 satellites (i.e. Starlink constellation) could be the following:

- Constellation ID: Two alphanumeric digits (taking as a starting point the
codes specified for the NMEA standard)
- Satellite ID: 5 numerical digits could cover up to 99.999 satellites.

This could be mapped to a 32bit structure (in C) that could represent the satellite

```c
struct satellite {
    uint16_t constellation_id;  // Up to 255 constellations
    uint16_t svn;  // Up to ca. 65k satellites
};
```

This is a separate, exploratory proposal and is not the satellite-ID format
specified in [`rinex_leo_format.md`](./rinex_leo_format.md), which currently
uses a one-character constellation type and a five-digit satellite ID for LEO.
The two approaches remain to be reconciled.

## Constellation ID

Taking as a starting point the 2-digit characters, that could be based on the
[NMEA Talker ID](https://gpsd.gitlab.io/gpsd/NMEA.html#_talker_ids)

|NMEA Talker ID| Numerical ID | Constellation                                         |
|:------------:|:------------:|:------------------------------------------------------|
| 00           | 0            | Undefined                                             |
| GA           | 2            | Galileo Positioning System                            |
| GB           | 3            | BeiDou (China)                                        |
| GI           |              | NavIC, IRNSS (India)                                  |
| GL           | 4            | GLONASS, according to IEIC 61162-1                    |
| GN           |              | Combination of multiple satellite systems (NMEA 1083) |
| GP           | 1            | Global Positioning System receiver                    |
| GQ           |              | QZSS regional GPS augmentation system (Japan)         |

## Satellite ID

For satellite ID, a 5 digit numeric ID can be proposed, with a valid range
between 0 and 65535 (so that it fits in a `uint16_t` type)
