# feature:export — Grupo ALICE

**Integrantes:** ALICE (completar con nombres y roles)

## Qué implementamos

Pantalla de exportación de viajes completados en dos formatos:

| Formato | Descripción |
|---------|-------------|
| **CSV** | Tabla con encabezado, lista para abrir en Excel / Google Sheets |
| **Texto plano** | Reporte legible con todos los campos del viaje |

### Flujo de la pantalla

1. **Pantalla de configuración**: muestra resumen (cantidad de viajes, distancia total, CO₂), selector de formato y botón "Generar".
2. **Vista previa**: muestra el contenido generado en una caja monoespaciada con scroll, botón de copiar y botón para volver.
3. El contenido se copia al portapapeles del dispositivo con un tap.

## Arquitectura

```
ExportScreen.kt     → UI en Jetpack Compose (3 estados: loading / config / preview)
ExportViewModel.kt  → Lógica de negocio con StateFlow, sin dependencias de Android
```

- `ExportViewModel` recibe `TripRepository` por constructor (inyectable / testeable).
- `buildCsv` y `buildPlainText` son funciones `internal` testeables directamente.

## Cómo correr y probar (en aislado)

```bash
./gradlew :feature:export:test
```

## Decisiones técnicas

- **Sin File I/O**: la regla prohíbe agregar dependencias; en su lugar se genera el texto en memoria y se copia al portapapeles, que es la forma más compatible sin permisos adicionales.
- **Offline-first**: la exportación funciona completamente sin red, usa solo datos locales del `TripRepository`.
- **Separación de estados**: `isLoading`, `exportedContent` y `error` son campos independientes en `ExportUiState` para evitar estados imposibles.

## Contratos

- Consume: `TripRepository.completedTrips()` de `core:domain`
- Expone: `ExportScreen(viewModel: ExportViewModel)`

## Limitaciones / pendientes

- No exporta a archivo `.csv` en el almacenamiento del dispositivo (requeriría `READ/WRITE_EXTERNAL_STORAGE` o `MediaStore`, fuera del alcance por la regla de no agregar dependencias).
- No exporta puntos GPS individuales, solo resumen por viaje.
