# NUMo-Refuel: Nagoya Urban Mobility (NUMo) with Refueling Stops

**NUMo-Refuel** extends the baseline
**[NUMo (Nagoya Urban Mobility) scenario](https://github.com/ToyotaInfoTech/numo)**
(the city-scale SUMO traffic scenario of Nagoya, Japan) by probabilistically
adding **refuel stops at gas stations** along each vehicle's existing route.
Each trip is assigned a **vehicle category** and the refuel probability is
computed **per category** from public statistics (see *Derivation of refueling
parameters* below). The original route edge sequence is never altered and no detour is
introduced: only an on-route `<stop>` at a `parkingArea` is inserted
to preserve the calibrated road traffic volume. 

This repository is a **self-contained companion** to the upstream NUMo
scenario. Everything needed to run the 24-hour production scenario standalone
is included here. 

## Associated publication

This repository contains independently maintained software documentation,
scenario files, and reproducibility materials associated with the following
accepted paper:

```
Takamasa Higuchi, “Poster: A City-Scale Mobility Simulation with Statistically Calibrated Refueling Stops,” accepted for EAI MobiQuitous 2026, 2026. To appear.
```

Final bibliographic details will be added once the paper is published.

## Contents

Everything the simulation needs at run time lives under `scenario/`.

```
.
├── scenario/                             # Everything SUMO reads at run time
│   ├── nagoya_with_refuel.sumocfg        # SUMO simulation config
│   ├── nagoya.net.xml                    # Road network (network data unchanged from original NUMo)
│   ├── nagoya_waut.add.xml               # Traffic-light WAUT program (unchanged from NUMo)
│   ├── gas_stations.reachable.add.xml    # Gas station parkingAreas (after reachability filter)
│   └── routes/                           # Route files for 30-min bins with refuel <stop> inserted
│       └── nagoya_HH_MM.refuel.rou.xml
├── docker/
│   └── Dockerfile                        # Minimal headless SUMO 1.19.0 runtime (see below)
├── LICENSE.md                            # Per-component licensing of this bundle
└── README.md                             # This file
```

## How to run

### Requirements

- **SUMO 1.19.0**

> **Note:** This scenario has been tested with **SUMO 1.19.0**. Running it with a
> different SUMO version may produce different results.

### Run (24 hours)

```bash
# from the repository root
sumo -c scenario/nagoya_with_refuel.sumocfg           # headless, end=86400 (24h)
# or, to watch it:
sumo-gui -c scenario/nagoya_with_refuel.sumocfg
```

### Recording outputs

The config produces no output files by default. Add whatever you need on the command
line, e.g.:

```bash
sumo -c scenario/nagoya_with_refuel.sumocfg \
     --tripinfo-output out/tripinfo.xml \
     --stop-output     out/stops.xml \
     --summary-output  out/summary.xml
```

(Create the target directory first; `--stop-output` records each gas station refuel stop.)

### Run in a Docker container

If you don't want to install SUMO on the host, a minimal
[`docker/Dockerfile`](docker/Dockerfile) is included. It builds a small image containing
**only the headless SUMO 1.19.0 runtime** . The scenario data is
**not** baked in but mounted at run time, so the image stays small and the exact SUMO
version is guaranteed regardless of what is on the host. 

```bash
docker build -t numo-refuel docker

# run the 24-hour scenario, mounting scenario/ at /scenario (read-only)
# and a writable host directory at /out for the outputs
mkdir -p out
docker run --rm \
    --user "$(id -u):$(id -g)" \
    -v "$PWD/scenario":/scenario:ro \
    -v "$PWD/out":/out \
    numo-refuel \
    --tripinfo-output /out/tripinfo.xml \
    --stop-output     /out/stops.xml \
    --summary-output  /out/summary.xml
```

Notes:

- The container's default command is `sumo -c nagoya_with_refuel.sumocfg`; anything you
  pass after the image name is appended as extra SUMO options (e.g. `--end 3600` for a
  short run, or the `--*-output` flags above).
- `--user "$(id -u):$(id -g)"` makes the outputs owned by you instead of `root`. Omit it if
  you don't mind root-owned files.
- This is a **headless** runtime. `sumo-gui` (which needs an X display) is not the
  intended use of this image; run the GUI on the host instead.

## Model summary

- The baseline NUMo road network, vehicle identifiers, departure times, and
  route edge sequences are retained. Refueling stops are added to derived
  copies of the route files without adding, removing, or reordering route edges. 
  Refueling is realized by exactly
  two additions:
  1. A set of `parkingArea` elements representing gas stations
     (`scenario/gas_stations.reachable.add.xml`).
  2. Probabilistically inserted **on-route** `<stop parkingArea=... parking="true"/>`
     entries in each 30-minute route file (no detour: a stop is only ever placed at a
     `parkingArea` that already lies on the trip's original route).
- Gas station locations are derived from **240 OSM fuel stations** across Nagoya. After
  snapping to the network (≤ 50 m to a drivable edge) and a duarouter
  reachability filter, 174 physical gas stations remain. 
- Each gas station is represented by up to two independent parkingAreas, `gs_XXXXX_fwd` and
  `gs_XXXXX_rev`, one per travel direction, because a SUMO `parkingArea` binds to a
  single lane and cannot serve both directions of a road at once.
- Each `parkingArea` is 20 m long and has `roadsideCapacity="999"`, so that capacity
  never limits refueling. Both are model values, not surveyed forecourt dimensions.

## Derivation of refueling parameters

The released route files were generated using the parameters and procedure
specified below. Vehicle categories are used only during scenario generation.

### Refueling probability

Each trip is assigned one of 11 vehicle categories according to the
trip shares $w_c$. For category $c$, the target per-trip refueling probability
is

$$
p_{\mathrm{trip},c}
  = \frac{d_c e_c}{C_c \eta_c m_c},
$$

where:

| Symbol | Definition | Unit |
|---|---|---|
| $d_c$ | distance traveled per registered vehicle per day | km/vehicle/day |
| $e_c$ | fuel consumed per kilometer | L/km |
| $C_c$ | representative tank capacity | L |
| $\eta_c$ | effective fill ratio | dimensionless |
| $m_c$ | trips per registered vehicle per day | trips/vehicle/day |

The effective fill ratio is fixed at $\eta_c = 0.75$ for all categories.

| Category | $w_c$ | $d_c$ | $e_c$ | $m_c$ | $C_c$ | $p_{\mathrm{trip},c}$ |
|---|--:|--:|--:|--:|--:|--:|
| Commercial freight, ordinary/small/special | 0.031 | 27.16 | 0.153 | 4.5 | 95 | 0.01296 |
| Commercial freight, kei | 0.010 | 45.97 | 0.090 | 4.5 | 30 | 0.04086 |
| Commercial passenger, bus/taxi | 0.046 | 82.16 | 0.097 | 20.0 | 95 | 0.00559 |
| Private freight, ordinary | 0.021 | 17.96 | 0.158 | 2.0 | 75 | 0.02522 |
| Private freight, small | 0.046 | 32.70 | 0.098 | 2.0 | 70 | 0.03052 |
| Private freight, kei | 0.110 | 18.58 | 0.076 | 2.0 | 30 | 0.03138 |
| Private passenger, bus/special | 0.018 | 21.63 | 0.150 | 1.7 | 95 | 0.02679 |
| Private passenger, hybrid | 0.160 | 27.48 | 0.064 | 1.7 | 40 | 0.03448 |
| Private passenger, ordinary | 0.161 | 21.49 | 0.104 | 1.7 | 50 | 0.03506 |
| Private passenger, small | 0.124 | 20.09 | 0.083 | 1.7 | 40 | 0.03270 |
| Private passenger, kei | 0.273 | 18.75 | 0.066 | 1.7 | 30 | 0.03235 |

The trip shares are calculated as

$$
w_c = \frac{N_c m_c}{\sum_k N_k m_k},
$$

where $N_c$ is the estimated registered-vehicle population of category $c$.

Using the unrounded build parameters, the weighted scenario-wide target is

$$
p_{\mathrm{trip}}
  = \sum_c w_c p_{\mathrm{trip},c}
  \approx 0.0310.
$$

The values displayed in the table are rounded. Recalculation from the displayed
values may therefore differ slightly from calculations using the unrounded
build parameters.

### Adjustment for on-route eligibility

NUMo-Refuel does not introduce detours. A trip is eligible for a refueling stop
only if its original route contains a reachable gas-station `parkingArea`.

Let $q_c$ be the fraction of trips in category $c$ that satisfy this condition.
For an eligible trip, the scenario builder uses

$$
p_{\mathrm{adj},c}
  = \min\left(1,\frac{p_{\mathrm{trip},c}}{q_c}\right).
$$

For the released scenario, $q_c$ is approximately 0.73 for every category.
The maximum value of $p_{\mathrm{adj},c}$ is approximately 0.056, so the upper
clamp is not active.

### Parameter provenance

The parameters have the following provenance:

- **$d_c$ and $e_c$:** Derived from the annual gasoline tables of the MLIT
  *Survey on Motor Vehicle Fuel Consumption*.
- **$m_c$:** Derived from the FY2021 *Automobile Origin–Destination Survey* of the
  MLIT *Road Traffic Census*.
  Source vehicle subclasses were aggregated into four operational classes
  using estimated vehicle populations as weights. The resulting values are
  1.7 for private passenger, 2.0 for private freight, 4.5 for commercial
  freight, and 20.0 for commercial passenger vehicles.
- **$N_c$ and $w_c$:** Vehicle populations are based on the MLIT
  *The Number of Motor Vehicles Owned*. Trip shares are calculated from
  $N_c m_c$ and normalized over the 11 categories.
- **Hybrid share:** The hybrid passenger category is separated from the
  ordinary and small private-passenger population using the hybrid share of
  registered passenger cars. 
- **$C_c$:** Fixed model values of 30, 40, 50, 70, 75, and 95 L, based on
  published specifications for representative Japanese vehicles and rounded
  by vehicle-size class.
- **$\eta_c$:** Model assumption fixed at 0.75. It is not derived from the
  cited public statistics.

### Statistical-data notice

The statistical sources are used as inputs to NUMo-Refuel calculations.
The project performs category aggregation, unit conversion, calculation of
trip rates and trip shares, and derivation of refueling probabilities.

The resulting parameters are derived model values, not official statistics.
The Government of Japan does not endorse or guarantee this derived scenario.

All three statistics below are cited as published through e-Stat, the portal site of
official statistics of Japan, and are used under the e-Stat Terms of Use.

Sources:

1. Ministry of Land, Infrastructure, Transport and Tourism,
   [Survey on Motor Vehicle Fuel Consumption](https://www.e-stat.go.jp/en/statistics/00600370),
   Government Statistics Code 00600370.
2. Ministry of Land, Infrastructure, Transport and Tourism,
   [Road Traffic Census](https://www.e-stat.go.jp/en/statistics/00600580)
   (全国道路・街路交通情勢調査), Automobile Origin–Destination Survey
   (自動車起終点調査), FY2021, Government Statistics Code 00600580.
3. Ministry of Land, Infrastructure, Transport and Tourism,
   [The Number of Motor Vehicles Owned](https://www.e-stat.go.jp/en/statistics/00600700),
   Government Statistics Code 00600700.
4. [e-Stat Terms of Use](https://www.e-stat.go.jp/terms-of-use).

Anyone reusing the derived refueling parameters must carry the following notice,
which the e-Stat Terms of Use require:

```
出典：「自動車燃料消費量調査」（国土交通省）、
「全国道路・街路交通情勢調査 自動車起終点調査」（国土交通省）、
「自動車保有車両数」（国土交通省）を加工して作成
```

In English: created by processing the *Survey on Motor Vehicle Fuel Consumption*,
the *Automobile Origin–Destination Survey* of the *Road Traffic Census*, and
*The Number of Motor Vehicles Owned* (all Ministry of Land, Infrastructure,
Transport and Tourism).

The derived parameters are not published by, endorsed by, or attributable to the
Ministry, and must not be presented as if the Ministry or any other organ of the
Japanese state had produced them.

### Dwell-time model

Public statistics used by this project do not provide per-visit gas-station
dwell times. NUMo-Refuel therefore uses the following synthetic model:

$$
X = \exp(\mu + \sigma U),
\qquad
D = \max(X, 1),
\qquad
U \sim \mathcal{N}(0,1),
$$

where $D$ is in minutes, $\mu = \ln 5$, and $\sigma = 0.4$.

This gives a log-normal distribution with a median of 5 min and a model lower
bound of 1 min. The distribution parameters and lower bound are model
assumptions, not estimates from the cited public statistics.

## License

The SUMO network file `scenario/nagoya.net.xml`, the gas-station parking areas in
`scenario/gas_stations.reachable.add.xml`, the route files under `scenario/routes/`
and the traffic-light programs in `scenario/nagoya_waut.add.xml` are databases
derived from OpenStreetMap and from Japanese open data. Each of those files, taken
as a whole, is a database made available under the
[ODbL 1.0](http://opendatacommons.org/licenses/odbl/1.0/), and different terms apply
to their individual contents. The container build files under `docker/` are licensed
under the [MIT License](https://opensource.org/licenses/MIT), and all other files
under [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/).

See [LICENSE.md](LICENSE.md) for the authoritative terms, including the attribution
notices that redistribution has to carry. 

### OpenStreetMap attribution
 
Those four groups of files contain information from OpenStreetMap, which is made
available under the ODbL.

```
© OpenStreetMap contributors
```