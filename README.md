# Panipat pit detection — sand, mining and brick-kiln pits from free satellite data

Finds excavation pits — including small ones — in Panipat district, Haryana (India) using only free data:
Sentinel-2, Sentinel-1, Dynamic World, building maps and AlphaEarth AI embeddings, processed in
Google Earth Engine and at full 10 m detail in Python.

![All detected pits](results/panipat_2026-06-30_to_2026-09-30/jpg/00_pit_map_full_resolution.jpg)

## Results — Panipat district, 30 June to 30 September 2026

| | Pits | Area |
|---|---:|---:|
| Outside the Yamuna riverbed | 237 | 53 ha |
| In the Yamuna riverbed | 86 | 50 ha |
| **Total in Panipat** | **323** | **103 ha** |
| New since the same months of 2025 | 64 | |
| Of these, in registered mining pits (HSAC "Mining Pit", kept as pits) | 2 | 0 ha |
| Removed: registered ponds (HSAC pond register) | 28 | 5 ha |
| Removed: likely village ponds (water next to a village) | 5 | 1 ha |
| Removed: in canals and drains (1 Tributary Drain No.2, 1 Gangesar Disty, 1 Main Drain No. 2, 1 Parallel Delhi Branch) | 4 | 0 ha |
| Removed: on roads and highways (1 unnamed road) | 1 | 0 ha |
| Removed: the Yamuna's own channel (riverbed water bodies of 5 ha or more) | 3 | 126 ha |
| Just across the district border (Sonipat 8, Samli 6, Jind 2, Baghpat 2, Karnal 1) | 19 | 6 ha |

By size: 120 tiny (< 0.1 ha), 149 small (0.1–0.5 ha), 45 medium (0.5–2 ha), 9 large (> 2 ha).
1,312 km² analysed with 44 Sentinel-2 and 24 Sentinel-1 scenes. Removed detections are listed in
[`excluded.csv`](results/panipat_2026-06-30_to_2026-09-30/excluded.csv) with the reason and the pond, canal or road, and are not counted above.
The analysis area is today's district boundary plus 250 m, so the 19 pits found just across the border
are listed with their own district and not counted for Panipat.

### Pits by tehsil

| Tehsil | Pits | Area (ha) | New pits |
|---|---:|---:|---:|
| Matlauda | 102 | 25.9 | 21 |
| Bapoli | 87 | 44.8 | 16 |
| Israna | 64 | 11.2 | 9 |
| Samalkha | 45 | 15.5 | 8 |
| Panipat | 25 | 5.5 | 10 |
| **Total** | **323** | **102.9** | **64** |

Villages with the most pits are in [`pits_by_village.csv`](results/panipat_2026-06-30_to_2026-09-30/pits_by_village.csv).

**Registered ponds, canals and roads are removed.** Every detection lying in a water body of the Haryana
pond register (HSAC Pond MIS, see below), in a canal or drain (Irrigation Department network from HSAC)
or on a road or highway (OpenStreetMap, including roads under construction) is taken out of the pit list
and written to `excluded.csv` with the reason and the pond, canal or road. So are water bodies of 5 ha or
more in the Yamuna bed: they are the river's own channel, which moved onto last year's sand bars and
fields in the 2026 monsoon (the two largest, 74 ha at Raksehra and 44 ha at Bilaspur, are 2 km long strips
of river water that were almost never water in 1984–2021, JRC Global Surface Water). The model is trained with
registered ponds as "not pit" examples, and road and canal pixels are never used as pit examples.

**Registered mining pits stay pits.** Around Panipat the register's 35 entries typed "Mining Pit" are
excavations: clay pits "Near To Breaklin" (brick kiln) at Dikadla, a "Minnig Pit" at Kiwana, a "Pit" at
Buana Lakhu, dry pits marked "No Pond Here" at Karhansh and Manana, and fish ponds dug in old pits at
Kakoda. Detections in them stay in the pit list with `hsac_type` = "Mining Pit" and the register's name
(`POND_KEEP_TYPES` in the notebook, `pondKeepTypes` in the Earth Engine script).

**Treat these as candidates to verify, not confirmed pits.** There is no ground truth yet, so the pit
threshold is the default 0.6, and pits whose average probability (`prob_mean`) is below 0.65 are
dropped (`MIN_PIT_PROB` in the notebook, `minPitProb` in the Earth Engine script). The most confident
detections are small bright, bare patches among fields in Matlauda tehsil (Bhadaur, Bhandari, Alupur) —
pits 3 and 6 lie next to what looks like a brick kiln (the reddish patch) and are plausible clay pits, but
bright bare patches such as pits 1 and 2 can also be harvested or fallow fields. In the Yamuna bed the
remaining detections are up to 3.6 ha and may include pieces of river water and sand bars as well as
sand mining.
Labelling a few dozen examples (below) is the way to fix these.

| Gallery (10 m view · water · radar · probability) | Statistics |
|---|---|
| ![Gallery](results/panipat_2026-06-30_to_2026-09-30/jpg/05_gallery_1_most_confident.jpg) | ![Statistics](results/panipat_2026-06-30_to_2026-09-30/jpg/07_statistics.jpg) ![Pit history](results/panipat_2026-06-30_to_2026-09-30/jpg/06_pit_history_example.jpg) |

Result files in [`results/panipat_2026-06-30_to_2026-09-30/`](results/panipat_2026-06-30_to_2026-09-30/):

| File | What it is |
|---|---|
| `pits.csv` | Every pit: **latitude, longitude, district, tehsil, village**, area, probability, new or older, riverbed or not, nearest road and canal and the distances to them (`road_distance_m`, `canal_distance_m`), the register's type and name when the pit is a registered mining pit (`hsac_type`, `hsac_name`), Google Maps link |
| `pits_new.csv`, `pits_old.csv` | The same list split in two: **new pits** (cropland in the same months of 2025, bare or flooded now) and **older pits**, with the same columns |
| `pits_new.kml`, `pits_old.kml` | Google Earth files of the new pits (red) and the older pits (magenta) only, the same pins and outlines as `pits.kml` |
| `excluded.csv`, `excluded.geojson` | Detections removed, with the reason in `category`: registered ponds (the register's `hsac_id`, name and type, and the share of the detection inside it), likely village ponds (water next to a village), detections in canals / drains and on roads / highways (the canal or road and the share of the detection on its corridor) |
| `pits_by_tehsil.csv`, `pits_by_village.csv` | Number of pits, hectares and new pits per tehsil and per village |
| `pits.geojson`, `pits.gpkg` | Pit outlines for QGIS / ArcGIS, with latitude / longitude attributes |
| `pits.kml` | Google Earth: outline + numbered pin per pit, coordinates and a Google Maps link in each pin (red = new, magenta = older) |
| `pit_probability.tif` | Pit probability for every 10 m pixel (0–100) |
| `jpg/` | Numbered pit map with tehsil and village boundaries, registered ponds (light blue), canals (blue), main roads (orange) and a latitude / longitude grid, coordinate tables of the new and largest pits (08, 09), pits per tehsil (10), inputs, training examples, validation, probability, 4 gallery pages (coordinates in every title), pit history, statistics |
| `panipat_pit_detection_executed.ipynb` | The notebook with all outputs from this run |

Coordinates are WGS84 decimal degrees (6 decimals ≈ 0.1 m) of each pit's centre — or of a point
inside the pit when its shape is curved — so every pin falls on its pit. The numbers on the map are
the `pit_id` column of `pits.csv`. Each pit's tehsil and village are the boundaries its location falls
in (or the nearest one); pits just across the district border get their own district's name instead.

**Analysis area and boundaries** ([`data/`](data/)): Panipat's boundary today (2021, 1,262 km²) plus 250 m,
its 5 tehsils — Panipat, Israna, Samalkha, Bapoli and Matlauda (2018) — 189 villages and towns (2011
Census) and the neighbouring districts (Sonipat, Karnal, Jind, Shamli, Baghpat) from
[geoBoundaries](https://www.geoboundaries.org), made by [`tools/get_boundaries.py`](tools/get_boundaries.py):
GeoJSON for the notebook and `panipat_boundaries_shp.zip` (shapefiles of the tehsils, villages and the
district outline) for Earth Engine uploads. Earth Engine's own district layer (FAO GAUL 2015) misses
91 km² (7 %) of today's Panipat — strips along the Yamuna on the east edge (the largest 18 km² near
29.25° N, 77.15° E) and 29 km² in the south — so the notebook uses today's boundary instead.

**Registered ponds** ([`data/panipat_hsac_ponds.geojson`](data/panipat_hsac_ponds.geojson)): the pond inventory of
the Haryana Pond & Waste Water Management Authority, mapped by the Haryana Space Applications Centre
([HSAC Pond MIS](https://hsac.org.in/pond_mis/)) — 2,458 water bodies in and within ~5 km of Panipat,
1,275 of them in the district (2,104 ponds, 258 tanks, 58 water-logged areas, 3 lakes and 35 entries
typed "Mining Pit", which are kept as pits — see above), read from HSAC's public map service by
[`tools/get_hsac_ponds.py`](tools/get_hsac_ponds.py), which works for any Haryana district (HSAC's
server fails about half its requests at random, so the tool retries). `panipat_hsac_ponds_shp.zip` is the
same layer as a shapefile for an Earth Engine upload.

**Roads and highways** ([`data/panipat_roads.geojson`](data/panipat_roads.geojson)): 20,500 road segments
(about 6,450 km) in and within 5 km of Panipat from [OpenStreetMap](https://www.openstreetmap.org), read
by [`tools/get_osm_roads.py`](tools/get_osm_roads.py) — national highways NH-44 (GT Road), NH-709AD
(Panipat–Shamli road) and NH-709, state highways SH-11 and SH-14, district and village roads, and 6 km of
roads under construction. Each road is a corridor of its class's half-width (motorway 30 m, trunk 25 m,
primary 15 m, secondary 12 m, tertiary 10 m, village roads 6–7 m, under construction 25 m), widened on
either side to the bare strip seen on Sentinel-2 where a road is being built or widened (up to 40 m,
only where crops start at the strip's edge). Farm tracks are not used: they run along field edges and
into pits. `panipat_roads_shp.zip` is the shapefile for an Earth Engine upload.

**Canals and drains** ([`data/panipat_canals.geojson`](data/panipat_canals.geojson)): the Haryana Irrigation &
Water Resources Department's canal network (WJC Main Branch, Hansi Branch, Parallel Delhi Branch, the
Carrier Lined Channel, Bhalaut Sub Branch, Israna and Gohana distributaries, minors — about 1,690 km in
and around Panipat) and drains (about 760 km), plus the canal footprint polygons of the state canal layer, all from HSAC's public map services,
read by [`tools/get_hsac_canals.py`](tools/get_hsac_canals.py) for any Haryana district. Canal lines are
corridors of a half-width set by category and by discharge at the head (5 m for small minors up to 35 m
for canals of 2,000+ cusecs), widened like roads where Sentinel-2 shows a wider water / bank strip;
footprints get a 10 m margin. The Yamuna, which the source lists among the canals, is left out — its bed
is where sand is mined. `panipat_canals.csv` (WKT geometry) is the same layer for an Earth Engine upload.

## Pit coordinates

Every pit's latitude, longitude, tehsil and village are in [`pits.csv`](results/panipat_2026-06-30_to_2026-09-30/pits.csv) (also in the
GeoJSON, GeoPackage and KML), split into [`pits_new.csv`](results/panipat_2026-06-30_to_2026-09-30/pits_new.csv) (69 new pits)
and [`pits_old.csv`](results/panipat_2026-06-30_to_2026-09-30/pits_old.csv) (273 older pits); these include the 19 pits just across
the border, which have their own district in the `district` column. The tables below list the new pits and the largest pits; **Map** opens the
spot in Google Maps.

| New pits — coordinates | Largest pits — coordinates |
|---|---|
| ![New pits](results/panipat_2026-06-30_to_2026-09-30/jpg/08_new_pits_coordinates.jpg) | ![Largest pits](results/panipat_2026-06-30_to_2026-09-30/jpg/09_largest_pits_coordinates.jpg) |

<details>
<summary><b>69 new pits</b> (cropland in the same months of 2025, bare or flooded now) — largest first</summary>

| Pit | Latitude | Longitude | Village | Tehsil | Area (ha) | Setting | Map |
|---:|---:|---:|---|---|---:|---|---|
| 102 | 29.283528 | 77.080216 | Begumpur Tahrpur (63) | Bapoli | 2.37 | outside riverbed | [open](https://www.google.com/maps?q=29.283528,77.080216) |
| 100 | 29.284073 | 77.128672 | Goela Khurd (61) | Bapoli | 1.95 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.284073,77.128672) |
| 17 | 29.322457 | 76.877395 | Bhadaur (39) | Matlauda | 1.71 | outside riverbed | [open](https://www.google.com/maps?q=29.322457,76.877395) |
| 136 | 29.419396 | 76.979033 | Panipat Taraf Ansar (CT) | Panipat | 1.65 | outside riverbed | [open](https://www.google.com/maps?q=29.419396,76.979033) |
| 145 | 29.445890 | 76.854191 | Bal Jattan (11) | Matlauda | 1.18 | outside riverbed | [open](https://www.google.com/maps?q=29.445890,76.854191) |
| 34 | 29.345308 | 77.052280 | Rasulpur (43) | Bapoli | 1.09 | outside riverbed | [open](https://www.google.com/maps?q=29.345308,77.052280) |
| 64 | 29.274789 | 76.793744 | Khalila Majra Nain (59) | Israna | 1.04 | outside riverbed | [open](https://www.google.com/maps?q=29.274789,76.793744) |
| 46 | 29.336098 | 76.719165 | Urlana Kalan (51) | Matlauda | 0.97 | outside riverbed | [open](https://www.google.com/maps?q=29.336098,76.719165) |
| 27 | 29.362323 | 77.063221 | Chhajpur Kalan (42) | Bapoli | 0.71 | outside riverbed | [open](https://www.google.com/maps?q=29.362323,77.063221) |
| 69 | 29.256982 | 77.059138 | Jurasi Khalsa (74) | Samalkha | 0.62 | outside riverbed | [open](https://www.google.com/maps?q=29.256982,77.059138) |
| 26 | 29.330700 | 76.883747 | Bhadaur (39) | Matlauda | 0.49 | outside riverbed | [open](https://www.google.com/maps?q=29.330700,76.883747) |
| 147 | 29.475275 | 76.895938 | Baholi (12) | Panipat | 0.45 | outside riverbed | [open](https://www.google.com/maps?q=29.475275,76.895938) |
| 108 | 29.368822 | 77.115170 | Ladhopur | Bapoli | 0.41 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.368822,77.115170) |
| 142 | 29.416075 | 76.848157 | Khandra (10) | Matlauda | 0.40 | outside riverbed | [open](https://www.google.com/maps?q=29.416075,76.848157) |
| 169 | 29.476204 | 76.895834 | Baholi (12) | Panipat | 0.37 | outside riverbed | [open](https://www.google.com/maps?q=29.476204,76.895834) |
| 11 | 29.327152 | 76.780316 | Alupur (60) | Matlauda | 0.32 | outside riverbed | [open](https://www.google.com/maps?q=29.327152,76.780316) |
| 126 | 29.426246 | 76.742328 | Joshi (5) | Matlauda | 0.29 | outside riverbed | [open](https://www.google.com/maps?q=29.426246,76.742328) |
| 68 | 29.260544 | 76.803986 | Pardhana (65) | Israna | 0.27 | outside riverbed | [open](https://www.google.com/maps?q=29.260544,76.803986) |
| 106 | 29.361758 | 77.067947 | Ataullahpur (39) | Bapoli | 0.26 | outside riverbed | [open](https://www.google.com/maps?q=29.361758,77.067947) |
| 116 | 29.179708 | 77.129030 | Raksehra (91) | Samalkha | 0.23 | outside riverbed | [open](https://www.google.com/maps?q=29.179708,77.129030) |
| 160 | 29.385482 | 77.044715 | Chhajpur Khurd (41) | Bapoli | 0.23 | outside riverbed | [open](https://www.google.com/maps?q=29.385482,77.044715) |
| 81 | 29.295802 | 76.636038 |  | (Jind district) | 0.20 | outside riverbed | [open](https://www.google.com/maps?q=29.295802,76.636038) |
| 248 | 29.294791 | 76.636850 |  | (Jind district) | 0.20 | outside riverbed | [open](https://www.google.com/maps?q=29.294791,76.636850) |
| 103 | 29.282751 | 77.122937 | Sanjoli (62) | Bapoli | 0.20 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.282751,77.122937) |
| 99 | 29.289791 | 76.814050 | Karad(64) | Israna | 0.19 | outside riverbed | [open](https://www.google.com/maps?q=29.289791,76.814050) |
| 141 | 29.442120 | 76.987985 | Azizullapur (5) | Panipat | 0.18 | outside riverbed | [open](https://www.google.com/maps?q=29.442120,76.987985) |
| 25 | 29.329817 | 76.884442 | Bhadaur (39) | Matlauda | 0.18 | outside riverbed | [open](https://www.google.com/maps?q=29.329817,76.884442) |
| 150 | 29.458377 | 76.978450 | Ali Asgharpur (32) | Panipat | 0.18 | outside riverbed | [open](https://www.google.com/maps?q=29.458377,76.978450) |
| 303 | 29.260642 | 77.142541 |  | (Baghpat district) | 0.17 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.260642,77.142541) |
| 153 | 29.387329 | 77.070250 | Kurar (22) | Bapoli | 0.16 | outside riverbed | [open](https://www.google.com/maps?q=29.387329,77.070250) |
| 57 | 29.280991 | 76.827770 | Karad(64) | Israna | 0.16 | outside riverbed | [open](https://www.google.com/maps?q=29.280991,76.827770) |
| 94 | 29.283650 | 76.791748 | Khalila Majra Nain (59) | Israna | 0.16 | outside riverbed | [open](https://www.google.com/maps?q=29.283650,76.791748) |
| 203 | 29.406364 | 76.820248 | Madlauda (7) | Matlauda | 0.15 | outside riverbed | [open](https://www.google.com/maps?q=29.406364,76.820248) |
| 110 | 29.404253 | 77.104021 | Janbha (34) | Bapoli | 0.15 | outside riverbed | [open](https://www.google.com/maps?q=29.404253,77.104021) |
| 161 | 29.222541 | 76.952907 | Namonda (80) | Samalkha | 0.14 | outside riverbed | [open](https://www.google.com/maps?q=29.222541,76.952907) |
| 109 | 29.443310 | 76.982928 | Azizullapur (5) | Panipat | 0.14 | outside riverbed | [open](https://www.google.com/maps?q=29.443310,76.982928) |
| 162 | 29.455030 | 76.839987 | Dharamgarh (1) | Matlauda | 0.12 | outside riverbed | [open](https://www.google.com/maps?q=29.455030,76.839987) |
| 13 | 29.239652 | 77.089958 | Atta (80) | Samalkha | 0.12 | outside riverbed | [open](https://www.google.com/maps?q=29.239652,77.089958) |
| 152 | 29.387477 | 77.056092 | Chhajpur Khurd (41) | Bapoli | 0.12 | outside riverbed | [open](https://www.google.com/maps?q=29.387477,77.056092) |
| 29 | 29.360672 | 77.107842 | Nanhera (57) | Bapoli | 0.10 | outside riverbed | [open](https://www.google.com/maps?q=29.360672,77.107842) |
| 124 | 29.191389 | 77.141952 |  | (Baghpat district) | 0.09 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.191389,77.141952) |
| 111 | 29.187114 | 77.063789 | Dehra (85) | Samalkha | 0.09 | outside riverbed | [open](https://www.google.com/maps?q=29.187114,77.063789) |
| 165 | 29.387912 | 77.071202 | Kurar (22) | Bapoli | 0.08 | outside riverbed | [open](https://www.google.com/maps?q=29.387912,77.071202) |
| 341 | 29.258571 | 76.899998 | Balana (71) | Israna | 0.08 | outside riverbed | [open](https://www.google.com/maps?q=29.258571,76.899998) |
| 182 | 29.177969 | 77.129075 | Raksehra (91) | Samalkha | 0.07 | outside riverbed | [open](https://www.google.com/maps?q=29.177969,77.129075) |
| 140 | 29.425649 | 76.742123 | Joshi (5) | Matlauda | 0.07 | outside riverbed | [open](https://www.google.com/maps?q=29.425649,76.742123) |
| 53 | 29.315691 | 76.846842 | Kalkha (41) | Matlauda | 0.07 | outside riverbed | [open](https://www.google.com/maps?q=29.315691,76.846842) |
| 163 | 29.376663 | 76.871489 | Sutana (23) | Matlauda | 0.06 | outside riverbed | [open](https://www.google.com/maps?q=29.376663,76.871489) |
| 329 | 29.442977 | 76.847037 | Bal Jattan (11) | Matlauda | 0.06 | outside riverbed | [open](https://www.google.com/maps?q=29.442977,76.847037) |
| 73 | 29.259045 | 77.138325 | Bhanauli | Samalkha | 0.06 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.259045,77.138325) |
| 151 | 29.387452 | 77.056736 | Chhajpur Khurd (41) | Bapoli | 0.06 | outside riverbed | [open](https://www.google.com/maps?q=29.387452,77.056736) |
| 238 | 29.315577 | 76.842760 | Kalkha (41) | Matlauda | 0.05 | outside riverbed | [open](https://www.google.com/maps?q=29.315577,76.842760) |
| 8 | 29.327837 | 76.884950 | Bhadaur (39) | Matlauda | 0.05 | outside riverbed | [open](https://www.google.com/maps?q=29.327837,76.884950) |
| 220 | 29.405348 | 77.104003 | Janbha (34) | Bapoli | 0.05 | outside riverbed | [open](https://www.google.com/maps?q=29.405348,77.104003) |
| 331 | 29.199078 | 76.881443 | Kakoda (84) | Israna | 0.04 | outside riverbed | [open](https://www.google.com/maps?q=29.199078,76.881443) |
| 131 | 29.420106 | 76.978796 | Panipat Taraf Ansar (CT) | Panipat | 0.04 | outside riverbed | [open](https://www.google.com/maps?q=29.420106,76.978796) |
| 270 | 29.439570 | 76.992573 | Azizullapur (5) | Panipat | 0.04 | outside riverbed | [open](https://www.google.com/maps?q=29.439570,76.992573) |
| 263 | 29.222034 | 76.952784 | Namonda (80) | Samalkha | 0.04 | outside riverbed | [open](https://www.google.com/maps?q=29.222034,76.952784) |
| 251 | 29.320016 | 76.734390 | Dumyana (56) | Matlauda | 0.04 | outside riverbed | [open](https://www.google.com/maps?q=29.320016,76.734390) |
| 125 | 29.410629 | 76.850367 | Mahayudinpur Thirana (9) | Matlauda | 0.04 | outside riverbed | [open](https://www.google.com/maps?q=29.410629,76.850367) |
| 232 | 29.328634 | 77.134276 | Rahimpur Kheri | Bapoli | 0.04 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.328634,77.134276) |
| 144 | 29.466952 | 76.840795 | Dharamgarh (1) | Matlauda | 0.04 | outside riverbed | [open](https://www.google.com/maps?q=29.466952,76.840795) |
| 174 | 29.243619 | 76.883775 | Bandh (87) | Israna | 0.04 | outside riverbed | [open](https://www.google.com/maps?q=29.243619,76.883775) |
| 2 | 29.330297 | 76.884491 | Bhadaur (39) | Matlauda | 0.04 | outside riverbed | [open](https://www.google.com/maps?q=29.330297,76.884491) |
| 123 | 29.441901 | 76.988513 | Azizullapur (5) | Panipat | 0.03 | outside riverbed | [open](https://www.google.com/maps?q=29.441901,76.988513) |
| 91 | 29.294246 | 76.818688 | Karad(64) | Israna | 0.03 | outside riverbed | [open](https://www.google.com/maps?q=29.294246,76.818688) |
| 181 | 29.440554 | 76.986082 | Azizullapur (5) | Panipat | 0.03 | outside riverbed | [open](https://www.google.com/maps?q=29.440554,76.986082) |
| 158 | 29.221861 | 76.808939 |  | (Sonipat district) | 0.03 | outside riverbed | [open](https://www.google.com/maps?q=29.221861,76.808939) |
| 115 | 29.439351 | 76.811064 | Sherah (2) | Matlauda | 0.03 | outside riverbed | [open](https://www.google.com/maps?q=29.439351,76.811064) |

</details>

<details>
<summary><b>15 largest pits</b></summary>

| Pit | Latitude | Longitude | Village | Tehsil | Area (ha) | Setting | Map |
|---:|---:|---:|---|---|---:|---|---|
| 156 | 29.171937 | 77.131965 | Raksehra (91) | Samalkha | 3.55 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.171937,77.131965) |
| 196 | 29.309761 | 77.156113 | Goela Kalan (60) | Bapoli | 3.01 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.309761,77.156113) |
| 186 | 29.247533 | 77.137696 | Bilaspur (79) | Samalkha | 2.89 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.247533,77.137696) |
| 172 | 29.482232 | 77.132627 | Rana Majra (93) | Bapoli | 2.78 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.482232,77.132627) |
| 225 | 29.315662 | 77.154784 | Mirzapur (59) | Bapoli | 2.56 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.315662,77.154784) |
| 297 | 29.421914 | 77.138046 | Tamsabad (29) | Bapoli | 2.51 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.421914,77.138046) |
| 14 | 29.323317 | 76.797418 | Alupur (60) | Matlauda | 2.40 | outside riverbed | [open](https://www.google.com/maps?q=29.323317,76.797418) |
| 102 | 29.283528 | 77.080216 | Begumpur Tahrpur (63) | Bapoli | 2.37 | outside riverbed | [open](https://www.google.com/maps?q=29.283528,77.080216) |
| 82 | 29.313471 | 76.827843 | Bhaupur (63) | Israna | 2.09 | outside riverbed | [open](https://www.google.com/maps?q=29.313471,76.827843) |
| 100 | 29.284073 | 77.128672 | Goela Khurd (61) | Bapoli | 1.95 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.284073,77.128672) |
| 17 | 29.322457 | 76.877395 | Bhadaur (39) | Matlauda | 1.71 | outside riverbed | [open](https://www.google.com/maps?q=29.322457,76.877395) |
| 136 | 29.419396 | 76.979033 | Panipat Taraf Ansar (CT) | Panipat | 1.65 | outside riverbed | [open](https://www.google.com/maps?q=29.419396,76.979033) |
| 127 | 29.410421 | 77.131842 | Tamsabad (29) | Bapoli | 1.47 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.410421,77.131842) |
| 86 | 29.304040 | 77.154129 | Goela Khurd (61) | Bapoli | 1.47 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.304040,77.154129) |
| 166 | 29.173876 | 77.130382 | Raksehra (91) | Samalkha | 1.35 | Yamuna riverbed | [open](https://www.google.com/maps?q=29.173876,77.130382) |

</details>

## Repository

| Path | What it is |
|---|---|
| [`notebooks/panipat_pit_detection.ipynb`](notebooks/panipat_pit_detection.ipynb) | Main workflow: Earth Engine features → download → features, training, pit outlines, gallery, maps, pit history, statistics, exports |
| [`gee/mine_pit_detection.js`](gee/mine_pit_detection.js) | The same idea entirely in the Earth Engine Code Editor (no download), with click-to-label layers — see [Use it in Google Earth Engine only](#use-it-in-google-earth-engine-only-no-python) |
| [`data/`](data/) | District, tehsil, village and neighbouring-district boundaries, registered ponds, roads and canals of Panipat (GeoJSON + Earth Engine upload files) |
| [`tools/get_boundaries.py`](tools/get_boundaries.py) | Makes the tehsil and village boundary files for any Indian district: `python tools/get_boundaries.py Sonipat` |
| [`tools/get_hsac_ponds.py`](tools/get_hsac_ponds.py) | Downloads the registered ponds (HSAC pond inventory) of any Haryana district: `python tools/get_hsac_ponds.py Sonipat` |
| [`tools/get_osm_roads.py`](tools/get_osm_roads.py) | Downloads the roads and highways (OpenStreetMap) of any district, after `get_boundaries.py`: `python tools/get_osm_roads.py Sonipat` |
| [`tools/get_hsac_canals.py`](tools/get_hsac_canals.py) | Downloads the canals and drains (HSAC / Irrigation Department) of any Haryana district, after `get_boundaries.py`: `python tools/get_hsac_canals.py Sonipat` |
| [`tools/pit_verify.py`](tools/pit_verify.py) | Optional stage 2 for high-resolution imagery: confirm / outline candidates with a YOLOv8 detector |
| [`tools/export_jpgs.py`](tools/export_jpgs.py) | Saves every figure of an executed notebook as JPG, plus a numbered full-resolution pit map with tehsil / village boundaries and a latitude / longitude grid, coordinate tables and a pits-per-tehsil table |

## How it works

1. **Earth Engine** builds features for the analysis window and the same months a year earlier:
   cloud-free Sentinel-2 statistics (10 m vegetation / water / bare-soil indices), Sentinel-1 radar,
   Dynamic World land-cover shares, four building sources that "vote" (Open Buildings, WorldCover,
   Dynamic World, radar double bounce), the Yamuna riverbed zone, and AlphaEarth similarity to typical
   pits / buildings / water / crops.
2. The features are **downloaded** as one GeoTIFF (resumable, with timeouts).
3. **Locally at 10 m**: context features (each pixel vs. its 130 m surroundings), sub-pixel
   water / bare / vegetation / building fractions, automatic training examples (registered ponds as
   "not pit" examples) plus your own points, a gradient-boosting model validated on 5 km blocks it
   never trained on.
4. **Pit objects**: hysteresis outlines; building pixels removed; pits with an average probability
   below 0.65 dropped; tiny pits kept only when very confident; canals, drains, roads and thin lines
   dropped; registered ponds, likely village ponds, detections in canal or road corridors and the
   Yamuna's own channel (riverbed water bodies of 5 ha or more) removed to `excluded.csv`; riverbed pits
   tagged.
5. Optional **2.5 m super-resolution** (SEN2SR) for close-ups and refined outlines.

## Run it

1. Open `notebooks/panipat_pit_detection.ipynb` in Jupyter or Google Colab.
2. In **Settings**, set `PROJECT_ID` to the Google Cloud project you use with Earth Engine.
   Keep `AREA = 'test'` for a quick first run (14 × 13 km along the Yamuna east of Panipat, a few
   minutes), then use `AREA = 'district'` (≈ 0.5 GB download, roughly 10–20 minutes).
3. Run all cells.

**Improve the results:** in section 3, zoom the Satellite basemap to 16–18 and save 30–50 points on
real pits and 30–50 on false detections (canals, fish ponds, roads, roofs). Re-run from section 6 —
no new download. The validation table then shows real accuracy against your points and the
threshold is tuned on them.

Optional super-resolution: `pip install sen2sr mlstac torch`, then re-run from section 10.

## Use it in Google Earth Engine only (no Python)

[`gee/mine_pit_detection.js`](gee/mine_pit_detection.js) does the detection entirely in the
[Earth Engine Code Editor](https://code.earthengine.google.com) — nothing to install or download.
You only need an Earth Engine account with a Cloud project (free for non-commercial use:
[register here](https://code.earthengine.google.com/register)).

**1. Open the script**

1. Go to [code.earthengine.google.com](https://code.earthengine.google.com) and pick your Cloud project
   (top-right, next to your profile picture).
2. **Scripts** panel (left) → **NEW** → **File**, name it e.g. `panipat_pits`.
3. Paste the whole of `gee/mine_pit_detection.js` into the editor and press **Save**.

**2. Choose the area and dates** in the `CONFIG` block at the top of the script:

| Setting | Default | Change it to… |
|---|---|---|
| `state`, `districts` | `'Haryana'`, `[['Panipat']]` | another district, e.g. `[['Sonipat', 'Sonepat']]` (alternative spellings), or several: `[['Panipat'], ['Karnal']]` |
| `endDate` | `null` (today) | a fixed date such as `'2026-09-30'`, so later runs use the same period |
| `monthsBack` | `3` | the length of the window; it is always compared with the same months a year earlier |
| `probabilityThreshold` | `0.6` | lower (e.g. `0.4`) to catch more pits with more false alarms, higher for fewer |
| `minPitAreaHa` | `0.05` | the smallest pit kept (0.05 ha = 5 Sentinel-2 pixels) |
| `minPitProb` | `0.65` | the lowest average probability a pit may have; raise it for fewer, surer pits |
| `riverChannelMinHa` | `5` | water bodies this large in the Yamuna bed count as the river's channel, not pits |

To use your own area instead of a district, draw a polygon with the geometry tools and name the
import `aoi`.

**3. Run** — press **Run** and wait 1–3 minutes until the panel (bottom-left) says *Model ready*.

- **Map** (Layers menu, top-right of the map): *Pit pixels* (magenta), *New pits since this time last
  year* (red), *Removed: looks like a building* (grey) and *District boundaries* are on; *Pit probability*,
  *Model unsure*, Sentinel-2, radar, Dynamic World, building and riverbed layers can be switched on.
  Zoom to level 13+ to see small pits.
- **Console** (right): the period, Sentinel scene counts, training samples, a validation table per
  threshold, and the features the model relies on most.
- **Panel**: **Outline + count pits in current view** (zoom 13+) draws the outlines and counts pits —
  total, new, under 0.5 ha, hectares. Tick **Click map to find pits similar to the clicked one** and
  click any pit to highlight everything that looks like it (AlphaEarth similarity).

**4. Download every pit with its latitude and longitude** — open the **Tasks** tab (top-right) and
press **RUN** on:

| Task | Saved to Google Drive folder `GEE_pits` |
|---|---|
| `panipat_pits_list_<dates>` | CSV: district, tehsil, village, **latitude, longitude**, area_ha, prob, new_pit, water_frac, built_frac, compact, `maps_link` (Google Maps) |
| `panipat_pits_polygons_<dates>` | Shapefile of the pit outlines with the same attributes (change `fileFormat` to `'KML'` for Google Earth) |
| `panipat_pit_probability_<dates>` | GeoTIFF of the pit probability (0–100) for every 10 m pixel |
| `panipat_excluded_<dates>` | Only with `pondAsset`, `canalAsset` or `roadAsset` (below): CSV of the detections removed, with the reason (`category`), the register's `hsac_id`, name and type, the canal's name, or the road's number, name and class |

**Optional — today's district outline, tehsil and village boundaries in Earth Engine.** Earth Engine has
no tehsil or village layer for India and its district layer (FAO GAUL 2015) misses 7 % of Panipat, so
upload the repository's ones once:

1. Unzip [`data/panipat_boundaries_shp.zip`](data/panipat_boundaries_shp.zip).
2. **Assets** tab (top-left) → **NEW** → **Shape files** → select the five `panipat_tehsils.*` files
   (`.shp`, `.shx`, `.dbf`, `.prj`, `.cpg`) → name the asset `panipat_tehsils` → **UPLOAD**. Do the same
   for the five `panipat_villages.*` files as `panipat_villages` and the five `panipat_district.*` files
   as `panipat_district`. The upload runs in the Tasks tab.
3. In `CONFIG`, set `districtAsset: 'projects/<your-project>/assets/panipat_district'`,
   `tehsilAsset: 'projects/<your-project>/assets/panipat_tehsils'` and
   `villageAsset: 'projects/<your-project>/assets/panipat_villages'`, then **Run**.

The script then analyses today's district (plus `districtBuffer`, 250 m) instead of the GAUL outline,
the map shows *Tehsil boundaries* (orange) and *Village boundaries* (white, in the Layers menu), the
panel counts pits per tehsil, and the exported CSV and shapefile fill the `tehsil` and `village` columns.
For another district, make its files first with `python tools/get_boundaries.py <District>`.

**Optional — remove registered ponds in Earth Engine.** Upload
[`data/panipat_hsac_ponds_shp.zip`](data/panipat_hsac_ponds_shp.zip) the same way (unzip, **NEW** →
**Shape files**, the five `panipat_hsac_ponds.*` files, asset name `panipat_hsac_ponds`) and set
`pondAsset: 'projects/<your-project>/assets/panipat_hsac_ponds'`. Registered ponds then become "not pit"
training examples, detections with at least `pondMatch` (25 %) of their area in a registered pond (widened
by `pondTolerance`, 15 m) leave the pit list and the counts (registered "Mining Pit" entries excepted:
`pondKeepTypes`), the *Registered ponds (HSAC)* layer is added,
and the Tasks tab gets an `excluded` export. For another Haryana district run
`python tools/get_hsac_ponds.py <District>` first.

**Optional — remove pits on roads in Earth Engine.** Upload
[`data/panipat_roads_shp.zip`](data/panipat_roads_shp.zip) the same way (the five `panipat_roads.*` files,
asset name `panipat_roads`) and set `roadAsset: 'projects/<your-project>/assets/panipat_roads'`. Each road
becomes a corridor of its class's half-width (`half_width`), pit candidates on roads are not used as pit
examples, detections with at least `roadMatch` (25 %) of their area on a corridor leave the pit list,
and *Main roads and highways* / *Road corridors* layers are added. The script uses the class widths only;
the notebook also widens roads that are being built or widened. For another district run
`python tools/get_osm_roads.py <District>` first.

**Optional — remove pits in canals in Earth Engine.** Upload [`data/panipat_canals.csv`](data/panipat_canals.csv)
(**NEW** → **CSV file**, asset name `panipat_canals`; the `geometry` column is WKT) and set
`canalAsset: 'projects/<your-project>/assets/panipat_canals'`. Canals, drains and canal footprints become
corridors (`half_width_m`), pit candidates in them are not used as pit examples, detections with at
least `canalMatch` (25 %) of their area in a corridor leave the pit list, and *Canals and drains* /
*Canal corridors* layers are added. For another Haryana district run
`python tools/get_hsac_canals.py <District>` first.

**5. Improve it by labelling** — after the first run the drawing tools (top-left of the map) contain
two point layers, `pits` (red) and `nonPits` (blue). Zoom the Satellite basemap to 16–18, start where
the *Model unsure* layer is, select a layer and click to add points — real pits as `pits`, false
detections (roofs, canals, fish ponds, roads, brick kilns, sand bars) as `nonPits`. 30–50 of each is a
good start. Press **Run** again: the Console now shows a table *Against YOUR points (real accuracy)* —
set `probabilityThreshold` to the row with the best balance. The points are saved with the script, so
keep adding where it is still wrong. With ~100+ of each, `useAutoLabels: false` learns from your
points only.

**If you get "Computation timed out"**: halve `trainingCells` and `riverCells`, or run the
`cache_training_samples_v3_<dates>` task in the Tasks tab, put its asset ID in `samplesAsset`, set
`endDate` to the printed end date and run again — training then takes seconds.

**GEE-only vs the notebook:** the Code Editor script uses the same data, building knowledge,
automatic labels, context features and block validation, with a Random Forest. The notebook adds
gradient boosting at full 10 m, sub-pixel fractions, removal of canal / drain / road fragments and thin
lines, village-pond and riverbed tags, the gallery, pit histories and JPGs — so its counts are cleaner.
The results in this repository come from the notebook.

## Limitations

- Sentinel-2 pixels are 10 m: about 3 pixels (≈ 0.03 ha, e.g. 15 × 20 m) is the smallest pit
  detected reliably.
- Without your labels the model learns from rules, so accuracy is unknown.
- Riverbed detections can be sand mining but also natural sand bars, flood pools or bank erosion. The
  Yamuna moves every monsoon, so riverbed water on last year's ground is marked "new" even when it is
  the river; water bodies of 5 ha or more there are removed as the channel (`RIVER_CHANNEL_MIN_HA`), but
  smaller pieces of channel water can remain.
- AlphaEarth embeddings are annual (2025 for this run); recent change comes from Sentinel-2,
  Sentinel-1 and Dynamic World.
- Super-resolved images are an AI estimate, useful for viewing — not real high-resolution imagery.
- The analysis area is today's district boundary (geoBoundaries, 2021) plus 250 m; the Earth Engine
  script uses Earth Engine's FAO GAUL 2015 outline unless `districtAsset` is set (see above).
- Only ponds in the HSAC register are removed by name; unregistered ponds and fish ponds can remain.
  Training on registered ponds also teaches the model what a pond looks like, which removes many
  unregistered ponds — but a water-filled pit looks the same at 10 m. To only remove registered ponds,
  set `SAMPLES['registered_pond'] = 0` in the notebook (`pondSamples: 0` in the Earth Engine script).
- Roads come from OpenStreetMap, which is mostly complete for Panipat's highways and village roads but
  not for every new road. A detection is removed when at least a quarter of it is on a road corridor;
  excavations *beside* a road (for example borrow pits along a road being widened) stay in the list
  with `nearest_road` and `road_distance_m`, so they can be filtered or reviewed.
- Canals come from the Irrigation Department's network as published by HSAC; field channels (watercourses)
  are not in it, and a few canals carry an inconsistent category, which is why the corridor width also
  uses the discharge.
- Tehsil boundaries are from 2018 and village boundaries from the 2011 Census; their edges do not match
  exactly, and newer administrative changes are not included. Pits right at a boundary may be assigned
  to the neighbouring unit.

## Data and credits

Copernicus Sentinel-1 and Sentinel-2; Cloud Score+; Google Dynamic World; Google DeepMind AlphaEarth
Foundations satellite embeddings; Google Open Buildings 2.5D Temporal; ESA WorldCover; WWF HydroSHEDS
Free-Flowing Rivers; FAO GAUL 2015 boundaries — all via Google Earth Engine. District (2021), tehsil
(2018) and village (2011 Census) boundaries: [geoBoundaries](https://www.geoboundaries.org) gbOpen India
ADM2 / ADM3 / ADM5
(source: Pathways Data, Local Government Directory and Census of India), Open Data Commons ODbL 1.0. Registered
ponds: Haryana Pond & Waste Water Management Authority / Haryana Space Applications Centre (HSAC) pond
inventory, [Pond MIS](https://hsac.org.in/pond_mis/), from HSAC's public map service. Canals and drains:
Haryana Irrigation & Water Resources Department canal network and HSAC canal layer, from HSAC's public map
services. Roads: © OpenStreetMap
contributors, Open Data Commons ODbL 1.0, via the Overpass API. Optional SEN2SR
super-resolution by ESA OpenSR. The detection ideas (context, shape, block validation, local labels)
were informed by [BapakmuLah/Satellit-Based-Open-Pit-Mining-Detection](https://github.com/BapakmuLah/Satellit-Based-Open-Pit-Mining-Detection).
