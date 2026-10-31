# Kreiskarte Ludwigsburg

Klickbare Karte der 39 Städte und Gemeinden im Landkreis Ludwigsburg für den Volt-Hub in Haiilo.

- `index.html` – nur die Karte, zum Einbetten im Haiilo-Widget iFrame
- `links.json` – Adressen: je Gemeinde eine Haiilo-Seite, je Nachbarkreis der Hub der Volt-Gruppe
- `vollansicht.html` – Einzelseite mit Karte und Steckbrief
- `3d.html` – Prototyp 3D-Karte (MapLibre): Gelände DGM5, LoD2-Gebäude, Ebenen Glasfaser/Mobilfunk/HWGK/ÜSG
- `data/` – Gemeinde- und Kreisgrenzen als GeoJSON (Quelle BKG VG250, dl-de/by-2-0), gezogen per WFS `wfs_vg250-ew`
- `docs/konzept-3d-ebenen.md` – Konzept: Ebenen-Modell, Selbstbau vs. fertig, Ausstellung

## Adressen eintragen

In `links.json` beim passenden Eintrag die Adresse zwischen die Anführungszeichen bei `"url"` setzen. Leer = Fläche nicht anklickbar.
`"ziel": "_top"` öffnet den Link im Haiilo-Fenster selbst, `"_blank"` in einem neuen Tab.

Quellen: Grenzen LGL-BW (2026), Datenlizenz Deutschland – Namensnennung 2.0 · Gemeindedaten Landratsamt Ludwigsburg · Einwohner Statistisches Landesamt BW, Stand 31.03.2026.

`3d.html` zusätzlich: Hintergrund/Gelände/Gebäude basemap.de (© GeoBasis-DE / BKG) · Grenzen © BKG (VG250), dl-de/by-2-0 · Glasfaser/Mobilfunk Breitbandatlas © BNetzA · Hochwasser HWGK/ÜSG LUBW Baden-Württemberg.
