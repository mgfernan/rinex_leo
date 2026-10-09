# RINEX-LEO

This section describes the RINEX-LEO extension proposed by Rokubun. The current
proposal is based on the
[RINEX 4.01 format](https://files.igs.org/pub/data/format/rinex_4.01.pdf).

## File header and versioning

RINEX-LEO files use the standard `RINEX VERSION / TYPE` record to declare the
underlying RINEX version, and add a separate `RINEX-LEO EXTENSION` record to
declare the RINEX-LEO extension version. The namespaced label reduces the risk
of colliding with a header record introduced by a future RINEX release. The two
version numbers identify independent things: a future RINEX release (for
example, RINEX 5 or 6) is reflected in `RINEX VERSION / TYPE`, and does not by
itself change the RINEX-LEO extension version.

The `RINEX-LEO EXTENSION` record is required exactly once in the header of a
RINEX-LEO file. As with other RINEX header records, its label occupies columns
61–80; the extension name and version occupy the data area in columns 1–60.
Place it immediately after `RINEX VERSION / TYPE`. It is an extension header
record, so generic RINEX readers may skip it; RINEX-LEO readers must recognize
and validate it. For example:

```text
     4.01           OBSERVATION DATA    M                   RINEX VERSION / TYPE
RINEX-LEO 1.00                                              RINEX-LEO EXTENSION
```

The extension version uses `major.minor` notation. Increment the major
component for incompatible changes to the RINEX-LEO extension and the minor
component for backward-compatible changes. A reader must not infer the
extension version from the RINEX version; it should report an unsupported
extension version rather than silently interpreting it as a version it
understands.

One of the main objectives of having a RINEX file adapted for LEO is having a
consistency of satellite ID between Navigation and Observation files. This
makes it easy to analyze data (either real or simulated) and obtain various
performance metrics.

## Navigation file

For navigation files, this proposal takes advantage of the fact that the
navigation blocks are preceded by a header (starting with `>`), which allows
certain flexibility in the satellite ID length. Moreover, since the satellite ID
is included in the header, it does not then need to be added in the actual
navigation block (as done for MEO GNSS satellites). This helps to maintain the
lengths of subsequent fields of the `EPOCH / SV CLOCK` line.

The proposal for the extension is as follows (following the same format
description used in the RINEX format):

```text
TYPE / SV           - New Record identifier: '>'                A1
                    - Navigation Data Record Type – 'EPH'       1X,A3
                    - LEO constellation type                    1X,A1
                    - LEO satellite id                          I5

EPOCH / SV CLK      - Void spaces (Sat ID already in header)    3X
                    - year (4 digits)                           1X,I4
                    - month, day, hour, minute, second          5(1X,I2)
                    - SV clock bias; a_f0 (seconds)             D19.12
                    - SV clock drift; a_f1 (sec/sec)            D19.12
                    - SV clock drift rate; a_f2 (sec/sec2)      D19.12

ORBIT - 1           - A DOT (meters/sec)                        4X,4D19.12
                    - C_rs (meters)
                    - Delta n0 (radians/sec)
                    - M0 (radians)

ORBIT - 2           - C_uc (radians)                            4X,4D19.12
                    - e Eccentricity
                    - C_us (radians)
                    - sqrt(A) (sqrt(m))

ORBIT - 3           - toe sow (seconds of week)                 4X,4D19.12
                    - C_ic (radians)
                    - OMEGA0 (radians)
                    - C_is (radians)

ORBIT - 4           - i0 (radians)                              4X,4D19.12
                    - C_rc (meters)
                    - omega (radians)
                    - OMEGA DOT (radians/sec)

ORBIT - 5           - IDOT (radians/sec)                        4X,4D19.12
                    - Delta n0 dot (radians/sec^2)
                    - toe week
                    - reserved

ORBIT - 6           - reserved                                  4X,4D19.12
                    - reserved
                    - TGD
                    - reserved

ORBIT - 7           - ISC_S9C                                   4X,4D19.12
                    - reserved
                    - reserved
                    - reserved
```

An example of a navigation block for a Starlink LEO satellite 2434 is shown
below. This example has been built transforming the TLE elements of the Starlink
satellite found at [Celestrak](https://celestrak.org/NORAD/elements/gp.php?GROUP=starlink&FORMAT=tle) and assuming no orbit perturbations
(see [[Garcia-Fernandez, 2023]](https://arxiv.org/pdf/2401.17767)).

```text
> EPH Z02434
    2023 11 02 11 10 24 4.887547846484e-05 4.303787327448e-12 0.000000000000e+00
     0.000000000000e+00 0.000000000000e+00 1.989675347274e-09 4.642884733583e+00
     0.000000000000e+00 1.811000000000e-04 0.000000000000e+00 2.835776281159e+03
     3.858240000000e+05 0.000000000000e+00 5.882354736496e+00 0.000000000000e+00
     9.288031413876e-01 0.000000000000e+00 1.642393223370e+00 0.000000000000e+00
     0.000000000000e+00 0.000000000000e+00 2.286000000000e+03 0.000000000000e+00
     0.000000000000e+00 0.000000000000e+00 1.027763024283e-09 0.000000000000e+00
    -9.724269021370e-10 0.000000000000e+00 0.000000000000e+00 0.000000000000e+00
```

The one-character `LEO constellation type`, would be specific to each
constellation. Obviously, an "official" list of constellation types would have
to be agreed upon.

The `LEO satellite id` field (5 digits) will identify the Space Vehicle
(constellation dependent, PRN number for MEO satellites). Similarly to the
constellation type, the list of the IDs will have to be agreed upon as well.
In the example provided, the ID has been extracted from the first field of the
TLE entry for this Starlink satellites.

A tentative list of constellations and satellite id's are considered below

| `LEO constellation type`  | Constellation                  | `LEO satellite id` |
|:-------------------------:|:------------------------------:|:------------------:|
| `Z`                       | Centispace                     | NORAD id           |
| `Y`                       | Geely                          | NORAD id           |
| `L`                       | Generic LEO-PNT constellation  | custom             |
| `A`                       | Globalstar                     | NORAD id           |
| `D`                       | Iridium                        | NORAD id           |
| `O`                       | Oneweb                         | NORAD id           |
| `B`                       | Orbcomm                        | NORAD id           |
| `V`                       | Spire                          | NORAD id           |
| `X`                       | Starlink                       | NORAD id           |
| `P`                       | Xona                           | NORAD id           |

## Observable file

The limitation of 2-digit for the satellite ID in the Observation file is a
major blocking point unless a breaking change is introduced in the format.
To address this issue, the full `LEO satellite id` (5 digits with zero padding)
will be used right after the constellation letter. The RINEX reader/writer
will then require customized measurement line parser/formatter for MEO
or LEO.

The format for the observable data record description, following the RINEX 4
format, is:

```text
+----------------------------------------------------------------------------+
|               OBSERVATION DATA FILE - DATA RECORD DESCRIPTION  (LEO)       |
+-------------+-------------------------------------------------+------------+
| DESCRIPTION                                                   |   FORMAT   |
+---------------------------------------------------------------+------------+
| EPOCH record                                                  |            |
|                                                               |            |
|     − Record start identifier : ">"                           |     A1     |
| - Epoch:                                                      |            |
|     - year (4 digits)                                         |    1X,I4   |
|     - month, day, hour, min (2 digits)                        | 4(1X,I2.2) |
|     - sec                                                     |    F11.7   |
|     - Epoch flag [*]                                          |    2X,I1   |
|     - Number of satellites in current epoch                   |     I3     |
|     - (reserved)                                              |     6X     |
|     - receiver clock offset (seconds, optional)               |    F15.12  |
|                                                               |            |
| [*] EPOCH FLAG follow the same definition as RINEX 4.0        |            |
|                                                               |            |
+---------------------------------------------------------------+------------+
| OBSERVATION record for LEO-PNT satellites [**]                |            |
|                                                               |            |
| - Satellite number (LEO system dependent, e.g. NORAD ID)      |    A1,I5   |
| - m fields of observation data in the same sequence as given  |  m(F14.3,  |
|   in the "SYS / # / OBS TYPES" header record                  |            |
| - LLI              | each obs.type (same seq                  |     I1,    |
| - Signal strength  | as given in header)                      |     I1)    |
|                                                               |            |
| [**] Observation records for MEO GNSS are the same as RINEX 4 |            |
|                                                               |            |
+-------------+-------------------------------------------------+------------+
```

An example of data file would be:

```text
> 2022 12 20 12  1  4.6030000  0 30
G23  24718012.436   129894024.173    24718009.543   101216196.673
Z43012    732793.036      732794.384
```

## Observable codes

Another important aspect of the RINEX observation types are the codes that
identify the different measurements.

| GNSS system | Freq. band / frequency | Channel or code| Pseudo range | Carrier phase | Doppler | Signal strength |
|:-----------:|:----------------------:|:--------------:|:------------:|:-------------:|:-------:|:---------------:|
| **LEO**     | L1 / 1575.42           | C/A            | C1C          | L1C           | D1C     | S1C             |
|             | L5 / 1176.45           | C/A            | C5C          | L5C           | D5C     | S5C             |
|             | S[^1][^2] / 2492.028   | C/A            | C9C          | L9C           | D9C     | S9C             |
| **Xona**    | X1 / 1593.3225         | C/A            | C1C          | L1C           | D1C     | S1C             |
|             | X5 / 1190.51625        | C/A            | C5C          | L5C           | D5C     | S5C             |
|             | XC / 5020.3725         | C/A            | C9C          | L9C           | D9C     | S9C             |
| **Globalstar** | S / 2486.1          | -              | -            | -             | D9C     | S9C             |
| **Iridium** | L / 1626.270833        | -              | -            | -             | D6C     | S6C             |
| **Orbcomm** | VHF / 137.2 (centre)   | FDMA, see below| -            | -             | D3C     | S3C             |

Signals of opportunity (Globalstar, Iridium, Orbcomm) only provide Doppler and
signal strength (C/N0, dB-Hz). Band digits `6` and `3` are only meaningful within their own
constellation. The nominal band frequencies can be documented in `COMMENT` header records.

### Frequency slot observable (`F`)

FDMA constellations transmit on several carrier frequencies, so the frequency of
each observation must be recorded. For this purpose, the observable type `F`
(*Frequency slot*) is defined:

| Observable | Description                                     | Units            | Format    |
|:----------:|:-----------------------------------------------:|:----------------:|:---------:|
| `Fna`      | Frequency slot of the carrier, `n` band, `a` attribute | integer slot number | F14.3 (integer value, e.g. `205.000`) |

The frequency is obtained from the slot number as

$$f = f_{c} + \Delta f \cdot F$$

where the centre frequency $f_{c}$ and slot size $\Delta f$ are defined by the constellation:

| Constellation | Observable | Centre frequency $f_{c}$ | Slot size $\Delta f$ | Example                                   |
|:-------------:|:----------:|:------------------------:|:--------------------:|:-----------------------------------------:|
| **Orbcomm**   | `F3C`      | 137.2 MHz                | 2.5 kHz              | `F3C = 205` → 137.2 MHz + 512.5 kHz = 137.7125 MHz |

`F` observables are listed in `SYS / # / OBS TYPES` like any other observable (for
Orbcomm: `B    3 D3C S3C F3C`), are written with the usual `F14.3` field, and carry no
LLI nor signal strength flag. Readers not supporting the `F` observable can ignore it.

Additionally, other authors propose the following bands:

- C-band (e.g. 5020MHz, [De Bast et al., 2023])
- Ka-band (e.g. 11 GHz, [Humphreys et al., 2023]), for compatibility with communication satellites (), albeit this can be greatly affected by channel losses [Prol et al., 2022]

In the navigation block definition, placeholders for the *signal biases* have been reserved, which may
depend on the navigation system. This will may eventually trigger the need of defining custom navigation
blocks for each LEO constellation as well.

[^1]:[Prol et al., 2022]

[^2]:Parallelism to NavIC system, as per [Rinex 4.0 definition](https://files.igs.org/pub/data/format/rinex_4.01.pdf)

### Signal biases: LEO

Based on the convention used in GPS, described in, for instance, Paragraph 20.3.3.3.1.2.1 of GPS [ICD IS-GPS-705H](https://www.gps.gov/technical/icwg/IS-GPS-705H.pdf) (page 78), the single-frequency observables could be obtained by correcting the *dual-frequency ionospheric-free* clock  broadcasted by the navigation message.

For the generic `LEO` constellation (`L`), it is assumed that the clock ($\Delta t_{SV}$) has been computed using the C1C and C5C ionospheric free combination. Therefore, the single-frequency *clocks* could be computed as follows:

- $(\Delta t_{SV})_{C1C} = \Delta t_{SV} - t_{gd}$, where $t_{gd}$ corresponds to the `TGD` parameter of the RINEX file.
- $(\Delta t_{SV})_{C5C} = \Delta t_{SV} - (f_{L1}/f_{L5})^2 \cdot t_{gd}$
- $(\Delta t_{SV})_{C9C} = \Delta t_{SV} - t_{gd} + ISC_{S9C}$, where $ISC_{S9C}$ corresponds to the `ISC_S9C` parameter of the RINEX file

### Header records

Records are written in this order. All header lines are 80 characters long:
columns 1–60 contain the data and columns 61–80 the label.

| Label                  | Content / format                                                                 |
|:-----------------------|:---------------------------------------------------------------------------------|
| `RINEX VERSION / TYPE` | `F9.2,11X,A20,A20`: `4.01`, `OBSERVATION DATA`, `M` (mixed)                      |
| `RINEX-LEO EXTENSION`  | `RINEX-LEO 1.00`                                                                 |
| `PGM / RUN BY / DATE`  | `A20,A20,` date as `yyyymmdd hhmmss UTC`                                          |
| `COMMENT` (optional)   | Free text. `pygnss` documents the band and nominal frequency of each constellation, e.g. `B: band 3, 137.200000 MHz` and `B: F = slot of 2.5 kHz` |
| `MARKER NAME`          | `A60`; receiver/station name                                                     |
| `APPROX POSITION XYZ`  | `3F14.4`; ECEF position of the receiver (m)                                      |
| `SYS / # / OBS TYPES`  | `A1,2X,I3,13(1X,A3)`; one record per constellation (up to 13 types per line; continuation lines start with 6 blanks) |
| `TIME OF FIRST OBS`    | `5I6,F13.7,5X,A3`; first epoch and time system (`GPS` or `UTC`)                  |
| `TIME OF LAST OBS`     | Same format, last epoch                                                          |
| `END OF HEADER`        | Empty                                                                            |

Example (Globalstar `A`, Orbcomm `B` and Iridium `D`):

```text
     4.01           OBSERVATION DATA    M                   RINEX VERSION / TYPE
RINEX-LEO 1.00                                              RINEX-LEO EXTENSION
pygnss                                  20261009 093836 UTC PGM / RUN BY / DATE
A: band 9, 2486.100000 MHz                                  COMMENT
D: band 6, 1626.270833 MHz                                  COMMENT
B: band 3, 137.200000 MHz                                   COMMENT
B: F = slot of 2.5 kHz                                      COMMENT
LEO                                                         MARKER NAME
  4780861.4222   176383.0427  4204318.1078                  APPROX POSITION XYZ
A    2 D9C S9C                                              SYS / # / OBS TYPES
B    3 D3C S3C F3C                                          SYS / # / OBS TYPES
D    2 D6C S6C                                              SYS / # / OBS TYPES
  2026    10     1     0     0   19.6169900     GPS         TIME OF FIRST OBS
  2026    10     1     0     8   17.9919020     GPS         TIME OF LAST OBS
                                                            END OF HEADER
```

### Observables per constellation

| Constellation | Letter | Band digit | Nominal frequency | Observation types   | Units                                         |
|:-------------:|:------:|:----------:|:-----------------:|:-------------------:|:----------------------------------------------|
| Globalstar    | `A`    | 9          | 2486.1 MHz        | `D9C S9C`           | Doppler (Hz), C/N0 (dB-Hz)                    |
| Iridium       | `D`    | 6          | 1626.270833 MHz   | `D6C S6C`           | Doppler (Hz), C/N0 (dB-Hz)                    |
| Orbcomm       | `B`    | 3          | 137.2 MHz centre  | `D3C S3C F3C`       | Doppler (Hz), C/N0 (dB-Hz), slot (integer)    |

- `D?C` is the measured Doppler shift in Hz, copied without any sign change from
  the source measurements. It follows the RINEX definition: positive when the
  satellite is approaching the receiver.
- `S?C` contains the signal-to-noise density ratio (C/N0) in dB-Hz as a regular
  `F14.3` value, **not** the 1-digit signal strength flag of RINEX.
- The LLI and signal strength flag characters that follow each value are blank.
- A missing measurement is written as 16 blanks (`F14.3` plus the 2 flag characters).
- There are no pseudorange nor carrier phase observables.

### Observation records

- The epoch record follows the format above. Times are written with 0.1 µs
  resolution (`F11.7`). Fields of the epoch may be blank-padded (`> 2026 10  1  0  0 19.6169900`)
  or zero-padded; readers must accept both. The epoch flag is `0` and the receiver
  clock offset is not written.
- The time system is declared in `TIME OF FIRST OBS` / `TIME OF LAST OBS`.
  `pygnss` writes GPS time by default (UTC plus the GPS-UTC leap seconds, 18 s since 2017)
  and optionally UTC.
- Epoch records are written in increasing time order. Measurements with
  exactly the same time (after rounding to 0.1 µs) share an epoch; no
  binning is done. Within an epoch, satellites are sorted by their identifier.
- Satellite identifiers are the constellation letter followed by the 5-digit
  zero-padded NORAD id (`A37741`, `D41921`, `B41182`).

Example of observation records (the third and fourth lines show an Orbcomm
satellite with `D3C`, `S3C` and `F3C`, and an Iridium satellite):

```text
> 2026 10  1  0  0 19.6169900  0  1
A37741    -19706.715          11.280
> 2026 10  1  0  1 37.2938220  0  1
B41182      2270.507           3.680         205.000
> 2026 10  1  0  0 48.5935590  0  1
D41921     34545.898           7.280
```

The `F14.3` field of `F3C` contains the integer slot (`205.000` above), so the Orbcomm
carrier frequency of that observation is 137.2 MHz + 205 · 2.5 kHz = 137.7125 MHz.

### Satellite names and NORAD ids

The measurements identify satellites by its NORAD ID number. Some examples of satellite names and their corresponding NORAD ids are shown below:

| Name              | NORAD id |
|:------------------|:--------:|
| GLOBALSTAR M078   | 39076    |
| GLOBALSTAR M086   | 38045    |
| GLOBALSTAR M091   | 37741    |
| IRIDIUM 103       | 41918    |
| IRIDIUM 105       | 41921    |
| IRIDIUM 109       | 41919    |
| IRIDIUM 166       | 43570    |
| ORBCOMM FM110     | 41182    |

Other names can be resolved with the object name in the CelesTrak catalogue
(`https://celestrak.org/NORAD/elements/gp.php?NAME=<name>&FORMAT=json`, field `NORAD_CAT_ID`).

## References

[De Bast et al., 2023] De Bast, Sibren, Jean-Marie Sleewaegen, and Wim De Wilde. "Analysis of Multipath Code-Range Errors in Future LEO-PNT Systems." Engineering Proceedings 54, no. 1 (2023): 34.

[Humphreys et al., 2023] Humphreys, Todd E., Peter A. Iannucci, Zacharias M. Komodromos, and Andrew M. Graff. "Signal structure of the starlink ku-band downlink." IEEE Transactions on Aerospace and Electronic Systems (2023).

[Prol et al., 2022] Prol, Fabricio S., R. Morales Ferre, Zainab Saleem, Petri Välisuo, Cristina Pinell, Elena-Simona Lohan, Mahmoud Elsanhoury et al. "Position, navigation, and timing (PNT) through low earth orbit (LEO) satellites: A survey on current status, challenges, and opportunities." IEEE Access (2022).
