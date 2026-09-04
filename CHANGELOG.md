# Changelog

Todos los cambios notables de este proyecto se documentan en este archivo.

## [2026-09-04]

Actualización grande: se retira la pestaña 📄 Por Lista y la generación de polígono desde imagen/OCR (quedó experimental y sin terminar), y se suman una pestaña nueva (Espacios Verdes), mejoras de acumulación en Por Cuadras/Por Entrecalles/Polígono, una herramienta de unión de atributos estilo QGIS, deshacer por pestaña, y un separador de KML por capa.

### Agregado
- **Pestaña 🌳 Espacios Verdes** — callejero propio de 781 plazas/parques/plazoletas/jardines de CABA, embebido offline (reproyectado desde Gauss-Krüger/Campo Inchauspe a lat/lon y simplificado con tolerancia de 1 m a partir del GeoPackage oficial de Espacios Verdes del GCBA).
  - Buscador por nombre con autocompletar (`evDoSearch`, `evAddZona`), resultados acumulables (individual/ZIP/KML unificado multi-parte vía `MultiGeometry`, ya que un mismo espacio puede tener varios fragmentos de polígono).
  - Por lote (Excel/CSV) buscando una lista de nombres de una sola vez (`buildEvLote`, `findEspacioByName`).
  - Generador de polígonos a mano independiente de los resultados de búsqueda, con nombre editable en cualquier momento (`evPolyFinalizar`, `evRenameDrawnZona`, `evEditDrawnZona`).
  - **Unir atributos por valor de campo** (como la herramienta homónima de QGIS): matchea por nombre contra un Excel/CSV o un `.kml` (leyendo `ExtendedData`) y pega todos los atributos de la fila/Placemark coincidente a cada elemento (`runAttributeJoin`, `parseJoinFile`).

- **Pestaña 📏 Por Cuadras**
  - Corrección: las cuadras se ordenan por altura antes de cortarlas en bloques de N, para que cada KML sea siempre un tramo **consecutivo** (antes salían mezcladas porque el callejero embebido no viene ordenado).
  - Por lote (Excel/CSV): sube una columna "calle y altura" y arma un recorrido en línea que cambia de color cada vez que cambia la calle, con toda la info de cada fila incluida en la descripción de cada punto del KML (`buildCuadrasLista`, `buildCuadrasTramoKml`).

- **Pestaña 📍 Por Entrecalles**
  - Las entrecalles ahora también aceptan una **altura directa** en vez de un nombre (ej. entrecalle 1: 102, entrecalle 2: 200), tanto en el modo manual como en el lote — sin afectar el flujo de dos entrecalles con nombre, que sigue igual.
  - Los tramos trazados a mano se **acumulan** en una lista (antes se reemplazaban) con descarga individual, ZIP o KML unificado.
  - Toda la info del Excel/CSV del modo por lote queda en la descripción del KML de cada tramo.

- **Pestaña 🔷 Polígono**
  - Las zonas finalizadas se **acumulan** en una lista (editable, renombrable) en vez de reemplazarse; descarga individual, ZIP o KML unificado.
  - Cada zona detecta automáticamente qué calles y comunas del callejero embebido caen dentro del polígono, y las incluye en la descripción del KML (`findStreetsAndComunasInZona`).
  - Subida de polígonos KML existentes para verlos y seguir editándolos (`handlePolyKmlFiles`).
  - **✂️ Separar KML por capa:** sube un KML con varias capas/folders y lo divide en un archivo por capa (con vista previa en el mapa), respetando los límites de Google My Maps (10 capas, 2.000 features y 5 MB por archivo).

- **↩️ Deshacer por pestaña**: antes de cada acción importante (generar, finalizar, limpiar, unir atributos, etc.) se guarda una foto del estado de esa pestaña; el botón "Deshacer último cambio" la restaura.

### Corregido
- Los KML "unificados" (varios tramos/zonas en un solo archivo) ya no usan `<Folder>` para agrupar: Google My Maps convierte cada `Folder` de primer nivel (incluso anidada) en una capa separada al importar, así que ahora todo va como `Placemark` plano dentro de un único `Document`, para que entre siempre como una sola capa.

### Quitado
- Pestaña 📄 Por Lista (geocodificación de listados con ruteo OSRM) — reemplazada en la práctica por las opciones "Por lote" de Por Cuadras y Por Entrecalles.
- Generación de polígono desde imagen (flood fill + OCR con Tesseract.js) — había quedado experimental y sin resultados confiables; se retira hasta poder revisarla en profundidad.

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
