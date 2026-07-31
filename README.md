# 🗺️ Generador de Recorridos KML

Aplicación web de un solo archivo (`index.html`), 100% offline salvo el ruteo opcional, para que inspectores de calle del **Gobierno de la Ciudad de Buenos Aires (GCBA)** armen recorridos y los exporten a **KML** (Google Earth / My Maps).

El callejero oficial de CABA (calles, cuadras y comunas) viene **embebido en el propio HTML**, comprimido en base64, y se descomprime en el navegador — no requiere backend ni conexión para funcionar (salvo el mapa base y el ruteo por calles, que son opcionales).

---

## 📖 Descripción

La herramienta permite generar archivos KML compatibles con Google Earth y otras plataformas SIG, facilitando la planificación de recorridos y la organización territorial de inspecciones.

---

## 🚀 Funcionalidades

La app tiene **tres pestañas**:

### 📏 Por Cuadras
Búsqueda de una calle y partición del recorrido en bloques de N cuadras, con descarga individual o en ZIP.

### 📍 Por Entrecalles
- **Manual:** calle principal + dos entrecalles → traza y exporta solo el tramo entre esas dos esquinas, con corte exacto por altura de esquina (sin pasarse de cuadra).
- **Por lote (Excel/CSV):** columnas `calle principal`, `entrecalle 1`, `entrecalle 2`, `nombre de recorrido` → genera todos los tramos de una lista y los exporta en KML individuales o ZIP.

### 📄 Por Lista (Excel/CSV)
Geocodifica un listado de direcciones (dirección + inspector) contra el callejero embebido, arma un recorrido ordenado por inspector (vecino más cercano + 2-opt) y exporta a KML/ZIP. Puede trazar el recorrido "tipo GPS" por calles reales usando OSRM público, con fallback a línea recta si no hay conexión.

### Otras características
- Trazado sin líneas cruzadas: cada cuadra se dibuja como polilínea independiente (evita saltos entre cuadras no contiguas).
- Geocodificación offline tolerante a abreviaturas y orden de nombre/apellido, con desambiguación de calles homónimas por altura y comuna.
- Exportación de recorridos individuales o en lote (ZIP) desde cualquiera de las tres pestañas.

Ver [CHANGELOG.md](./CHANGELOG.md) para el detalle de cambios por versión.

---

## 🛠️ Tecnologías utilizadas

- HTML5 + JavaScript puro (sin build)
- [Leaflet](https://leafletjs.com/) — mapa interactivo
- [JSZip](https://stuk.github.io/jszip/) — exportación en lote (ZIP)
- [SheetJS/xlsx](https://sheetjs.com/) — lectura de Excel/CSV
- [OSRM](http://project-osrm.org/) público — ruteo por calles (opcional)
- KML / Google Earth

---

## 🌐 Aplicación en línea

Disponible en:

**https://derlismarce.github.io/Generador-Recorridos-KML/**

---

## 👨‍💻 Autor

**Derlis Marcelo Fernandez Rivas**
