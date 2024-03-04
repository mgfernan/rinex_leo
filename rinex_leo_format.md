# RINEX-LEO

This section includes the modifications proposed by Rokubun to accomodate
LEO data in the [RINEX 4.01 format](https://files.igs.org/pub/data/format/rinex_4.01.pdf).

One of the main objectives of having a RINEX file adapted for LEO is having a
consistency of satellite ID between Navigation and Observation files. This
makes it easy to analyze data (either real or simulated) and obtain various
performance metrics.

# Navigation file

For navigation files, this proposal takes advantage of the fact that the
navigation blocks are preceded by a header (starting with `>`), which allows
certain flexibility in the satellite ID length. Moreover, since the satellite ID
is included in the header, it does not then need to be added in the actual
navigation block (as done for MEO GNSS satellites). This helps to maintain the
lengths of subsequent fields of the `EPOCH / SV CLOCK` line.

The proposal for the extension is as follows (following the same format
description used in the RINEX format):

```
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

ORBIT - 3           - reserved                                  4X,4D19.12
                    - C_ic (radians)
                    - OMEGA0 (radians)
                    - C_is (radians)

ORBIT - 4           - i0 (radians)                              4X,4D19.12
                    - C_rc (meters)
                    - omega (radians)
                    - OMEGA DOT (radians/sec)

ORBIT - 5           - IDOT (radians/sec)                        4X,4D19.12
                    - Delta n0 dot (radians/sec^2)
                    - reserved
                    - reserved
```

An example of a navigation block for a Starlink LEO satellite 2434 is shown
below. This example has been built transforming the TLE elements of the Starlink
satellite found at [Celestrak](https://celestrak.org/NORAD/elements/gp.php?GROUP=starlink&FORMAT=tle) and assuming no orbit perturbations
(see [[Garcia-Fernandez, 2023]](https://arxiv.org/pdf/2401.17767)).

```
> EPH Z02434
    2023 11 02 11 10 24 0.000000000000e+00 0.000000000000e+00 0.000000000000e+00
     0.000000000000e+00 0.000000000000e+00 1.989675347274e-09 4.642884733583e+00
     0.000000000000e+00 1.811000000000e-04 0.000000000000e+00 2.835776281159e+03
     0.000000000000e+00 0.000000000000e+00 5.882354736496e+00 0.000000000000e+00
     9.288031413876e-01 0.000000000000e+00 1.642393223370e+00 0.000000000000e+00
     0.000000000000e+00 0.000000000000e+00 0.000000000000e+00 0.000000000000e+00
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

|`LEO constellation type`| Constellation | `LEO satellite id` |
|:----------------------:|:-------------:|:------------------:|
| `L` | Generic LEO constellation | custom |
| `X` | Xona                      | NORAD id |
| `Y` | Geely                     | NORAD id |
| `Z` | Starlink                  | NORAD id |
| `O` | Oneweb                    | NORAD id |
| `V` | Spire                     | NORAD id |

# Observable file

The limitation of 2-digit for the satellite ID in the Observation file is a
major blocking point unless a breaking change is introduced in the format.
A possible way forward would be to keep, in the measurement lines, the 2-digit
satellite id notation as specified in the RINEX standard and using the
constellation letter as usual (see table of LEO constellations above). For this
to happen, a dictionary of alias that links the `LEO satellite id` with the
temporary satellite designation (*alias*) will be required. This *alias* dictionary
applies only to the observable file being processed. For that purpose, a `COMMENT`-like
field with the label `SAT ID ASSIGNEMENT` could be used through two possible ways:

- In the RINEX **header**. Example

```
     4.99           OBSERVATION DATA    M                   RINEX VERSION / TYPE
ARGOS               ROKUBUN             20210702 000126 UTC PGM / RUN BY / DATE
.
.
.
Z95 = Z43012                                                SAT ID ASSIGNEMENT
.
.
.
                                                            END OF HEADER
```

- Using **EVENT FLAG**: Example:

```
> 2006 12 20 12  1  0.0000000  4 1
Z95 = Z43012                                                SAT ID ASSIGNEMENT
> 2022 12 20 12  1  4.6030000  0 30
G23  24718012.436   129894024.173    24718009.543   101216196.673
Z95    732793.036      732794.384
```

## Observable codes

Another important aspect of the RINEX observation types are the codes that
identify the different measurments. 



| GNSS system | Freq. band / frequency | Channel or code| Pseudo range | Carrier phase | Doppler | Signal strength |
|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| **LEO** | L1 / 1575.42 | C/A | C1C | L1C | D1C | S1C |
|         | L5 / 1176.45 | C/A | C5C | L5C | D5C | S5C |
|         | S[^1][^2] / 2492.028 | C/A | C9C | L9C | D9C | S9C |

Additionaly, other authors propose the following bands:

- C-band (e.g. 5020MHz, [De Bast et al., 2023])
- Ka-band (e.g. 11 GHz, [Humphreys et al., 2023]), for compatibility with communication satellites (), albeit this can be greatly affected by channel losses [Prol et al., 2022]

[^1]:[Prol et al., 2022]

[^2]:Parallelism to NavIC system, as per [Rinex 4.0 definition](https://files.igs.org/pub/data/format/rinex_4.01.pdf)

## References


[De Bast et al., 2023] De Bast, Sibren, Jean-Marie Sleewaegen, and Wim De Wilde. "Analysis of Multipath Code-Range Errors in Future LEO-PNT Systems." Engineering Proceedings 54, no. 1 (2023): 34.

[Humphreys et al., 2023] Humphreys, Todd E., Peter A. Iannucci, Zacharias M. Komodromos, and Andrew M. Graff. "Signal structure of the starlink ku-band downlink." IEEE Transactions on Aerospace and Electronic Systems (2023).

[Prol et al., 2022] Prol, Fabricio S., R. Morales Ferre, Zainab Saleem, Petri Välisuo, Cristina Pinell, Elena-Simona Lohan, Mahmoud Elsanhoury et al. "Position, navigation, and timing (PNT) through low earth orbit (LEO) satellites: A survey on current status, challenges, and opportunities." IEEE Access (2022).