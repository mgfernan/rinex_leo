# Static dataset LEO only

The data contained in this folder contains have been generated using the
[COTS hardware LEO-PNT simulator](https://www.rokubun.cat/cots-based-leo-pnt-hardware-simulator/)

- Broadcast file: `sat_01300.rnx`, which contains the ephemeris of 20 LEO satellites.
- The broadcast file uses conventional RINEX 2.11 and does not implement the
  RINEX-LEO header or satellite-ID proposal described in the repository format
  document.
- U-blox binary file: `leo_01300.ubx` is the u-blox binary packet with the observables collected in this simulated scenario using a u-blox ZED-F9P receiver
- Septentrio binary file: `leo_01300.sbf` similar than the previous one using a Septentrio Mosaic receiver
- Simulation epoch: 2022-01-01 00:00
- Simulated position:
  - xyz: 4789757, 181209, 4193819
  - lon/lat/height: 2.166615475874435, 41.375261174548093, 49.323
