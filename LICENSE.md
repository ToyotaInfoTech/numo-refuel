# OSM-derived files

Four groups of files in this repository are derived from the [OpenStreetMap](https://www.openstreetmap.org/): the SUMO network file `scenario/nagoya.net.xml`, the gas-station parking areas in `scenario/gas_stations.reachable.add.xml`, the route files under `scenario/routes/`, and the traffic-light programs in `scenario/nagoya_waut.add.xml`. Each of those files, taken as a whole, is a database made available under the Open Database License: http://opendatacommons.org/licenses/odbl/1.0/. The licence applying to the individual contents of those databases differs between the network and gas-station files and the rest, and is stated in the sections below.

Copyright (C) OpenStreetMap contributors; derived databases (C) 2024-2026 Toyota Motor Corporation.

These files contain information from OpenStreetMap, which is made available under the ODbL.

Three of the four come from the upstream NUMo scenario, in modified or unmodified form; what was changed is set out under *Files from the upstream NUMo scenario* below.

## The SUMO network file

`scenario/nagoya.net.xml` was originally imported from the OpenStreetMap. Any rights in the individual contents of that database are licensed under the Database Contents License: http://opendatacommons.org/licenses/dbcl/1.0/.

## The gas-station parking areas

`scenario/gas_stations.reachable.add.xml` is a database assembled by Toyota Motor Corporation from the OpenStreetMap. It records those OpenStreetMap fuel stations (`amenity=fuel`) of Nagoya that could be mapped onto a drivable lane of `scenario/nagoya.net.xml` and reached there, as SUMO `parkingArea` elements, so that both the station locations and the lane references and offsets are OpenStreetMap-derived. The selection criteria and the resulting station count are described under *Model summary* in [README.md](README.md). Any rights in the individual contents of that database are licensed under the Database Contents License: http://opendatacommons.org/licenses/dbcl/1.0/.

## Route files and traffic-light programs

The route files under `scenario/routes/` and the traffic-light programs in `scenario/nagoya_waut.add.xml` are databases assembled by Toyota Motor Corporation from third-party open data. They are derived from the OpenStreetMap through the network in `scenario/nagoya.net.xml`: they reference its edges and junctions by identifiers that are for the most part built from the OpenStreetMap way and node identifiers, and they reflect parts of its structure.

The individual contents of those databases, other than any OpenStreetMap data they carry, are licensed under the Creative Commons Attribution 4.0 International License: http://creativecommons.org/licenses/by/4.0/. Those contents are the ones derived from the Ministry of Land, Infrastructure, Transport and Tourism and the Japan Road Traffic Information Center open data described in the subsections below, together with Toyota Motor Corporation's own contribution to those files, which in the route files includes the refueling stops this repository inserts. Those stops reproduce no third-party data of their own: they are the output of a probability model, and the Japanese government statistics from which its parameters were derived are cited, with the credit and processing notice that their terms of use require, under *Statistical-data notice* in [README.md](README.md).

To the extent that these files carry OpenStreetMap data, that data is content of the OpenStreetMap database and remains subject to the DbCL 1.0 cited above.

### Source: road traffic census (`scenario/routes/`)

The traffic volumes that the route files reproduce are derived from the general traffic volume survey of the FY2021 nationwide road traffic census (令和3年度 全国道路・街路交通情勢調査 一般交通量調査, commonly 道路交通センサス), published by the Ministry of Land, Infrastructure, Transport and Tourism of Japan at https://www.mlit.go.jp/road/census/r3/. That survey is published under the [Public Data License 1.0](https://www.digital.go.jp/resources/open_data/public_data_license_v1.0) (公共データ利用規約（第1.0版）, PDL1.0), which requires the source to be credited and both the fact of any processing of the content and the party that performed it to be stated, and is declared compatible with CC BY 4.0.

The route files were produced by processing that survey, in the upstream NUMo scenario from which this repository inherits the trips. Anyone redistributing them must carry the following notice:

```
出典：「令和3年度 全国道路・街路交通情勢調査 一般交通量調査」（国土交通省）
（https://www.mlit.go.jp/road/census/r3/）を加工してトヨタ自動車株式会社が作成
```

In English: created by Toyota Motor Corporation by processing the *FY2021 Nationwide Road Traffic Census, General Traffic Volume Survey* (Ministry of Land, Infrastructure, Transport and Tourism), https://www.mlit.go.jp/road/census/r3/.

The route files are not published by, endorsed by, or attributable to the Ministry, and must not be presented as if the Ministry or any other organ of the Japanese state had produced them.

### Source: intersection control information (`scenario/nagoya_waut.add.xml`)

The signal cycle lengths and phase splits that the traffic-light programs reproduce are derived from the intersection control information (交差点制御情報) published as open data by the Japan Road Traffic Information Center (公益財団法人日本道路交通情報センター, JARTIC) at https://www.jartic.or.jp/service/opendata/, under the [terms of use](https://www.jartic.or.jp/d/opendata/riyou_kiyaku.pdf) published there. Those terms require the source to be credited and any processing of the data to be stated (Article 2(1)), and declare in Article 6(2) that data to which they apply may also be used under CC BY 4.0.

`scenario/nagoya_waut.add.xml` was produced by processing that data, again in the upstream NUMo scenario. Anyone redistributing it must carry the following notice:

```
出典：「交差点制御情報」（公益財団法人日本道路交通情報センター）
（https://www.jartic.or.jp/service/opendata/）を加工して作成
```

In English: created by processing the *Intersection Control Information* (Japan Road Traffic Information Center), https://www.jartic.or.jp/service/opendata/.

The traffic-light programs are not published by, endorsed by, or attributable to JARTIC, the Japanese state or any of its organs, and must not be presented as if any of them had produced the programs.

# Files from the upstream NUMo scenario

`scenario/nagoya.net.xml`, the route files under `scenario/routes/`, `scenario/nagoya_waut.add.xml` and `scenario/nagoya_with_refuel.sumocfg` originate from the NUMo (Nagoya Urban Mobility) scenario at https://github.com/ToyotaInfoTech/numo. The first three are OSM-derived files and are governed by the sections above; the configuration file is governed by *All other files* below.

The route files `scenario/routes/nagoya_*.refuel.rou.xml` are derived from the upstream `routes/nagoya_*.rou.xml`: vehicle identifiers, departure times and route edge sequences are unchanged, and an on-route `<stop>` at a gas-station `parkingArea` is inserted into a subset of the trips. `scenario/nagoya_with_refuel.sumocfg` is derived from the upstream `nagoya.sumocfg` with the route file list replaced and `gas_stations.reachable.add.xml` added to the additional files; the simulation settings are unchanged. `scenario/nagoya.net.xml` and `scenario/nagoya_waut.add.xml` are unchanged apart from their header comments.

# Container build files

Everything under `docker/` (e.g. `docker/Dockerfile`) is made available under the [MIT License](https://opensource.org/licenses/MIT).

```
MIT License

Copyright (c) 2026 Toyota Motor Corporation

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

Note that the image built from `docker/Dockerfile` installs the
[Eclipse SUMO](https://eclipse.dev/sumo/) runtime, which is distributed under the
[EPL 2.0](https://www.eclipse.org/legal/epl-2.0/). The MIT license above covers
only the build files in this repository, not SUMO itself or the other
dependencies the image pulls in.

# All other files

All other files are licensed under the Creative Commons Attribution 4.0 International License: http://creativecommons.org/licenses/by/4.0/. Copyright (C) 2024-2026 Toyota Motor Corporation. At present they are `README.md` and `scenario/nagoya_with_refuel.sumocfg`, together with any file added later that no section above covers.

This section does not apply to the OSM-derived files — `scenario/nagoya.net.xml`, `scenario/gas_stations.reachable.add.xml`, the route files under `scenario/routes/`, and `scenario/nagoya_waut.add.xml` — whose terms are set out under *OSM-derived files* above.

This section does not apply to `LICENSE.md` itself: it is the licence notice for the files above, not one of the works being licensed.
