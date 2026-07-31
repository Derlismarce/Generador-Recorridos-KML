# Changelog

Todos los cambios notables de este proyecto se documentan en este archivo.

## [2026-07-31]

### Agregado
- **Pestaña 📍 Por Entrecalles**
  - Modo manual: calle principal + dos entrecalles → traza y exporta solo el tramo exacto entre esas dos esquinas.
  - Corte exacto por esquina: `nearestPointOnMain` ubica la intersección real sobre la calle principal y `cornerAltura` promedia las alturas de borde de las cuadras que comparten ese vértice, evitando el bug de "pasarse una cuadra".
  - Modo por lote (Excel/CSV): columnas `calle principal`, `entrecalle 1`, `entrecalle 2`, `nombre de recorrido` → genera todos los tramos de la lista y los exporta en KML individuales o ZIP (`buildEntreKmlNamed`, `buildEntreLote`, `parseEntreFile`, `drawEntreLote`, `downloadEntreKml`, `downloadEntreZip`).
  - Desambiguación de entrecalles homónimas mediante nube de puntos de todas las candidatas (`crossCloud`, `crossPointsFor`), seleccionando la que efectivamente cruza la calle principal.

- **Pestaña 📄 Por Lista**
  - Geocodificación offline de un listado de direcciones (Excel/CSV) contra el callejero embebido: separación de altura y nombre, detección de intersecciones, scoring tolerante a abreviaturas y orden de nombre/apellido (`geocodeAddr`, `scoreStreet`, `tokMatch`, `findStreetByName`, `addrToks`, `normAddr`).
  - Interpolación del punto exacto sobre la cuadra según la altura (`interpCuadra`).
  - Armado de recorrido por inspector con ordenamiento vecino más cercano + 2-opt (`orderStops`).
  - Trazado "tipo GPS" por calles reales vía OSRM público, con fallback automático a línea recta si falla o no hay conexión (`osrmRoute`).
  - Exportación de recorridos por inspector en KML individuales o ZIP (`buildListRoute`, `buildKmlForRoutes`, `downloadListKml`, `downloadListZip`), con reporte de resultados (`renderListReport`).

- Selector de columnas para los archivos Excel/CSV importados en ambas pestañas nuevas (`populateColSelects`, `populateEntreCols`).

### Corregido
- **Trazado sin líneas cruzadas** en las tres pestañas: como las cuadras son frentes/veredas no encadenados punta a punta, dibujarlas como una sola polilínea unía cuadras no contiguas y generaba saltos de hasta ~1,7 km entre calles distintas. Ahora `drawCuadras` dibuja cada cuadra como polilínea independiente, y el KML exportado usa `MultiGeometry` con una `LineString` por cuadra.

### Validado
- Geocodificación offline: 12/12 direcciones reales, residuo medio ~8 m contra coordenadas oficiales USIG (máximo 14 m).
- Corte por entrecalles: caso Córdoba entre San Martín (500) y Maipú (700) → alturas 502–700, 2 veredas, resultado independiente del orden de las entrecalles ingresadas.

## Versiones anteriores

- Versión inicial: pestaña única **📏 Por Cuadras** — búsqueda de calle, partición en bloques de N cuadras, descarga individual o ZIP.
