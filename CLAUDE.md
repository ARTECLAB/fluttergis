# Flutter GIS — GeoCollect · CLAUDE.md
> Archivo específico para `carpetanueva/fluttergis/`.
> Lee también `carpetanueva/CLAUDE.md` para las reglas globales.

---

## El curso

**Nombre:** Flutter GIS — GeoCollect  
**Nivel:** Básico-intermedio  
**Duración:** 10 clases × 2 horas = 20 horas  
**Hosting:** `fluttergis.danielquisbert.com` (GitHub Pages)  
**Paleta:** teal `#006A6A` / `#4DD0CC`  
**App que construimos:** GeoCollect — colecta de datos de campo en Android

---

## Archivos del curso — orden exacto

### Slides (raíz del subdominio)

| Archivo | Clase | Título | Slides |
|---------|-------|--------|--------|
| `index.html` | — | Página principal del curso | — |
| `fundamentos.html` | — | Preparación para el curso | — |
| `widget.html` | 1 | El mapa que late: flutter_map desde cero | 13 |
| `stream.html` | 2 | GPS en tiempo real: el mapa que te sigue | 14 |
| `layer.html` | 3 | Capas WMS desde GeoServer: el servidor habla | 10 |
| `feature.html` | 4 | GeoJSON local: el mapa de provincias en tu app | 10 |
| `collect.html` | 5 | Colecta de campo: toca el mapa, captura el punto | 20 |
| `heatmap.html` | 6 | Mapa de calor: los datos que brillan | 9 |
| `provider.html` | 7 | Arquitectura en 3 capas: la app que crece bien | 12 |
| `offline.html` | 8 | Offline Maps: cuando no hay señal | 7 |
| `export.html` | 9 | Exportar y compartir: los datos que viajan | 7 |
| `release.html` | 10 | GeoCollect completa: proyecto final y APK | 8 |

### Quizzes (subcarpeta test/)

Mismo nombre de archivo que el slide correspondiente.  
Cada quiz: 12 preguntas · 2 intentos · retroalimentación inmediata.

---

## Stack tecnológico — versiones exactas

```yaml
flutter_map: ^8.0.0          # mapa base, TileLayer, MarkerLayer, PolylineLayer
latlong2: ^0.9.0             # coordenadas LatLng
geolocator: ^13.0.0          # GPS, permisos, getPositionStream
permission_handler: ^11.0.0  # permisos en runtime
sqflite: ^2.0.0              # SQLite local
path_provider: ^2.0.0        # rutas del sistema de archivos
http: ^1.0.0                 # peticiones HTTP (WMS, REST API GeoServer)
connectivity_plus: ^6.0.0    # estado de red, sync automática
flutter_map_heatmap_fix: ^1.0.0   # fork de flutter_map_heatmap compatible con flutter_map ^8.0.0 — HeatMapLayer, WeightedLatLng
flutter_map_tile_caching: latest  # FMTC, cache y descarga offline
image_picker: latest         # fotos de campo
share_plus: ^10.0.0          # exportar y compartir archivos
flutter_launcher_icons: ^0.13.0  # iconos de la app
```

---

## Arquitectura de la app — 3 capas

```
lib/
├── main.dart                 ← solo runApp(GeoCollectApp()). Sin ProviderScope.
├── models/
│   └── colecta.dart          ← Clase Colecta: campos, toMap(), fromMap()
├── data/
│   ├── colecta_db.dart       ← ColectaDB — SQLite: INSERT, SELECT, UPDATE, pendientes
│   └── geoserver_api.dart    ← http GET/POST a GeoServer REST API
├── services/
│   ├── gps_service.dart      ← Stream GPS, permisos, Haversine, pause/resume
│   ├── colecta_service.dart  ← ColectaService: guardar local + sincronizar (connectivity_plus)
│   ├── wms_service.dart      ← URLs WMS, filtros CQL, opciones de capa
│   └── export_service.dart   ← GeoJSON, CSV, share_plus
├── screens/
│   ├── mapa_screen.dart      ← FlutterMap + GpsService + capas WMS + onTap captura punto + Marker por colecta guardada
│   ├── collect_screen.dart   ← Recibe LatLng punto por constructor + foto + ColectaService
│   ├── heatmap_screen.dart   ← HeatMapLayer + ColectaDB
│   └── export_screen.dart    ← botones + ExportService
└── widgets/
    ├── layer_panel.dart      ← Toggle + Slider de capas WMS
    └── collect_form.dart     ← Formulario de colecta reutilizable
```

### Regla de oro de la arquitectura
Las flechas solo bajan: `UI → Services → Data`.  
`Data` nunca importa `Services`. `Services` nunca importa widgets de Flutter.  
Si un archivo de `data/` tiene `import 'package:flutter/material.dart'`, está mal.

### Nombres canónicos (app y slides deben coincidir — verificado 2026-07-15)
- `MapaScreen` / `_MapaState` / `mapa_screen.dart` — **nunca** MapScreen ni map_screen.dart
- `ColectaDB` (con B mayúscula) — **nunca** ColectaDb
- `ColectaService` (lib/services/colecta_service.dart): `guardarColecta()`, `sincronizar()`, `escucharConectividad()`, `_publicarEnGeoServer()`
- `GpsService`: `iniciar()`, `pausar()`, `reanudar()`, `detener()`, getter `posicion`
- La app de referencia vive en `carpetanueva/geocollect_app/`

### Sin Riverpod
El curso usa arquitectura de 3 capas con `setState()` y `StreamBuilder`.  
**No usar Riverpod**, Provider, Bloc ni ningún gestor de estado externo.  
Riverpod puede mencionarse como "existe para proyectos más avanzados" pero no se usa.

---

## Decisiones técnicas importantes

### GeoJSON local en assets/ — NO WFS desde Flutter
La Clase 4 usa GeoJSON en `assets/geo/` leído con `rootBundle.loadString()`.  
**No usar WFS** desde Flutter porque:
- Requiere el servidor encendido durante la clase
- Problemas de CORS en entornos de prueba
- La misma experiencia pedagógica con assets locales
- WFS puede mencionarse como concepto GIS pero no se implementa en el código

### GPS en background
En el curso usamos **primer plano** con `WidgetsBindingObserver`:
- `paused` → `_gps.pausar()` (ahorra batería)
- `resumed` → `_gps.reanudar()`

El rastreo real en background requiere `ACCESS_BACKGROUND_LOCATION` + Foreground Service.  
Se explica en el slide "¿El GPS funciona en background?" de `stream.html` pero no se implementa.

### distanceFilter: 5
Siempre usar `distanceFilter: 5` en `LocationSettings` — emite solo al moverse 5 metros.  
Sin esto el stream emite ~10 veces por segundo y drena la batería.

### FMTC para offline maps — API v10 (corregido 2026-07-20)
`flutter_map_tile_caching` v10 usa el backend ObjectBox: inicializar con `await FMTCObjectBoxBackend().initialise()`, **no** `FlutterMapTileCaching.initialise()` (API de v6/v7, ya no existe).  
El tile provider se construye con `FMTCTileProvider(stores: {'campo_laPaz': BrowseStoreStrategy.readUpdateCreate}, loadingStrategy: BrowseLoadingStrategy.cacheFirst)` — `FMTCStore(...).getTileProvider()` sigue existiendo pero está deprecado, y la clase `FMTCTileProviderSettings` ya no existe.  
`CacheBehavior` es un alias deprecado de `BrowseLoadingStrategy` — usar el nombre nuevo en código nuevo.  
`store.download.check(region)` devuelve `Future<int>` (solo cuenta de tiles) — v10 ya no entrega el tamaño en MB directamente, hay que estimarlo multiplicando por un promedio por tile (~3KB para tiles OSM).  
`store.download.startForeground(region: region)` es sincrónico (no lleva `await`) y devuelve un record `({tileEvents, downloadProgress})` — escuchar con `download.downloadProgress.listen(...)`.  
Pre-descarga zoom 10-16 para La Paz (~15.000 tiles, ~45MB).  
`runApp(GeoCollectApp())` directo — sin `ProviderScope`.

### Error crítico de coordenadas
GeoJSON usa `[longitud, latitud]` — LatLng espera `(latitud, longitud)`.  
Siempre: `c[1]` = lat, `c[0]` = lon al parsear GeoJSON.  
Este error coloca los puntos en el océano Pacífico.

### TileLayer WMS — sin crossOrigin
**No usar** `crossOrigin: 'anonymous'` en `TileLayer` con WMS.  
Los tiles se cargan como `Image.network` (no XHR), CORS no aplica en apps móviles.

### Heatmap — flutter_map_heatmap_fix, no flutter_map_heatmap (corregido 2026-07-20)
El paquete original `flutter_map_heatmap` (última versión pub.dev) exige `flutter_map >=7.0.0 <8.0.0` — incompatible con el `flutter_map: ^8.0.0` del curso, pub falla al resolver dependencias.  
Usar **`flutter_map_heatmap_fix`** (fork actualizado para `flutter_map ^8.0.0`).  
`WeightedLatLng` tiene constructor **posicional**: `WeightedLatLng(latLng, intensity)` — nunca `intensity:` como parámetro nombrado, no existe.  
`HeatMapLayer` se pinta como un `TileLayer` interno cacheado por URL — `setState()` solo **no** repinta el heatmap. Hace falta un `StreamController<void>` pasado como `reset:` y llamar `_reset.add(null)` cada vez que cambian los datos o `heatMapOptions`; cerrarlo en `dispose()`.

---

## Datos del curso

| Dataset | Fuente | Clases |
|---------|--------|--------|
| Departamentos de Bolivia | GeoBolivia / instructor | 1, 3, 4, 5 |
| Provincias de Bolivia (GeoJSON) | GeoBolivia / instructor | 4 |
| Capas WMS/WFS del curso GeoServer | VM del instructor | 3, 4 |
| 500 puntos simulados La Paz | Generados en Dart | 6 |

Archivo GeoJSON de provincias: `assets/geo/provincias_bolivia.geojson` (112 provincias)  
Declarar en `pubspec.yaml` bajo `flutter: assets:`.

---

## Estado actual de los archivos

### ✅ Completados y revisados
- `index.html` — CSS autónomo, sin links a slides, dark mode
- `fundamentos.html` — tono inclusivo, sin "debes saber"
- `widget.html` — estructura mínima FlutterMap, marcador completo, dark mode automático
- `stream.html` — GPS completo: pubspec, permisos, GpsService, StreamBuilder, Haversine, slide de background
- `layer.html` — WMS, CQL_FILTER, sin WFS, sin crossOrigin
- `feature.html` — GeoJSON local assets/ (provincias), compute(), PolygonLayer simple (sin tap ni cache — se ven en clases posteriores)
- `collect.html` — reescrita (2026-07-15): el punto ya no se pide al GPS al abrir el formulario — se captura con onTap en el mapa (MapaScreen, Clases 1-2) y viaja a CollectScreen como parámetro del constructor (LatLng punto). pubspec.yaml + permisos (CAMERA, ACCESS_NETWORK_STATE), sqflite, toMap/fromMap, offline-first, CollectScreen en 3 partes (build, foto, guardar). Cierra el ciclo: ColectaDB.listar() + _cargarColectas() muestran las colectas guardadas como Marker en el mapa al volver del formulario, con GestureDetector para ver el detalle al tocar (color según sincronizado) — ya no es un formulario aislado, es una app de recolección real
- `heatmap.html` — corregida (2026-07-20): paquete `flutter_map_heatmap_fix` en vez de `flutter_map_heatmap` (incompatible con flutter_map ^8.0.0), constructor posicional de WeightedLatLng, patrón StreamController `reset:` para que el Slider realmente repinte el heatmap. Se agregó una slide conceptual "Que es normalizar?" con ejemplo numérico antes de la slide de código — antes el min-max saltaba directo al código sin explicar el concepto. 9 slides, nav-dots corregidos de 12 a 9
- `provider.html` — arquitectura 3 capas, GpsService completo, sin Riverpod
- `offline.html` — corregida (2026-07-20): faltaba un `</div>` de cierre en la 2.ª slide que anidaba TODAS las slides siguientes y el navbar dentro de ella — al no ser `.active` quedaban ocultas, por eso "no se veía nada" al navegar. La API de FMTC estaba desactualizada a v10: `FlutterMapTileCaching.initialise()` → `FMTCObjectBoxBackend().initialise()`, `getTileProvider(FMTCTileProviderSettings(...))` (deprecado) → `FMTCTileProvider(stores: {...}, loadingStrategy: ...)`, y `download.check()` ya no devuelve tamaño en MB (solo cuenta tiles — hay que estimar el tamaño multiplicando por un promedio por tile). Se eliminó la slide "Conectividad automatica" (connectivity_plus) — ya se enseña en `collect.html` Clase 5 (ColectaService.escucharConectividad()) y era redundante aquí; en su lugar se agregó un callout sobre `download.pause()`/`resume()`. 7 slides, nav-dots corregidos de 12 a 7
- `export.html` — GeoJSON Dart puro, CSV, share_plus, REST API GeoServer
- `release.html` — split-per-abi, obfuscación, App Bundle
- `test/*.html` — 10 quizzes, 120 preguntas, sin WFS, sin Riverpod, sin Claude

### ✅ Resuelto (2026-07-09) — problema de zoom en pantallas grandes
`slides.css` ya usa `justify-content: flex-start` en `.slide.active` y `font-size: clamp(10px, 1.1vw, 13px)` en `.code-block`, en fluttergis y en geoserver (son dos copias separadas del archivo, no compartidas — hay que aplicar cualquier cambio de CSS en ambas).

### ⚠️ Pendiente — comentarios en código
Todos los code-block deben tener comentarios explicativos en cada línea importante.  
`stream.html` fue parcialmente corregido pero los demás slides pueden tener ejemplos sin comentar.

---

## Convenciones del curso en los slides

### slide-cover de apertura
```html
<div class="cover-tag">
  <span class="cover-dot"></span>
  Módulo X · Clase Y · <span class="auto-year"></span>
</div>
<h1 class="cover-title">Título de la clase:<br><em>subtítulo en teal</em></h1>
<p class="cover-sub">descripción breve del contenido</p>
<div class="cover-pills">
  <span class="cover-pill">⏱ 2 horas</span>
  <span class="cover-pill">🏷️ Tema</span>
  <span class="cover-pill">👨‍💻 Daniel Quisbert</span>
</div>
```

### instructor-strip (en slide de objetivos)
```html
<div class="instructor-strip">
  <div class="is-avatar">DQ</div>
  <div>
    <div class="is-name">Daniel Quisbert</div>
    <div class="is-role">ARTECLAB · fluttergis.danielquisbert.com</div>
  </div>
</div>
```

### quiz-cta (en slide de cierre)
```html
<div class="quiz-cta">
  <div class="quiz-cta-text">
    <div class="quiz-cta-title">Cuestionario — Clase N: Tema</div>
    <div class="quiz-cta-sub">12 preguntas · 2 intentos</div>
  </div>
  <a href="test/ARCHIVO.html" target="_blank">Ir al cuestionario →</a>
</div>
```

---

## Módulos del curso

| Módulo | Clases | Tema |
|--------|--------|------|
| 1 — El mapa | 1, 2 | flutter_map y GPS |
| 2 — Los datos | 3, 4 | WMS y GeoJSON |
| 3 — La colecta | 5, 6 | SQLite y heatmap |
| 4 — La arquitectura | 7, 8 | 3 capas y offline |
| 5 — La entrega | 9, 10 | Export y APK |