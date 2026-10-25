# ARCA-gantt

Visor Gantt clásico del proyecto ARCA, publicado en GitHub Pages.

## Sitios

- Visor principal: https://jaredjomar.github.io/ARCA-gantt/
  Carga los datos en vivo desde el repo privado JaredJomar/ARCA. Pide un
  token de lectura de GitHub, que queda guardado en el navegador
  (localStorage) y no se envía a ningún otro sitio.
- Vista compartida: https://jaredjomar.github.io/ARCA-gantt/shared.html
  No pide token ni acceso al repo: los datos van incrustados en la página.
  Se regenera automáticamente cada mañana y solo se publica una versión
  nueva si los datos cambiaron.

## Cómo se genera el visor

No hay build ni framework. Todo el visor es un solo archivo, `index.html`,
que se publica directo desde la rama `main` con GitHub Pages.

- La gráfica la dibuja Frappe Gantt 0.6.1, cargado por CDN.
- JavaScript vanilla consulta la API de GitHub
  (`api.github.com/repos/JaredJomar/ARCA/issues`, todos los estados,
  paginado) y convierte cada issue en una barra.
- De cada issue se lee, desde el cuerpo: `GanttStart` y `GanttDue`
  (fechas de la barra), `Depende de:` (flechas de dependencia),
  `Progreso` (porcentaje) y los assignees (se muestran en la barra
  como `@usuario`, o `Sin asignar`).
- Un issue sin `GanttStart` usa su fecha de creación como inicio, y
  uno sin `GanttDue` se dibuja como una barra de 1 día.
- Colores: abierta azul, cerrada verde, atrasada (fecha pasada con el
  issue abierto) roja. Vistas Día, Semana y Mes. Clic en una barra
  abre el issue en GitHub.
- El visor muestra los 110 issues. La vista compartida, en cambio,
  incluye solo las 86 tareas del plan con fechas: los 24 sub-issues
  (#87-110) no tienen fechas de Gantt y no se incluyen ahí.

## Botones

- Exportar HTML: descarga una copia autocontenida del Gantt con los
  datos cargados en ese momento, lista para enviar como archivo.
- Compartir enlace: copia el enlace a la vista compartida
  (`shared.html`).

## Vista compartida

Un script en Python lee los issues en vivo por la API de GitHub,
genera `shared.html` con los datos incrustados y lo publica en este
repo. Corre a diario alrededor de las 7:07 AM (hora de Puerto Rico)
y solo publica si los datos cambiaron desde la última versión.
