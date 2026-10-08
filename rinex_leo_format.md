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
     4.01          OBSERVATION DATA    M                    RINEX VERSION / TYPE
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

## References

[De Bast et al., 2023] De Bast, Sibren, Jean-Marie Sleewaegen, and Wim De Wilde. "Analysis of Multipath Code-Range Errors in Future LEO-PNT Systems." Engineering Proceedings 54, no. 1 (2023): 34.

[Humphreys et al., 2023] Humphreys, Todd E., Peter A. Iannucci, Zacharias M. Komodromos, and Andrew M. Graff. "Signal structure of the starlink ku-band downlink." IEEE Transactions on Aerospace and Electronic Systems (2023).

[Prol et al., 2022] Prol, Fabricio S., R. Morales Ferre, Zainab Saleem, Petri Välisuo, Cristina Pinell, Elena-Simona Lohan, Mahmoud Elsanhoury et al. "Position, navigation, and timing (PNT) through low earth orbit (LEO) satellites: A survey on current status, challenges, and opportunities." IEEE Access (2022).
