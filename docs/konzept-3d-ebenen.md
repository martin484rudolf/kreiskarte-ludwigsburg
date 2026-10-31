# Konzept: Kreiskarte Ludwigsburg als geschichtete 3D-Infrastrukturkarte

Stand: 28.10.2026. Ausgangspunkt: `index.html` (reine SVG-Karte mit Glasfaser-Ebene
aus dem Breitbandatlas der BNetzA, Commit 2871caa) und das Schwesterprojekt
`vaihinger-nacht-der-demokratie` (React + three.js, eigene 3D-Karte der Vaihinger
Innenstadt aus LGL-Daten: DGM1-Gelände, DOP20-Luftbild, LoD2-Gebäude als GLB).

Ziel: Aus der flachen Kreiskarte wird eine Karte, auf der man **Ebenen umschalten**
(Zuständigkeit, Infrastruktur, Qualität/Ressourcen, Risiko) und bei Bedarf in eine
**3D-Ansicht mit Höhenlinien** wechseln kann — bis hin zu einer Projektion auf ein
physisches 3D-Modell für eine Ausstellung.

---

## 1. Was heute schon da ist (beide Repos)

| Baustein | Kreiskarte | Vaihingen-App |
|---|---|---|
| Darstellung | Reine SVG, Hand-gebaute Geometrie, kein Framework | three.js, eigene Szene (`src/karte/szene.ts`) |
| Datenbasis | LGL-Gemeindegrenzen (dl-de/by-2-0), links.json | LGL: DGM1, DOP20, LoD2 (dl-de/by-2-0), OSM-Wegenetz (ODbL) |
| Ebenen-Umschaltung | Glasfaser an/aus, Klasse + Ansicht + Deckkraft | Heute/Gartenschau (Textur-Tausch), Gebäude-Abrissliste |
| Externe Dienste | BNetzA-Breitbandatlas per WMS-GetMap (EPSG:25832) | — |
| 3D | nein | ja, lokal vorberechnete Assets (`public/karte/`) |

Die Glasfaser-Ebene ist das Muster, das sich verallgemeinern lässt: **Knopf oben
links → Panel mit Ebenen-Auswahl → WMS-Bild oder eigene Ebene wird unter/über die
Gemeindegrenzen gelegt, Deckkraft regelbar, Quelle mit Lizenzzeile.** Genau diese
Mechanik tragen wir auf alle weiteren Ebenen.

---

## 2. Ebenen-Modell (Vorschlag)

Vier Umschaltgruppen, dazu die bestehende Zuständigkeitskarte als Basis:

1. **Zuständigkeit** (Basis, immer an)
   - Gemeinden (heute), Nachbarkreise (heute)
   - Neu: **Wahlkreise** — Landtagswahlkreise 2021 und Bundestagswahlkreise 2025
     als GeoJSON vom Statistischen Landesamt BW
     ([Wahlkarten-Download](https://www.statistik-bw.de/service/karten-und-atlanten/wahlkreiskarten/)),
     zugeschnitten auf den Landkreis. Damit sieht man, welche Gemeinde zu welchem
     MdL/MdB-Wahlkreis gehört.

2. **Infrastruktur / Versorgung** (Qualität pro Fläche)
   - Glasfaser / Breitband (heute: BNetzA-Breitbandatlas, WMS, 100-m-Raster oder je Gemeinde)
   - Neu: **Mobilfunk (5G/4G)** — gleicher BNetzA-WMS, andere Layer
     (`Breitband_Mobilfunk__...`); Ebenennamen per GetCapabilities prüfen.

3. **Risiko / Umwelt**
   - **Hochwassergefahrenkarte (HWGK)**: LUBW-WMS, Layer überflutungsflächen /
     -tiefen für HQ10, HQ50, HQ100, HQextrem
     ([Kartendienste der LUBW](https://www.lubw.baden-wuerttemberg.de/umweltinformationssystem/kartendienste-der-lubw);
     Dienst über `rips-gdi.lubw.baden-wuerttemberg.de`, EPSG:25832).
     Für Vaihingen konkret relevant: Enzauen, HQ100 = festgesetztes
     Überschwemmungsgebiet (Stadt Vaihingen: [Hochwasserinformation](https://www.vaihingen.de/rathaus-service/buergerservice/hochwasserinformation)).
   - Neu: **festgesetzte Überschwemmungsgebiete (ÜSG)** — LUBW-WMS
     `WMS Überschwemmungsgebiet`, dazugehöriger WFS für exakte Grenzen.
   - Neu: **Starkregen** — Stadt Vaihingen veröffentlicht
     [Starkregengefahrenkarten](https://www.vaihingen.de/leben-wohnen/natur-klimaschutz/starkregengefahrenkarten);
     andere Kreiskommunen je nach Verfügbarkeit.

4. **3D / Gelände**
   - Höhenlinien (Höhenlinien-Archiv von basemap.de, oder selbst aus DGM1 gerechnet)
   - 3D-Ansicht mit Gelände — siehe Abschnitt 3.

Alle WMS-Ebenen bekommen dieselbe Panel-Mechanik wie die Glasfaser-Ebene:
Klasse wählen, Deckkraft, Legende (GetLegendGraphic), Quellenzeile mit Lizenz.
Ebenennamen ändern sich mit dem Datenstand — jeweils gegen GetCapabilities prüfen
(steht schon im Kommentar der Glasfaser-Ebene).

**Lizenzen:** LGL/BNetzA/LUBW/Statistisches Landesamt — Quellenangabe jeweils in
der Ebenen-Fußzeile, dl-de/by-2-0 bei LGL-Daten vermerken.

---

## 3. Die 3D-Frage: Selbstbau vs. fertig — Recherche-Ergebnis

### Fertig nutzbare Bausteine (kostenlos, ohne API-Key)

| Baustein | Quelle | Für uns |
|---|---|---|
| 3D-Gelände (DGM5, deutschlandweit) | [basemap.de 3D Gelände](https://basemap.de/produkte-und-dienste/3d/) — 3D-Tiles + quantized-mesh | Gelände für den ganzen Kreis, amtlich (AdV) |
| 3D-Gebäude (LoD2, deutschlandweit) | [basemap.de 3D Gebäude](https://basemap.de/produkte-und-dienste/3d/) — 3D-Tiles, jährl. zum 1.3. | Gebäude mit echten Dachformen |
| Fertiger Viewer | [web3d.basemap.de/maplibre/lod2.html](https://web3d.basemap.de/maplibre/lod2.html) | Referenz-Implementierung (MapLibre + 3D-Tiles-Addon) |
| 2D-Basiskarte (Vektor) | basemap.de Web Vektor, Style-JSON via `sgx.geodatenzentrum.de` | Straßennamen, Ortskerne unter den Ebenen |
| Höhen-Tiles global | [Mapterhorn](https://tiles.mapterhorn.com/tilejson.json) (Terrarium, bis z12 global, 1 m wo Länderdaten vorliegen), [AWS Terrain Tiles](https://s3.amazonaws.com/elevation-tiles-prod/terrarium/{z}/{x}/{y}.png) | MapLibre-`raster-dem`-Quelle, direkt ohne Schlüssel |
| Höhen-Tiles selbst gebaut | LGL **DGM1** (1 m) und **DGM025** (0,25 m!) als GeoTIFF/XYZ vom [Open GeoData Portal](https://opengeodata.lgl-bw.de/) | eigene Terrarium/Mapbox-RGB-Tiles via GDAL (`rio-rgbify` + `gdal2tiles`), gehostet als statische Dateien — analog zu unserer heutigen `gelaende.bin`-Pipeline |
| Vector-Tiles-Basemap (OSM) | [OpenFreeMap](https://github.com/hyperknot/openfreemap), Protomaps, basemap.de | falls basemap.de-Stil nicht passt |

**Wichtig:** Das LGL stellt DGM1 und seit März 2026 auch DGM025 (25 cm!) offen als
GeoTIFF bereit. Wir haben dieselben Rohdaten schon lokal in `Volt\LGL-Daten` —
die Pipeline aus dem Vaihingen-Projekt (`tools/gelaende.py`: DGM1 → Heightmap,
`tools/gebaeude.py`: LoD2 → GLB) ist direkt wiederverwendbar, nur der Ausschnitt
wächst von 1,35 km² (Innenstadt) auf ~810 km² (ganzer Kreis).

### Selbstbau vs. fertig — Abwägung

| Kriterium | Fertig (basemap.de 3D + MapLibre) | Selbstbau (eigene Assets, three.js) |
|---|---|---|
| Aufwand | gering: MapLibre einbinden, 3D-Tiles-Quelle konfigurieren | mittel: Python-Pipeline (GDAL/gltfpack) für DGM1/DGM025 + LoD2, GLB/Heightmap erzeugen |
| Datenaktualität | LoD2 jährlich (1.3.), Gelände mehrjährig | eigener Stand, wir steuern selbst (wir haben LoD2 2026 + DGM1 2019 lokal) |
| Lizenz | GeoBasis-DE / dl-de/by-2-0, Namensnennung | dl-de/by-2-0, Namensnennung |
| Offline (PWA) | nein — Tiles kommen live vom BKG | ja — alles in `public/karte/`, passt zum Konzept der Vaihingen-App |
| Größenbudget | klein (nur Code, Tiles streamen) | groß: DGM1 für 810 km² = 810 Mio. Zellen → Heightmap 5 m ≈ 26 MB (Uint16), 1 m ≈ 650 MB → **nicht machbar als eine Datei** |
| Handy-Performance | sehr gut (MapLibre + GPU-Terrain) | kritisch: 810 km² in drei.js als eine Szene = zu viel; müsste gekachelt/geladen werden |
| Konsistenz mit Vaihingen-App | andere Technik (MapLibre statt three.js) | gleiche Technik, gleiche Pipeline, gleiche Lizenzzeilen |
| Ausstellung (physisches Modell) | nein | ja — siehe Abschnitt 4 |

**Empfehlung: zweigleisig.**

- **Online-Karte (kreiskarte-ludwigsburg):** MapLibre GL JS + basemap.de 3D
  (Gelände + LoD2 als 3D-Tiles) + eigene Ebenen (Gemeindegrenzen als GeoJSON,
  BNetzA-WMS, LUBW-HWGK als WMS-Overlay). Fertig, kostenlos, amtlich, performant —
  und die WMS-Ebenen (Glasfaser, Hochwasser) lassen sich in MapLibre genauso als
  `raster`-Quelle einhängen wie heute als SVG-`<image>`.
- **Selbstbau-Pipeline (Wiederverwendung aus Vaihingen):** behalten und auf den
  Kreis ausdehnen, aber **nicht** als eine große Szene ausliefern, sondern als
  Kacheln (z. B. 10-km-Kacheln oder pro Gemeinde) — nur für die Ausstellung und
  für eine optionale Offline-3D-Ansicht der Vaihingen-Innenstadt (die existiert
  ja schon in der anderen App).

Die MapLibre-Route ist kein "entweder/oder": MapLibre rendert 2.5D-Gelände
(raster-dem), und das basemap.de-3D-Tileset liefert echte LoD2-Gebäude. Beides
lässt sich kombinieren (Gebäude-Tiles auf eigenem oder amtlichem Gelände).

### Höhenlinien

- Fertig: Höhenlinien-Archiv von basemap.de (Vektor, im "Relief"-Stil enthalten).
- Selbst: aus DGM1 mit GDAL (`gdal_contour`) — die Pipeline steht, 1-m-DGM für den
  Kreis ist lokal vorhanden. Höhenlinien als GeoJSON in die Karte, Stil: feine graue
  Linien, 10-m-Äquidistanz, 50-m-Hauptlinien beschriftet.

### Ausstellung / physisches 3D-Modell

Zwei Stufen, beide mit schmalem Budget machbar:

1. **Bildschirm/Beamer in der Ausstellung:** die Online-Karte (MapLibre) im
   Vollbild, 3D-Ansicht, Kamerafahrt entlang der Enz. Kosten: 0.
2. **Physisches Reliefmodell:** DGM1-Ausschnitt (z. B. Vaihingen mit Enztal oder
   der ganze Kreis grob) als Heightmap → 3D-Druck (FDM, weißes Filament, ~0,5–1 m
   Modell: ca. 100–300 € beim Dienstleister, oder eigener Drucker) oder
   CNC-gefräst. Darauf projiziert (Beamer von oben) oder beklebt mit ausgedruckter
   Karte (Luftbild + Ebenen). Daten: dieselbe `gelaende.bin`-Heightmap, nur anderer
   Ausschnitt — die Vaihingen-Pipeline erzeugt das direkt.

---

## 4. Konkrete nächste Schritte (Vorschlag, Reihenfolge)

1. **Ebenen-Panel verallgemeinern** in `index.html`: Glasfaser-Code in einen
   wiederverwendbaren Ebenen-Baustein überführen (heute schon fast so gebaut),
   zweite WMS-Ebene (Mobilfunk) und dritte (HWGK) ergänzen. Reine Handarbeit,
   kein Framework-Wechsel nötig für 2D.
2. **Wahlkreise als GeoJSON** vom Statistischen Landesamt holen, auf den Kreis
   zuschneiden, als vierte Zuständigkeits-Ebene einhängen.
3. **3D-Prototyp als eigene Seite** `3d.html` im selben Repo — **umgesetzt als Prototyp**
   (28.10.): MapLibre + basemap.de Web Vektor (2D) + DGM5-Gelände + LoD2-Gebäude
   (3D-Tiles, ab Zoom 15) + Gemeinde-/Kreisgrenzen als GeoJSON-Overlay
   (`data/*.geojson`, gezogen aus BKG-WFS `wfs_vg250-ew`, dl-de/by-2-0) +
   WMS-Ebenen Glasfaser (BNetzA), Mobilfunk 5G (BNetzA), HWGK HQ10/HQ100/HQextrem
   und festgesetzte Überschwemmungsgebiete (LUBW) umschaltbar.
   Getestete Dienst-Endpunkte:
   - Gelände: `https://sgx.geodatenzentrum.de/gdz_basemapde_3d_gelaende/dgm5_3857_rgb.json`
   - Gebäude: `https://sgx.geodatenzentrum.de/gdz_basemapde_3d_gebaeude/lod2_3857_null.json`
     (MapLibre-Einbindung über `Maplibre3DTiles.js` vom BKG, wie im offiziellen Beispiel)
   - HWGK Überflutungsflächen: LUBW-WMS `UIS_0100000026600001` (HQ10), `...003` (HQ100),
     `...004` (HQextrem); Tiefen: `UIS_0100000039300001–004`; Layer jeweils `0`, EPSG:25832
   - ÜSG festgesetzt: LUBW-WMS `UIS_0100000003900001`, Layer `1` (HQ100-Gebiet)
   - Mobilfunk 5G: BNetzA-WMS Layer
     `Breitband_Mobilfunk_Privathaushalte__100m_Gitter__Alle_Anbieter_5G63805`
4. **Selbstbau-Pipeline ausweiten** (Vaihingen-Repo, `tools/`): Ausschnitt auf
   den Kreis erweitern, Kachel-Schema definieren, Höhenlinien aus DGM1 rechnen.
   Nur wenn Ausstellung/Offline-3D gewünscht.
5. **Entscheidung mit Martin:** 3D online ja/nein, Ausstellung ja/nein,
   Budget für physisches Modell.

## 5. Offene Fragen

- Soll die 3D-Ansicht in die bestehende `index.html` integriert werden (ein Repo,
  eine Karte, 2D/3D-Umschalter) oder als eigene Seite `3d.html` (empfohlen:
  `index.html` bleibt schlank fürs Haiilo-iFrame-Widget)?
- Welche Ebenen sind für den Volt-Hub tatsächlich relevant — nur Infrastruktur
  (Glasfaser, Mobilfunk) oder auch Risiko (HWGK) und Politik (Wahlkreise)?
- Ausstellung: Bildschirm-Lösung reicht, oder physisches Reliefmodell gewünscht?
  Falls ja: welcher Ausschnitt (Vaihingen? ganzer Kreis?) und welches Budget?

## Quellen

- LGL Open GeoData Portal: https://opengeodata.lgl-bw.de/ (DGM1, DGM025, DOP20, LoD2; dl-de/by-2-0)
- LGL Open Data: https://www.lgl-bw.de/Produkte/Open-Data/index.html
- basemap.de 3D (BKG): https://basemap.de/produkte-und-dienste/3d/
- basemap.de Web Vektor: https://basemap.de/produkte-und-dienste/web-vektor/
- basemap.de 3D Viewer (MapLibre, LoD2): https://web3d.basemap.de/maplibre/lod2.html
- LUBW Kartendienste (HWGK/HWRK/ÜSG): https://www.lubw.baden-wuerttemberg.de/umweltinformationssystem/kartendienste-der-lubw
- Hochwasserportal BW: https://www.hochwasser.baden-wuerttemberg.de/hochwassergefahrenkarten
- Statistisches Landesamt BW, Wahlkarten: https://www.statistik-bw.de/service/karten-und-atlanten/wahlkreiskarten/
- BNetzA Breitbandatlas (bereits eingebunden): https://gigabitgrundbuch.bund.de/GIGA/DE/Breitbandatlas/start.html
- Mapterhorn Terrain-Tiles: https://tiles.mapterhorn.com/tilejson.json
- AWS Terrain Tiles (Terrarium): https://registry.opendata.aws/terrain-tiles/
- MapLibre GL JS 3D Terrain: https://maplibre.org/maplibre-gl-js/docs/examples/3d-terrain/
- OpenFreeMap (OSM-Vektor, kostenlos): https://github.com/hyperknot/openfreemap
- Stadt Vaihingen: Hochwasserinformation https://www.vaihingen.de/rathaus-service/buergerservice/hochwasserinformation · Starkregengefahrenkarten https://www.vaihingen.de/leben-wohnen/natur-klimaschutz/starkregengefahrenkarten
