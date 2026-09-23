# Hoja de Ruta

Convierte la planilla de entregas de GDM en un pedido de depósito en PDF, y vuelve de ese PDF al Excel original sin perder datos.

Sitio estático — un solo `index.html`, sin build ni backend. Todo corre en el navegador: los archivos nunca salen de la máquina del usuario.

## Qué hace

Se le suelta un `.xlsx` o un `.pdf` y detecta solo cuál es. Una vez cargado, exporta a cualquiera de los dos formatos.

**PDF** — armado como orden de preparación para depósito:

- Franja co-branded (GDM / Avancargo) sobre la banda con el N° de transporte
- Un bloque por parada, agrupado por solicitante y población, con dirección y contacto
- Tabla de picking con casilla para tildar a medida que se prepara
- Subtotal por parada, total general y pie de firmas

**Excel** — con el logo de GDM, encabezados fijos, autofiltro, fila de total con fórmula viva y la tabla de contactos.

## Round-trip

Al generar el PDF, los datos de origen quedan embebidos después del marcador `%%EOF` en una línea `%%HDR1:<base64>`. Los lectores de PDF ignoran todo lo que sigue al `%%EOF`, así que el archivo es válido y a la vez lleva su propia fuente de verdad.

Por eso la vuelta PDF → Excel es exacta y no depende de extraer texto de la página. La contra: **un PDF que no haya salido de esta app no se puede convertir**. La app lo dice con un mensaje claro en vez de devolver algo incompleto.

## Esquema de la planilla

Las 17 columnas son fijas; varía la cantidad de filas.

| # | Columna | Tipo | Obligatoria |
|---|---|---|---|
| 1 | N° Transporte | Texto o número | sí |
| 2 | Entrega | Texto o número | sí |
| 3 | Solicitante | Texto o número | no |
| 4 | Nombre del solicitante | Texto | sí |
| 5 | Nombre Pagador | Texto | no |
| 6 | Descripción Material | Texto | sí |
| 7 | Descripción Calibre | Texto | no |
| 8 | Cantidad entrega | **Número** | sí |
| 9 | Banda | Texto | no |
| 10 | Nombre Centro | Texto | no |
| 11 | Provincia | Texto | no |
| 12 | Población | Texto | sí |
| 13 | Dirección | Texto | no |
| 14 | Lote | Texto | no |
| 15 | Creado por | Texto | no |
| 16 | Teléfono Contacto | Texto o número | no |
| 17 | Nombre de Contacto | Texto | no |

«Obligatoria» aplica a la celda: la columna siempre tiene que existir, pero solo esas no pueden quedar vacías.

Las entregas se cortan en la primera fila sin `N° Transporte`. Debajo puede ir una fila de total y una tabla de contactos (`Nombre del solicitante` · `Contacto` · `Telefono`), que se usa como respaldo para archivos anteriores a las columnas 16 y 17.

## Validación

Las columnas se buscan **por nombre**, nunca por posición, así que el orden no importa y una planilla ajena se rechaza en vez de mapearse mal en silencio. Los encabezados se comparan normalizados: tolera `°` vs `º`, acentos, mayúsculas, espacios dobles y dos puntos al final.

Se rechaza el archivo, nombrando el problema, cuando: no aparece la fila de encabezados, falta o se repite alguna de las 17 columnas, `Cantidad entrega` no es numérica, o una celda obligatoria está vacía.

El lector busca la fila de encabezados en vez de asumir que está en la fila 1, así que tolera que el archivo arranque con una banda de logo.

## Desarrollo

No hay dependencias ni paso de build. Se abre `index.html` en el navegador y listo.

Las librerías se cargan desde CDN con versión fijada:

| Librería | Uso |
|---|---|
| SheetJS 0.18.5 | leer `.xlsx` |
| ExcelJS 4.4.0 | escribir `.xlsx` (SheetJS no inserta imágenes) |
| jsPDF 2.5.1 + autotable 3.8.2 | generar el PDF |

## Deploy

Vercel, como sitio estático. Sin variables de entorno.
