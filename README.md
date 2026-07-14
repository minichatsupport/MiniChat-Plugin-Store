# MiniChat Plugin Store

Der offizielle Plugin-Katalog für [MiniChat](https://github.com/minichatsupport/MiniChat).
Dieses Repo **ist** der Store: Die App lädt den signierten Index direkt von
GitHub — es läuft kein Server.

## Wie die App diesem Repo vertraut

Die Sicherheit hängt nicht an GitHub, sondern an Kryptographie:

1. **`store-index.json`** ist mit einem ed25519-Schlüssel signiert, dessen
   öffentliche Hälfte fest in der App steckt. Ein manipulierter Index wird
   von der App verworfen — egal woher er kommt.
2. Jeder Katalogeintrag trägt die **sha256** seines Pakets. Die App hasht
   jeden Download nach und verweigert bei Abweichung die Installation —
   eine ausgetauschte `.mcplugin`-Datei bewirkt nichts.
3. **`"revoked": true`** ist der Killswitch: Beim nächsten Store-Sync
   deinstalliert die App das Plugin lokal.

Der private Signierschlüssel liegt **nicht** in diesem Repo und darf es nie.

## Struktur

```
store-plugins.json   ← der editierbare Katalog (Quelle)
store-index.json     ← der signierte Index (generiert — NIE von Hand editieren)
plugins/             ← die .mcplugin-Pakete, versionierte Dateinamen
```

Versionierte Dateinamen (`<id>-<version>.mcplugin`) sind Absicht: Ein Update
bekommt eine **neue** Datei, die alte URL bleibt gültig, bis der neue Index
signiert und gepusht ist. So zeigt der Index nie auf Bytes, die nicht zu
seiner sha256 passen.

## Ein Plugin veröffentlichen / aktualisieren

Werkzeug: `mcplugin` (liegt im MiniChat-Hauptrepo unter `Plugin store/tools/`).

```powershell
# 1. Paket bauen (validiert zuerst; druckt sha256 + Größe)
mcplugin package mein-plugin/ plugins/com.example.mein-1.0.0.mcplugin

# 2. Katalogzeile erzeugen und in store-plugins.json einfügen/ersetzen
mcplugin catalog-entry plugins/com.example.mein-1.0.0.mcplugin `
  https://raw.githubusercontent.com/minichatsupport/MiniChat-Plugin-Store/main/plugins/com.example.mein-1.0.0.mcplugin

# 3. Index neu signieren (Keyfile liegt außerhalb jedes Repos!)
mcplugin sign-index store-plugins.json C:\Users\...\minichat-secrets\store-signing-key.txt store-index.json

# 4. Push — damit ist es live
git add -A ; git commit -m "store: mein-plugin 1.0.0" ; git push
```

## Icons & Bewertungen

- **`icons/<id>.png`** + `iconUrl` im Katalog = Artwork im Store. Dieselbe
  `icon.png` gehört auch ins Plugin-Paket (Root), dann zeigt sie der
  Plugin-Manager nach der Installation.
- **`rating` / `ratingCount`** sind aktuell **kuratierte Werte** — es gibt
  keinen Server, der echte User-Bewertungen sammeln könnte. Sobald das Backend
  läuft, liefert es echte Werte über dieselben Felder.

## Ein Plugin sperren (Killswitch)

In `store-plugins.json` beim Eintrag `"revoked": true` setzen → Schritt 3 + 4
von oben. Jede App entfernt das Plugin beim nächsten Store-Sync.
