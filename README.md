# 🗺️ Generador de Recorridos KML

Aplicación web de un solo archivo (`index.html`), 100% offline salvo el ruteo opcional, para que inspectores de calle del **Gobierno de la Ciudad de Buenos Aires (GCBA)** armen recorridos y los exporten a **KML** (Google Earth / My Maps).

El callejero oficial de CABA (calles, cuadras y comunas) viene **embebido en el propio HTML**, comprimido en base64, y se descomprime en el navegador — no requiere backend ni conexión para funcionar (salvo el mapa base y el ruteo por calles, que son opcionales).

---

## 📖 Descripción

La herramienta permite generar archivos KML compatibles con Google Earth y otras plataformas SIG, facilitando la planificación de recorridos y la organización territorial de inspecciones.

---

## 🚀 Funcionalidades

La app tiene **cuatro pestañas**, y cada una tiene su propio botón **↩️ Deshacer** para volver atrás si una acción (finalizar, limpiar, unir atributos, etc.) no salió como se esperaba.

### 📏 Por Cuadras
- Búsqueda de una calle y partición del recorrido en bloques de N **cuadras consecutivas** (ordenadas por altura antes de cortar, para que cada KML sea un tramo real sin saltos), con descarga individual o en ZIP.
- **Por lote (Excel/CSV):** subís un archivo con una columna "calle y altura" (y comuna opcional) → geocodifica cada dirección y arma un recorrido en línea que **cambia de color cada vez que cambia la calle**. Descarga individual, ZIP, o un KML unificado — con toda la info de cada fila del Excel en la descripción de cada punto.

### 📍 Por Entrecalles
- **Manual:** calle principal + dos entrecalles (o directamente dos **alturas**, ej. 102 y 200) → traza y acumula el tramo exacto entre esos dos puntos, con corte exacto por altura de esquina (sin pasarse de cuadra). Los tramos se van sumando a una lista descargable individual, en ZIP, o como KML unificado.
- **Por lote (Excel/CSV):** columnas `calle principal`, `entrecalle 1`, `entrecalle 2` (nombre o altura), `nombre de recorrido` → genera todos los tramos de la lista, con toda la info de cada fila incluida en el KML.

### 🔷 Polígono
Dibuja zonas/áreas de inspección y las acumula en una lista descargable.
- Vértices por dirección (`geocodeAddr`: "calle y altura" o "calle Y calle") o por clic directo en el mapa; arrastrar para mover o insertar, clic sobre un vértice para borrarlo.
- Finalizar Zona acumula cada polígono en una lista (editable, individual/ZIP/KML unificado), con las calles y comunas que caen dentro de cada zona detectadas automáticamente e incluidas en la descripción del KML.
- Subida de polígonos KML ya existentes para verlos y seguir editándolos.
- **✂️ Separar KML por capa:** subís un KML con varias capas (folders) y lo divide en un archivo por capa, listo para importar en Google My Maps respetando sus límites (10 capas, 2.000 features y 5 MB por capa).

### 🌳 Espacios Verdes
Callejero propio de **781 plazas, parques, plazoletas y jardines de CABA**, embebido offline (reproyectado desde el GeoPackage oficial de Espacios Verdes del GCBA).
- Buscador por nombre con autocompletar; cada resultado se acumula en una lista (individual/ZIP/KML unificado multi-parte, porque un mismo espacio verde puede tener varios fragmentos de polígono).
- **Por lote (Excel/CSV):** buscás una lista de nombres de una sola vez.
- **Generador de polígonos a mano**, separado de los resultados de búsqueda, con nombre editable en cualquier momento.
- **🔗 Unir atributos por valor de campo** (como en QGIS): subís un Excel/CSV o un `.kml` con una columna que matchee por nombre, y le pega todos sus atributos a cada espacio verde coincidente — quedan en la descripción del KML exportado.

### Otras características
- Trazado sin líneas cruzadas: cada cuadra se dibuja como polilínea independiente (evita saltos entre cuadras no contiguas).
- Geocodificación offline tolerante a abreviaturas y orden de nombre/apellido, con desambiguación de calles homónimas por altura y comuna.
- Los KML "unificados" (varios elementos en un solo archivo) se generan **sin carpetas anidadas**, porque Google My Maps convierte cada `Folder` en una capa separada — así entran siempre como una sola capa.
- Exportación individual, en lote (ZIP) o unificada desde cualquiera de las cuatro pestañas.

Ver [CHANGELOG.md](./CHANGELOG.md) para el detalle de cambios por versión.

---

## 🛠️ Tecnologías utilizadas

- HTML5 + JavaScript puro (sin build)
- [Leaflet](https://leafletjs.com/) — mapa interactivo
- [JSZip](https://stuk.github.io/jszip/) — exportación en lote (ZIP)
- [SheetJS/xlsx](https://sheetjs.com/) — lectura de Excel/CSV
- KML / Google Earth

---

## 🌐 Aplicación en línea

Disponible en:

**https://derlismarce.github.io/Generador-Recorridos-KML/**

---

## 👨‍💻 Autor

**Derlis Marcelo Fernandez Rivas**
