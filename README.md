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
- El texto `Depende de:` guarda también el tipo de enlace
  (SS, FF o SF); sin paréntesis es fin-a-inicio (FS). El camino
  crítico y "Mover dependientes" solo usan enlaces fin-a-inicio.
- Si el guardado falla, la barra vuelve a su fecha anterior y
  se muestra el error. Si el token es de solo lectura, el visor
  lo dice y revierte el cambio.
- Casilla "Edición": viene apagada por defecto; al marcarla se
  activa la edición.
- Casilla "Mover dependientes" (apagada por defecto): al mover
  una barra, desplaza también a sus sucesores fin-a-inicio para
  que no empiecen antes de que termine el predecesor, y pide
  confirmación con la cantidad de tareas que se moverán.
- "Deshacer" y "Rehacer" (botones o Ctrl+Z / Ctrl+Y) revierten
  movimientos, cambios de progreso y enlaces creados o quitados.
  No cubren la creación de tareas.
- "Nueva tarea" crea un issue nuevo con título, asignado y
  fechas (requiere token con escritura).

## Jerarquía y WBS

Los sub-issues de GitHub se muestran anidados bajo su tarea
padre (el mapa padre-hijo vive en `subissues.json` en este
repo). La columna "WBS" numera la jerarquía (1, 1.1, 1.2, 2).
Las fechas y el progreso de una tarea padre se calculan de
sus hijas (inicio mínimo, fin máximo, progreso ponderado por
duración), así que una tarea padre no se puede arrastrar: hay
que mover sus subtareas. La vista compartida y la exportación
muestran la misma jerarquía.

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
larga, solo con enlaces fin-a-inicio (FS). Los enlaces SS, FF
y SF no entran en el cálculo. El botón "Camino crítico" resalta las
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
  y búsqueda por texto o número de issue. Buscar un número
  (con o sin #, por ejemplo 108) muestra solo ese issue; si es
  una subtarea, su padre queda visible y abierto para que se vea.
- En móviles (hasta 700 px de ancho) la tabla muestra solo la
  columna de tarea y la línea de tiempo se desplaza; pestaña
  Gráfica con gráfica circular solo de tareas técnicas, por
  persona o por estado, según el filtro actual. Cada grupo
  tiene su propio color de una paleta de 11 (azul, verde, lima,
  dorado, naranja, rojo, rosa, morado, índigo, marrón y gris);
  Jared (JaredJomar) siempre es cian, sin importar cuántos
  grupos haya.
  Las tareas de PM quedan fuera de la gráfica: se detectan por palabras
  clave en el título (planificación, sprints, checkpoints,
  revisiones, hitos, documentación y demos, entre otras).
- En pantallas pequeñas el deslizamiento táctil mueve la línea
  de tiempo (DHTMLX con touch forzado); los filtros se guardan
  en el navegador y sobreviven a recargar la página, y el botón
  "Limpiar filtros" los deja en su valor por defecto sin
  recargar.
- Cuando el filtro deja pocas tareas visibles, las filas se
  estiran para usar el espacio disponible y que las flechas de
  dependencia tengan hueco entre las barras; con muchas tareas,
  las filas quedan compactas como antes.
- "Agrupar por persona" ordena las filas por asignado.
- "Carga por persona" muestra una tabla con tareas, abiertas,
  atrasadas, días laborables y máxima simultaneidad (la mayor
  cantidad de tareas abiertas de esa persona el mismo día) por
  asignado (una tarea con varios asignados cuenta para cada uno).
  Los días son laborables: lunes a viernes, sin los feriados de
  la lista del código.
- Las columnas "Desv. ini" y "Desv. fin" muestran en días la
  desviación contra la baseline (+ es atraso, - es adelanto;
  vacías si no hay baseline guardada).
- La columna del día actual se tiñe y lleva una línea roja,
  para ubicar hoy de un vistazo, y los fines de semana se
  sombrean. La edición Community no trae marcadores, así que
  la línea se logra con una sombra interior en cada celda.
- La tabla arranca con 885 px de ancho y cada columna se puede
  ensanchar arrastrando el borde derecho de su encabezado. La
  edición Community no incluye resize de columnas, así que está
  implementado a mano. Al pasar el cursor sobre el nombre de una
  tarea se ve el nombre completo.
- La gráfica ocupa el alto de la pantalla (menos los paneles),
  para ver más filas sin desplazar.
- "Imprimir / PDF" usa la impresión del navegador. Para una
  copia completa y portátil, mejor usar "Exportar HTML".

## Actualización automática y foco en dependencias

- "Actualización automática" (activada por defecto, se guarda
  en el navegador) revisa GitHub cada minuto y actualiza el
  Gantt en vivo, sin recargar la página: si un compañero cierra
  un issue, cambia una fecha o un progreso, la barra se mueve
  sola y la línea de estado dice qué cambió. No toca el
  desplazamiento, la selección ni los filtros, y no revisa
  mientras arrastras una barra, mientras se guarda un cambio o
  mientras la pestaña está oculta. Tras 3 intentos fallidos
  seguidos se pausa sola y avisa que revises el token.
- Clic en una flecha de dependencia filtra el Gantt al issue
  al que apunta la flecha y a todas sus dependencias (la cadena
  completa, no solo las directas), para distinguirlas cuando un
  issue tiene muchas. Para quitar el filtro: clic en la misma
  flecha, clic en un espacio vacío, pulsa Esc o usa el botón
  "Quitar filtro" del aviso amarillo. Borrar el enlace también
  quita el filtro. El aviso indica la flecha exacta, por
  ejemplo "Flecha #12 → #45 (Fin a inicio)".
- Flechas de colores: cada flecha lleva el color de la barra de
  la tarea origen (azul abierta, verde cerrada, roja atrasada),
  así que cuando varias se cruzan se distingue de dónde viene
  cada una. En camino crítico la flecha es roja. Al pasar el
  cursor por una tarea o por una flecha, las demás flechas se
  atenúan y solo queda visible su cadena.

## Botones

- Exportar HTML: descarga una copia autocontenida del Gantt
  con los datos cargados en ese momento, lista para enviar
  como archivo. Es de solo lectura e incluye filtros, camino
  crítico y baseline si la hay.
- Compartir enlace: copia el enlace a la vista compartida
  (`shared.html`).
- Exportar CSV: descarga las tareas (número, WBS, título,
  asignado, fechas, progreso y estado) en un archivo que abre
  Excel.
- Exportar Project XML: descarga las tareas, dependencias y
  asignados en el formato XML que abre MS Project como proyecto
  nuevo.

## Vista compartida

Un script en Python (`refresh_shared.py`, fuera de este repo)
lee los issues en vivo por la API de GitHub, calcula el camino
crítico, lee `baseline.json` si existe, genera `shared.html`
con los datos incrustados y lo publica en este repo. Corre a
diario alrededor de las 7:07 AM (hora de Puerto Rico) y solo
publica si los datos cambiaron desde la última versión. La
vista compartida incluye filtros, camino crítico, baseline,
jerarquía, WBS y varianza, igual que la exportación, pero sin
edición ni token. Al abrir, la línea de tiempo arranca en la semana actual (el lunes de esta semana); se puede desplazar horizontalmente para ver otras semanas.

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
real (los cambios se ven al recargar). No hay nivelación
automática de recursos ni programación automática con
restricciones: "Mover dependientes" solo empuja sucesores
fin-a-inicio. No se genera .mpp binario (el formato es
propietario); para MS Project se exporta XML. Algunas funciones de
DHTMLX son PRO (marcadores, capas y resize de columnas): el
visor las implementa a mano con código propio para seguir
siendo gratis.
