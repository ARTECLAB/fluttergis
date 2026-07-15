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
| `heatmap.html` | 6 | Mapa de calor: los datos que brillan | 8 |
| `provider.html` | 7 | Arquitectura en 3 capas: la app que crece bien | 12 |
| `offline.html` | 8 | Offline Maps: cuando no hay señal | 8 |
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
flutter_map_heatmap: latest  # HeatMapLayer, WeightedLatLng
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
│   ├── colecta_db.dart       ← SQLite: INSERT, SELECT, UPDATE, pendientes
│   └── geoserver_api.dart    ← http GET/POST a GeoServer REST API
├── services/
│   ├── gps_service.dart      ← Stream GPS, permisos, Haversine, pause/resume
│   ├── wms_service.dart      ← URLs WMS, filtros CQL, opciones de capa
│   └── export_service.dart   ← GeoJSON, CSV, share_plus
├── screens/
│   ├── map_screen.dart       ← FlutterMap + GpsService + capas WMS + onTap captura punto + Marker por colecta guardada
│   ├── collect_screen.dart   ← Recibe LatLng punto por constructor + foto + ColectaDb
│   ├── heatmap_screen.dart   ← HeatMapLayer + ColectaDb
│   └── export_screen.dart    ← botones + ExportService
└── widgets/
    ├── layer_panel.dart      ← Toggle + Slider de capas WMS
    └── collect_form.dart     ← Formulario de colecta reutilizable
```

### Regla de oro de la arquitectura
Las flechas solo bajan: `UI → Services → Data`.  
`Data` nunca importa `Services`. `Services` nunca importa widgets de Flutter.  
Si un archivo de `data/` tiene `import 'package:flutter/material.dart'`, está mal.

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

### FMTC para offline maps
`FMTCStore('campo_laPaz')` con `CacheBehavior.cacheFirst`.  
Pre-descarga zoom 10-16 para La Paz (~15.000 tiles, ~45MB).  
`runApp(GeoCollectApp())` directo — sin `ProviderScope`.

### Error crítico de coordenadas
GeoJSON usa `[longitud, latitud]` — LatLng espera `(latitud, longitud)`.  
Siempre: `c[1]` = lat, `c[0]` = lon al parsear GeoJSON.  
Este error coloca los puntos en el océano Pacífico.

### TileLayer WMS — sin crossOrigin
**No usar** `crossOrigin: 'anonymous'` en `TileLayer` con WMS.  
Los tiles se cargan como `Image.network` (no XHR), CORS no aplica en apps móviles.

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
- `heatmap.html` — WeightedLatLng, normalización min-max, filtro BBox
- `provider.html` — arquitectura 3 capas, GpsService completo, sin Riverpod
- `offline.html` — FMTC, descarga región, modo avión GIS, sin ProviderScope
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