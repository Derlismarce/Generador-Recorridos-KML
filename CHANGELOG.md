# Changelog

Todos los cambios notables de este proyecto se documentan en este archivo.

## [2026-08-04]

### Agregado
- **Pestaña 🔷 Polígono** — nueva pestaña para dibujar zonas/áreas de inspección y exportarlas a KML/GeoJSON.
  - Vértices por dirección reutilizando `geocodeAddr` ("calle y altura" o "calle Y calle") o por clic directo en el mapa (`polyAddVertex`, `polyAddByAddress`).
  - Edición visual sobre el mapa: arrastrar vértices para moverlos, insertar uno nuevo arrastrando un punto traslúcido del borde (`polyRebuildMidMarkersOnly`), borrar con un clic sobre el vértice, clic derecho para finalizar la zona.
  - Resumen en vivo de cantidad de vértices, área y perímetro (`polyArea`, `polyPerimeter`, proyección local en `polyProject`).
  - Flujo Finalizar/Editar/Limpiar (`polyFinalizar`, `polyEditar`, `polyClear`) y exportación a KML (`buildPolygonKml`/`downloadPolyKml`) y GeoJSON (`downloadPolyGeojson`).

- **🖼️ Generar polígono desde imagen (experimental — no queda funcionando como se esperaba, pendiente de revisión):**
  - Detección automática del contorno de un polígono dibujado en una captura subida por el usuario: flood fill por color a partir de un clic dentro del relleno (`piDetectFromSeed`), trazado radial del borde desde el centroide (`piRadialContour`) y simplificación a vértices reales con Douglas-Peucker adaptado a contorno cerrado (`piDouglasPeuckerClosed`).
  - Georreferenciación por 2 puntos de referencia (imagen ↔ dirección real o clic en el mapa real) resueltos con una transformación de similitud (rotación + escala + traslación) para convertir píxeles de la imagen en lat/lon (`piGenerateOnMap`).
  - Sugerencia automática de puntos de referencia por OCR (Tesseract.js): lee el texto visible en la imagen, lo matchea contra el callejero embebido y, si encuentra dos calles cercanas que efectivamente se cruzan en la realidad, propone el cruce como punto de referencia para confirmar con un clic (`piRunOcr`, `piBuildOcrSuggestions`, `piApplyOcrSuggestion`).
  - **Estado:** el pipeline corre sin errores y fue validado con imágenes sintéticas, pero en uso real el resultado no salió como se esperaba — precisión de la detección/OCR a mejorar antes de confiar en esta función para trabajo real.

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
