# ARCA-gantt

Visor Gantt clásico del proyecto ARCA, publicado en GitHub Pages.
La gráfica rivaliza con el Gantt de OpenProject sin salir de Pages
y sin ningún servidor: la página es estática y la API de GitHub
hace de backend para leer y guardar.

## Sitios

- Visor principal: https://jaredjomar.github.io/ARCA-gantt/
  Carga los datos en vivo desde el repo privado JaredJomar/ARCA.
  Pide un token de GitHub, que queda guardado en el navegador
  (localStorage) y no se envía a ningún otro sitio.
- Vista compartida: https://jaredjomar.github.io/ARCA-gantt/shared.html
  No pide token ni acceso al repo: los datos van incrustados en
  la página. Se regenera automáticamente cada mañana y solo se
  publica una versión nueva si los datos cambiaron.

## Cómo se genera el visor

No hay build ni framework. Todo el visor es un solo archivo,
`index.html`, que se publica directo desde la rama `main` con
GitHub Pages.

- La gráfica la dibuja DHTMLX Gantt Community Edition 10.0.3
  (licencia MIT, gratis), cargado por CDN con la versión fijada.
- JavaScript vanilla consulta la API de GitHub
  (`api.github.com/repos/JaredJomar/ARCA/issues`, todos los
  estados, paginado) y convierte cada issue en una barra.
- De cada issue se lee, desde el cuerpo: `GanttStart` y `GanttDue`
  (fechas de la barra), `Depende de:` (flechas de dependencia),
  `Progreso` (porcentaje) y los assignees (se muestran en la
  barra como `@usuario`, o `Sin asignar`).
- Un issue sin `GanttStart` usa su fecha de creación como inicio,
  y uno sin `GanttDue` se dibuja como una barra de 1 día.
- Un issue con `GanttStart` igual a `GanttDue` se muestra como
  hito (diamante).
- Colores: abierta azul, cerrada verde, atrasada (fecha pasada
  con el issue abierto) roja. Vistas Día, Semana y Mes. La tabla
  muestra tarea, asignado, inicio, fin y progreso. Clic en una
  barra abre el issue en GitHub.
- El visor muestra los 110 issues. La vista compartida, en
  cambio, incluye solo las 86 tareas del plan con fechas: los
  24 sub-issues (#87-110) no tienen fechas de Gantt y no se
  incluyen ahí.

## Edición (visor principal)

El visor no es solo de vista: con un token con escritura se
puede editar el plan desde la gráfica y cada cambio se guarda
en el issue real con un PATCH a su cuerpo (solo se reescriben
las líneas que cambia el visor, el resto del cuerpo no se toca).

- Arrastrar una barra cambia sus fechas; estirarla cambia su
  duración; arrastrar la parte de progreso cambia el porcentaje
  (en tareas cerradas el progreso siempre es 100%).
- Arrastrar de una barra a otra crea una dependencia; para
  quitar un enlace, haz clic en la flecha y pulsa Supr (pide
  confirmación).
- Al crear o quitar un enlace se actualiza el texto `Depende de:`
  del issue y también la relación nativa "Blocked by" en GitHub,
  para que ninguna de las dos fuentes quede vieja.
- El texto del issue solo guarda el par de issues, no el tipo
  de enlace: un enlace creado como SS, FF o SF se guarda como
  dependencia y al recargar vuelve como fin-a-inicio (FS).
- Si el guardado falla, la barra vuelve a su fecha anterior y
  se muestra el error. Si el token es de solo lectura, el visor
  lo dice y revierte el cambio.
- Casilla "Edición": viene apagada por defecto; al marcarla se
  activa la edición.
- Casilla "Mover dependientes" (apagada por defecto): al mover
  una barra, desplaza también a sus sucesores fin-a-inicio para
  que no empiecen antes de que termine el predecesor, y pide
  confirmación con la cantidad de tareas que se moverán.

## Tokens

- Colaboradores: token clásico con scope repo. En GitHub:
  Settings, Developer settings, Personal access tokens,
  Tokens (classic), Generate new token (classic), marca repo.
  El fine-grained no incluye repos de otra cuenta personal,
  por eso no sirve para ARCA si no eres el dueño.
- Dueño del repo: fine-grained con el repo JaredJomar/ARCA en
  lectura de Issues para ver, o en lectura y escritura para
  editar.
- No reutilizar para el navegador el token de automatización
  que usa el cron diario: si se revoca uno, el otro sigue
  funcionando. Un token expuesto se revoca en un clic desde
  GitHub y se genera otro.

## Camino crítico

Cálculo propio en JavaScript, sin la edición PRO de DHTMLX:
orden topológico sobre el grafo de dependencias y ruta más
larga, todo fin-a-inicio. El botón "Camino crítico" resalta las
tareas de la ruta con un borde rojo. Si hay un ciclo en las
dependencias, el visor avisa y no calcula la ruta.

## Baselines

- "Guardar baseline" guarda las fechas actuales de todas las
  tareas en `baseline.json` dentro de este repo (vía API). Pide
  confirmación antes de sobrescribir la anterior.
- "Mostrar baseline" dibuja barras fantasma grises debajo de
  las barras reales, para comparar plan contra real.
- La vista compartida incluye la baseline si existe una
  guardada cuando se regenera.

## Filtros y vistas

- Filtro por asignado, por estado (abierta, cerrada, atrasada)
  y búsqueda por texto o número de issue.
- "Agrupar por persona" ordena las filas por asignado.
- "Carga por persona" muestra una tabla con tareas, abiertas,
  atrasadas y días por asignado (una tarea con varios asignados
  cuenta para cada uno).
- La columna del día actual se tiñe de rojo claro, para ubicar
  hoy de un vistazo. La edición Community no trae marcadores,
  así que el tinte se aplica con una clase de celda.
- La tabla arranca con 660 px de ancho y cada columna se puede
  ensanchar arrastrando el borde derecho de su encabezado. La
  edición Community no incluye resize de columnas, así que está
  implementado a mano. Al pasar el cursor sobre el nombre de una
  tarea se ve el nombre completo.
- La gráfica ocupa el alto de la pantalla (menos los paneles),
  para ver más filas sin desplazar.
- "Imprimir / PDF" usa la impresión del navegador. Para una
  copia completa y portátil, mejor usar "Exportar HTML".

## Botones

- Exportar HTML: descarga una copia autocontenida del Gantt
  con los datos cargados en ese momento, lista para enviar
  como archivo. Es de solo lectura e incluye filtros, camino
  crítico y baseline si la hay.
- Compartir enlace: copia el enlace a la vista compartida
  (`shared.html`).

## Vista compartida

Un script en Python (`refresh_shared.py`, fuera de este repo)
lee los issues en vivo por la API de GitHub, calcula el camino
crítico, lee `baseline.json` si existe, genera `shared.html`
con los datos incrustados y lo publica en este repo. Corre a
diario alrededor de las 7:07 AM (hora de Puerto Rico) y solo
publica si los datos cambiaron desde la última versión. La
vista compartida incluye filtros, camino crítico y baseline,
igual que la exportación, pero sin edición ni token.

## Archivos de este repo

- `index.html`: el visor principal (DHTMLX, edición incluida).
- `shared.html`: la vista compartida, generada a diario.
- `baseline.json`: la baseline guardada (se crea al guardar
  la primera baseline desde el visor).
- `README.md`: este archivo.

## Lo que no hace

Esto rivaliza con el Gantt de OpenProject, no con OpenProject
entero: no hay tableros ágiles, reuniones, registro de tiempo,
costos ni notificaciones, y no hay colaboración en tiempo
real (los cambios se ven al recargar). Tampoco se pueden crear
issues nuevos desde el visor; solo se editan fechas, progreso
y dependencias de los que ya existen. Algunas funciones de
DHTMLX son PRO (marcadores, capas y resize de columnas): el
visor las implementa a mano con código propio para seguir
siendo gratis.
