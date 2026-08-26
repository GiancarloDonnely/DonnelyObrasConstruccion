# Donnely · Obras en Construcción

Dashboard financiero de una sola página (HTML + CSS + JS, sin backend ni build step) que muestra
el consolidado de compras del centro de costo **"Obras en Construcción"** de Donnely Ltda.:
cuánto se ha gastado en la obra mes a mes, desglosado por ítem (Honorarios, Materiales,
Maquinaria, Combustible, Arriendos), más un ranking de proveedores.

## Datos

Los datos se leen en vivo desde la hoja **"L-Compras"** (libro de compras) de la planilla de
Donnely, publicada como CSV. Si la lectura en vivo falla, el tablero usa un snapshot de respaldo
embebido en el código (Enero–Agosto 2026).

## Uso

Abrir `index.html` directamente en el navegador — no requiere servidor ni instalación.

## Ver también

Este proyecto es independiente del dashboard del Estado de Resultados (repo `DonnelyEERR`), pero
lee de la misma planilla de origen (pestaña distinta).
