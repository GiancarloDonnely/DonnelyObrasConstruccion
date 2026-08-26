# CLAUDE.md

Guía para Claude Code al trabajar en esta carpeta. **Este es un proyecto distinto al de
`DonnelyEERR-main`** (la carpeta hermana) — no mezclar convenciones ni datos entre ambos. Vive en
su propio repo de GitHub, independiente del repo `DonnelyEERR`.

## Qué es este proyecto

Dashboard HTML de una sola página (mismo patrón que `DonnelyEERR-main/index.html`: HTML+CSS+JS
inline, sin build step, Chart.js vía CDN) que muestra el **consolidado de compras de Obras en
Construcción** de Donnely Ltda.: cuánto se ha gastado en la obra, mes a mes y por ítem, desde
enero hasta la fecha.

## Planilla fuente

Misma spreadsheet que usa `DonnelyEERR-main` (documento con múltiples pestañas publicadas), pero
la pestaña relevante acá es **L-Compras** (el libro de compras), no la de EERR:

`https://docs.google.com/spreadsheets/d/e/2PACX-1vT-fXOCkPmK3JhdTqVKEzCu3pcVSIDqE4bMHFWc_SeKPU1Fig_Ng-Bxb-WicqutE4N-2EISVrMa9aT0/pub?gid=1265752540&single=true&output=csv`

## Estructura real de "L-Compras" (confirmada leyendo el CSV en vivo)

- Fila 1: título ("LIBRO DE COMPRAS — DONNELY LTDA. 2026"), fila decorativa, no es el encabezado.
- Fila 2: encabezado real, con estas columnas (nombres exactos, usados por `index.html` vía
  `header.indexOf(...)` — no por posición fija, así que tolera columnas reordenadas):
  `MES, Tipo Compra, Tipo Documento, RUT, PROVEEDOR, Folio, Fecha Docto, Condicion, Vencimiento,
  Monto Exento, Monto Neto, Monto IVA Recuperable, Monto Total, Detalle, Prov-N/Impor/Fabric,
  Centro de Costo, Centralizado, Transferencia, Fecha, Egreso, Cruce SII, NC_FLAG`.
- Cada fila es un documento de compra (factura/boleta). `MES` es un número 1-12 ya cargado a mano
  en la planilla — el dashboard agrupa por esta columna, **no** parsea `Fecha Docto` (esa columna
  trae formatos de fecha inconsistentes fila a fila, D/M/Y y M/D/Y mezclados).
- La columna que identifica si la compra es de la obra es **`Centro de Costo`**. El valor
  esperado es "Obras en Construcción", pero en la práctica aparece con variantes de
  mayúsculas/acentos (`OBRAS EN CONSTRUCCION`, `Obras en construcción`) — el filtro normaliza
  (minúsculas, sin tildes) antes de comparar.
- La columna **`Detalle`** es la especificación del gasto dentro de la obra (lo que el usuario
  describe como "si es arriendo, honorarios, etc."). En los datos reales (al 2026-08-25) aparecen
  como texto libre con inconsistencias de mayúsculas/espacios/typos: `HONORARIOS - PROYECTO`,
  `MATERIALES - PROYECTO`, `MAQUINARIA - PROYECTO` (a veces con el typo "PROYECYO"),
  `COMBUSTIBLE - PROYECTO` (a veces "COMBUSTUBLE"), etc. `canonItem()` en `index.html` los agrupa
  por palabra clave (`honorario`, `material`, `maquinaria`, `combust`, `arriendo` → si no calza
  con ninguna, cae en "Otros"). Por ahora ningún registro de Obras en Construcción usa
  literalmente la palabra "arriendo" en `Detalle` (los arriendos de maquinaria que sí existen
  quedan tageados como `MAQUINARIA - PROYECTO`), pero la categoría "Arriendos" ya está lista para
  cuando aparezca.
- El monto que usa el dashboard es **`Monto Neto`** (sin IVA), igual convención que
  `DonnelyEERR-main` ("moneda CLP, montos netos") — el IVA de estas compras es crédito fiscal
  recuperable, no es costo real de la obra.

## Cómo fluyen los datos

1. `index.html` hace `fetch(SHEET_CSV_URL, {cache:'no-store'})` sobre la URL de `L-Compras` de
   arriba.
2. `parseCompras()` ubica la fila de encabezado buscando una fila que contenga `MES`, `Centro de
   Costo` y `Detalle` (tolera que se inserten filas arriba), filtra por Centro de Costo = Obras en
   Construcción, agrupa por mes (columna `MES`) y por ítem canonizado (columna `Detalle`).
3. Si el fetch falla, se usan los datos de respaldo embebidos en el script (`D`, `MESES`,
   `TOTAL_DOCS`, `PROVEEDORES`) — snapshot real tomado el 2026-08-25 (Enero-Agosto 2026), no datos
   inventados.
4. El tablero se extiende solo al agregar meses nuevos al Sheet (detecta el mes máximo presente
   en las filas de Obras en Construcción), sin tocar el código.

## Convenciones de esta carpeta

- Un solo archivo HTML (HTML+CSS+JS inline), sin build step ni package manager — se abre
  directamente en el navegador.
- Mismo esquema de color que `DonnelyEERR-main/index.html` (fondo oscuro, mismas variables CSS
  `--teal/--gold/--green/--red`) para consistencia de marca, pero **sin compartir código ni
  archivos** entre ambos proyectos — son repos separados.
- Librería externa: Chart.js vía CDN.
- Idioma: español. Moneda: CLP (montos netos).
- Si cambian los nombres exactos de columna en "L-Compras", o el texto de "Centro de Costo",
  actualizar `findHeaderRow()`/el filtro en `index.html` — hoy dependen de esos nombres literales.
- Repo de GitHub independiente (no es el mismo remoto que `DonnelyEERR`).
