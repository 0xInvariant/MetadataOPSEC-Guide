# OPSEC de Metadatos: cómo no filtrar tu identidad al compartir archivos

*Una guía sobre los datos ocultos dentro de los archivos que envías — y cómo evitar que te delaten.*

**Versión:** 1.0 · **Última revisión:** junio de 2026

> **Qué es esta guía.** Una guía práctica, guiada por el modelo de amenaza, sobre los metadatos que viajan dentro de archivos corrientes —fotos, PDF, documentos, logs, capturas— y que pueden revelar tu ubicación, tu dispositivo, tu software, tu nombre real y tus hábitos a cualquiera con quien compartas esos archivos. La idea que se repite: **un archivo no es solo su contenido visible; es también un registro silencioso de cómo, cuándo, dónde y con qué se creó.**
>
> **Esta guía es agnóstica de herramientas.** Enseña el *marco de decisión* —qué buscar, por qué importa, y qué ninguna herramienta puede arreglar. Las herramientas son implementaciones de ese marco y cambian con el tiempo; esta guía señala herramientas offline establecidas (exiftool, mat2 y otras; ver §6) como los medios de hoy. El marco es permanente; las herramientas no.

> **Nota sobre el alcance.** Esta guía es defensiva: existe para ayudar a las personas a no filtrar su propia identidad. Quitar los metadatos de tus propios archivos antes de compartirlos es buena higiene, no evasión. No es asesoría legal, y no cubre eliminar metadatos para obstruir una investigación legítima —eso es otra cosa, con otras consecuencias.

---

## Índice

1. [Qué son los metadatos](#1-qué-son-los-metadatos)
2. [Por qué EXIF puede filtrar tu ubicación](#2-por-qué-exif-puede-filtrar-tu-ubicación)
3. [Riesgos en imágenes, PDF, DOCX, logs, capturas, video y audio](#3-riesgos-en-imágenes-pdf-docx-logs-capturas-video-y-audio)
4. [Qué metadatos son peligrosos — según el modelo de amenaza](#4-qué-metadatos-son-peligrosos--según-el-modelo-de-amenaza)
5. [Cómo revisar antes de compartir](#5-cómo-revisar-antes-de-compartir)
6. [Cómo limpiar sin dañar el original](#6-cómo-limpiar-sin-dañar-el-original)
7. [Qué no puede limpiar una herramienta](#7-qué-no-puede-limpiar-una-herramienta)
8. [Checklist antes de enviar archivos](#8-checklist-antes-de-enviar-archivos)
9. [Casos prácticos](#9-casos-prácticos)

---

## 1. Qué son los metadatos

Los metadatos son *datos sobre los datos*: información que un archivo guarda sobre sí mismo, separada del contenido que ves al abrirlo. Una foto te muestra una imagen; sus metadatos registran la cámara, el objetivo, la exposición, la fecha, el software de edición y —a menudo— las coordenadas GPS exactas de dónde se tomó. Un documento te muestra texto; sus metadatos pueden registrar el nombre del autor, la organización, la plantilla, el historial de revisiones y el nombre de usuario de quien lo guardó por última vez.

Esta información existe por razones legítimas. Las cámaras incrustan ajustes para que el software organice y corrija las imágenes. Las suites ofimáticas registran la autoría para colaborar. El problema no es que los metadatos existan; es que **viajan con el archivo por defecto, de forma invisible, y la mayoría de la gente nunca los mira antes de compartir.**

Dos distinciones que conviene retener:

**Incrustados vs. sidecar.** La mayoría de los metadatos peligrosos están *incrustados* —escritos dentro del propio archivo, así que lo siguen a todas partes. Algunos son *sidecar* (un archivo aparte junto al original); esos son más fáciles de notar y descartar. Esta guía se centra en los incrustados, que son los que sorprenden a la gente.

**Contenido vs. contenedor.** Quitar metadatos cambia el contenedor, no el contenido. Una foto sin su GPS sigue siendo la misma foto. Esto importa para la §6: limpiar bien significa quitar el registro oculto sin dañar lo que de verdad querías compartir.

---

## 2. Por qué EXIF puede filtrar tu ubicación

EXIF (Exchangeable Image File Format) es el estándar de metadatos incrustado en la mayoría de las fotos tomadas por teléfonos y cámaras. Es la forma más común en que la gente filtra su ubicación física sin darse cuenta, así que merece su propia sección.

Cuando un teléfono con localización activa toma una foto, puede escribir las **coordenadas GPS** —latitud, longitud, a veces altitud— directamente en el EXIF de la imagen. Esas coordenadas suelen tener precisión de pocos metros. Una sola foto, compartida una vez, puede revelar:

- **Dónde vives**, si se tomó en casa.
- **Dónde trabajas, duermes o sueles estar**, si compartes fotos a lo largo del tiempo —el patrón de coordenadas es más revelador que cualquier coordenada suelta.
- **Dónde está un lugar "privado"** —el refugio de una fuente, un albergue, un punto de encuentro— incluso cuando la imagen visible es inocua.

Más allá del GPS, el EXIF suele cargar también la **marca y el modelo de la cámara** (y en teléfonos, el modelo del dispositivo), la **fecha y hora originales** de la captura, el **software** usado para editar, y a veces **campos de autor o copyright** con un nombre real. Ninguno te ubica como el GPS, pero juntos construyen una huella: dos fotos "anónimas" que comparten el mismo modelo de dispositivo, el mismo software de edición y el mismo patrón horario pueden vincularse a la misma persona.

La trampa es que **nada de esto se ve al mirar la foto**, y compartir se siente casual. Las plataformas sociales suelen quitar el EXIF al subir (aunque no siempre, ni de forma predecible) —pero **el envío directo no**: una foto enviada por correo, por una app de mensajería que conserva el original, por un enlace de descarga o por la nube, muy a menudo lleva el EXIF intacto. En cuanto evitas el reprocesado de una plataforma, vuelves a estar enviando coordenadas crudas.

> Un instinto útil: trata cada foto que hayas tomado como si tuviera tu ubicación estampada, hasta que confirmes lo contrario. Ese es el valor por defecto seguro, y suele ser cierto.

---

## 3. Riesgos en imágenes, PDF, DOCX, logs, capturas, video y audio

Cada tipo de archivo filtra cosas distintas. Conocer la forma de cada riesgo es lo que te permite revisar el lugar correcto en vez de confiar a ciegas en una sola herramienta.

**Imágenes (JPEG, PNG, HEIC, TIFF).** La historia del EXIF de arriba (GPS, dispositivo, fechas, software, autor). Las capturas PNG suelen no llevar GPS, pero pueden cargar bloques de texto con software y fechas. HEIC (las fotos modernas de iPhone) lleva EXIF rico, incluida la ubicación.

**PDF.** Los PDF tienen un diccionario de información del documento y a menudo metadatos XMP: **autor, creador, productor** (el software que lo generó), fechas de creación y modificación, y a veces el título o las palabras clave. Un PDF "exportado desde" una plantilla o cuenta con nombre puede cargar ese nombre. Peor: los PDF pueden incrustar **fuentes, adjuntos y capas**, y —crucial— un PDF que parece una exportación limpia puede contener todavía **contenido anterior que fue tapado visualmente pero no eliminado**. Hay que decirlo sin rodeos: **dibujar un rectángulo negro no es redacción.** Si el texto todavía se puede seleccionar, buscar, copiar, extraer con una herramienta como `pdftotext`, o recuperar de capas y anotaciones, *no* fue redactado —solo fue ocultado. La redacción de verdad elimina el dato subyacente y luego aplana el archivo (ver §6 y §7).

**DOCX (y formatos modernos de Office).** Un DOCX es en realidad un archivo ZIP de archivos XML. Dentro, `docProps/core.xml` y `docProps/app.xml` registran **autor, modificado-por-última-vez, empresa, número de revisión, tiempo total de edición** y fechas. El campo "modificado por última vez" en particular ha quemado a mucha gente: es el nombre de usuario de quien guardó el archivo por última vez, y rara vez es lo que se pretendía publicar. Pero el punto clave es más amplio: **los archivos de Office son contenedores, y limpiar las propiedades del documento no basta.** Hay datos ocultos en muchos sitios —comentarios, cambios registrados, anotaciones, encabezados y pies, texto oculto, XML personalizado, imágenes incrustadas y notas del orador en presentaciones. Inspecciónalos todos, no solo `docProps`. (El Document Inspector integrado de Microsoft encuentra y elimina la mayoría de estas categorías; ejecútalo sobre una *copia*, porque algunos datos eliminados no se pueden restaurar.)

**Logs.** Los logs son metadatos-como-contenido: rutinariamente contienen **nombres de usuario, hostnames internos, direcciones IP, rutas de archivo (que incluyen tu nombre de usuario del directorio home), tokens de API, identificadores de sesión y marcas de tiempo** que revelan zona horaria y patrones de actividad. Un consultor que comparte un log crudo para demostrar un problema puede entregar la topología de red interna del cliente, el nombre de usuario de su propia máquina y credenciales vivas en un solo pegado. Los logs merecen más sospecha que cualquier otro tipo de archivo aquí, porque el dato sensible está tejido en el texto, no escondido en una cabecera.

**Capturas de pantalla.** Las capturas se sienten seguras porque "es solo lo que había en mi pantalla" —pero eso es exactamente el riesgo. Una captura puede capturar **la interfaz alrededor**: otras pestañas del navegador, banners de notificación con nombres de remitentes y vistas previas de mensajes, el reloj del sistema y la zona horaria, el nombre de usuario en el título de una ventana o en una ruta, nombres de archivos abiertos en la barra de tareas, nombres de otras personas en una barra lateral de chat. El contenido de la imagen *es* la fuga. Y las capturas modernas de teléfono cargan sus propios metadatos tipo EXIF (fecha, dispositivo) encima.

**Video y audio.** Los archivos de video pueden cargar hora de creación, campos de dispositivo y software, metadatos de GPS/ubicación (según la fuente), miniaturas incrustadas, subtítulos, capítulos y metadatos a nivel de contenedor. El audio puede exponer campos de grabadora/software, marcas de tiempo y —en el propio contenido— voces, sonidos de fondo y pistas de ubicación. Quitar los metadatos del contenedor o re-codificar ayuda con las *cabeceras*, pero no elimina la información identificable que viaja en el *contenido* del audio o video: una voz reconocible, un punto de referencia en el cuadro, un reflejo, o un sonido ambiente que ubica la grabación. Como con las imágenes, la cabecera y el contenido son dos problemas distintos.

---

## 4. Qué metadatos son peligrosos — según el modelo de amenaza

No todos los metadatos son igual de peligrosos, y "borra todo siempre" es un instinto tosco que malgasta esfuerzo y puede dañar archivos. El enfoque disciplinado es ordenar por **qué revela el metadato** y **contra quién te proteges** —la misma lógica de modelado de amenazas que el resto de este repositorio (ver el *Manual de Seguridad Operacional*).

Una jerarquía aproximada, por impacto:

**ALTO — te ubica o te nombra directamente.**
- Coordenadas GPS (ubicación física).
- Nombre real en campos de autor/copyright/modificado-por.
- Credenciales, tokens o identificadores de sesión en logs.
Deberían quitarse antes de casi cualquier compartición. No solo correlacionan —identifican.

**MEDIO — te crea una huella o te correlaciona.**
- Marca/modelo del dispositivo y cualquier campo tipo número de serie.
- Software de edición y versiones.
- Hostnames internos, direcciones IP, rutas de archivo (que filtran nombres de usuario).
Rara vez te identifican por sí solos, pero vinculan archivos separados al mismo origen, que es como falla la compartimentación.

**BAJO — menor, normalmente inofensivo.**
- Marcas de tiempo y zona horaria (aunque la zona horaria estrecha tu ubicación, y los patrones de horario correlacionan —así que "bajo" no es "cero").
- Orientación, resolución, perfil de color.

**DESCONOCIDO — la categoría más importante en seguridad.**
- Un formato que la herramienta no parseó, no pudo enumerar del todo, o no soporta.
- **"No detectado" no significa "limpio".** Trata un archivo no parseado o no soportado como *no probado limpio*, no como seguro. Esta es la categoría que atrapa a quien ejecutó una herramienta, no vio nada y envió el archivo. Ante la duda, re-codifica o convierte a un formato sin metadatos (o a texto plano), en vez de confiar en un escaneo vacío.

El modelo de amenaza decide el umbral:

- Para un **usuario común** que comparte fotos de vacaciones, quitar el GPS es la jugada de alto valor; el resto rara vez importa.
- Para un **periodista o activista**, el umbral baja con fuerza: las huellas de dispositivo y las fechas que vinculan archivos, y cualquier campo que nombre a una fuente, pasan todos a ALTO, porque el adversario está intentando activamente correlacionar.
- Para un **consultor**, dominan los logs y los campos de autor de documentos: el riesgo es filtrar los internos del cliente o tu propia identidad a través de los entregables.

El objetivo de ordenar no es relajarse —es gastar tu atención donde está el daño, y evitar la falsa seguridad de "ejecuté una herramienta una vez" sin saber qué importaba.

---

## 5. Cómo revisar antes de compartir

Revisar significa *mirar antes de enviar*, deliberadamente, como hábito —no confiar en que "seguro está bien". Una rutina viable, en orden de esfuerzo:

**Construye primero el instinto.** Antes de cualquier herramienta, interioriza el valor por defecto de la §2: asume que tus propias fotos llevan ubicación, que tus propios documentos llevan tu nombre, que tus propios logs llevan credenciales. Revisar es confirmar o quitar esas suposiciones, no descubrirlas desde cero.

**Inspecciona, no adivines.** Usa una herramienta (exiftool, el `--show` de mat2, u otro visor de metadatos de confianza) para *ver* lo que un archivo contiene realmente antes de decidir. La meta de un escaneo es convertir "seguro está bien" en una lista concreta: este archivo tiene GPS, aquel modelo de dispositivo, este nombre de autor. No puedes tomar una buena decisión sobre qué quitar hasta que puedes ver qué hay.

**Revisa los puntos débiles conocidos del tipo de archivo (§3).** Para una imagen, busca GPS y autor. Para un DOCX, mira `docProps` por autor y modificado-por, y revisa cambios registrados y comentarios. Para un PDF, mira productor/autor y pregúntate si algo fue tapado pero no eliminado. Para un log, léelo como texto y caza nombres de usuario, rutas, IP y tokens. Para una captura, mira el cuadro entero, no solo la parte que querías capturar.

**Revisa también el contenido visible, no solo las cabeceras.** Este es el paso que las herramientas no pueden hacer por ti. Un escaneo encuentra el GPS del EXIF; no nota que la foto muestra la placa de tu calle, que la captura incluye una notificación con un nombre real, o que el PDF "redactado" todavía tiene texto seleccionable bajo el rectángulo negro. Revisar metadatos y revisar contenido son dos pasadas, y necesitas ambas.

**Decide contra tu umbral (§4), luego actúa.** Sabiendo qué hay y contra quién te proteges, decide qué debe irse. Luego limpia —con cuidado (§6).

---

## 6. Cómo limpiar sin dañar el original

Limpiar metadatos es el momento con más probabilidad de salir mal, porque el acto de reescribir un archivo puede corromperlo, quitar algo que necesitabas, o —peor— darte falsa confianza de que funcionó cuando no. La disciplina aquí se reduce a un principio:

> **Flujo seguro — el procedimiento, en orden:**
> 1. **Haz una copia.** Nunca trabajes sobre el original.
> 2. **Inspecciona la copia** —mira qué metadatos hay de verdad.
> 3. **Revisa el contenido visible** —los píxeles/texto, no solo las cabeceras.
> 4. **Limpia la copia.**
> 5. **Vuelve a escanear la copia limpia** —confirma que los metadatos se fueron.
> 6. **Verifica a mano** —el juicio que la herramienta no aplica.
> 7. **Comparte solo la copia limpia.**
> 8. **Conserva o destruye el original** según tu modelo de amenaza y tus obligaciones. Si el archivo es prueba, un registro de negocio, material de un cliente, o parte de un proceso legal o de cumplimiento, **preserva el original** y trabaja solo sobre una copia redactada —destruirlo puede ser en sí mismo un problema (ver la nota de alcance al inicio de esta guía).

El único principio detrás de los ocho pasos:

> **Nunca modifiques el original por defecto.** Produce siempre una *copia limpia*, guarda el original offline, y comparte solo la copia. Una herramienta que edita in situ está a un flag accidental de destruir tu única versión.

Principios para limpiar de forma segura:

**Trabaja sobre una copia; preserva el original.** Guarda el original intacto en un lugar que controles y nunca lo compartas. Opera sobre un duplicado. Esto no es negociable: si limpiar corrompe el archivo o quita de más, no has perdido nada.

**Prefiere eliminar a editar.** Para la mayoría de archivos, la operación segura es *eliminar* el bloque de metadatos entero (p. ej., quitar el segmento EXIF de un JPEG) en vez de vaciar campos individuales, que puede dejar residuos. El enfoque correcto reconstruye el archivo sin el bloque de metadatos en vez de sobrescribir campos in situ.

**Re-codifica cuando necesites certeza —pero verifica.** Para imágenes donde quieras estar seguro de que nada oculto sobrevive, re-codificar (decodificar los píxeles y guardar un archivo nuevo) puede descartar los metadatos originales —*si tu flujo de re-codificación no copia metadatos al resultado*. Muchas herramientas preservan o re-adjuntan metadatos si se lo pides, y los formatos complejos pueden conservar metadatos de contenedor o de streams a través de una re-codificación. Así que trata la re-codificación como una opción fuerte, no como una garantía automática, y vuelve a escanear el resultado. Para capturas y fotos compartidas con desconocidos, el intercambio (ligera pérdida de calidad por un resultado limpio que luego verificas) suele valer la pena.

**Verifica después de limpiar —no asumas.** Vuelve a escanear la copia limpia. El fallo más común es creer que una herramienta funcionó cuando un campo sobrevivió, o cuando se omitió un segundo bloque de metadatos (p. ej., XMP junto al EXIF). Limpiar sin verificar es como la gente termina enviando archivos "limpios" que todavía llevan GPS.

**Aplana solo después de eliminar —aplanar por sí solo no es redacción.** Para PDF y documentos donde contenido anterior o cambios registrados puedan persistir, exportar/imprimir a un PDF nuevo y aplanado, o copiar el texto final a un documento limpio, elimina la historia enterrada. Pero aplanar *solo ayuda después de que el contenido sensible se haya eliminado de verdad*: aplanar un rectángulo negro sobre texto vivo puede dejar el texto recuperable, según el flujo de trabajo. Verifica siempre intentando seleccionar, buscar, copiar/pegar y extraer con `pdftotext` el contenido supuestamente eliminado —y revisa en un segundo visor. El PDF es traicionero; confirma, no asumas.

**Cuidado con las operaciones in situ y por lotes.** Las dos acciones de mayor riesgo son editar el original directamente y limpiar una carpeta entera de una vez. Si haces cualquiera, hazlo sobre copias, y verifica una muestra. Una herramienta bien diseñada exige un flag explícito y confirmación antes de tocar un original.

### Herramientas que implementan estos flujos

Estos flujos son agnósticos de herramientas, pero tienen que ejecutarse sobre *algo*. Herramientas establecidas, offline y de código abierto cubren la mayoría de las necesidades de hoy. Ninguna es perfecta; cada una tiene límites, que es justo por qué el marco de decisión de arriba importa más que la herramienta.

- **exiftool** — la veterana herramienta multiplataforma (Windows/Mac/Linux) para *leer, escribir y borrar* metadatos en un rango de formatos excepcionalmente amplio. Mejor para *inspección* (ver qué hay) y ediciones quirúrgicas. Al escribir, preserva el original por defecto, renombrándolo con `_original` añadido (usa `-overwrite_original` para suprimirlo). Aun así: trabaja sobre tus propias copias y verifica el resultado antes de borrar cualquier respaldo.
- **mat2** (Metadata Anonymisation Toolkit 2) — una herramienta defensiva de *eliminación* que, por diseño, escribe una copia `.cleaned` y nunca toca tu original, y solo muestra los metadatos que considera reveladores. Excelente para "limpia y listo" de un tiro. Nota su estado de mantenimiento: el repositorio oficial en 0xacab está archivado y es de solo lectura, así que es mejor tratarla como estable y terminada, no mantenida activamente —verifica el paquete de tu distribución, cualquier fork activo, y el estado general de mantenimiento antes de confiar en ella para flujos de alto riesgo, y verifica las salidas en vez de asumir que son completas. (Un front-end gráfico, Metadata Cleaner, usa mat2 por debajo para quien evita la terminal.)
- **Herramientas PDF** (`pdfinfo`, `pdftotext`, `qpdf`, y visores con soporte real de redacción) — para inspección, comprobaciones de extracción, inspección estructural y verificación. `pdftotext` es como *compruebas* si el texto supuestamente eliminado sigue ahí; `qpdf` inspecciona y reestructura. No las trates como herramientas de redacción automática.
- **ffmpeg** — para re-codificar o remuxear imágenes, audio y video, y quitar o evitar arrastrar metadatos cuando se configura correctamente; una de las vías más prácticas para los flujos comunes de metadatos de video, pero verifica después (no toca el contenido, ni subtítulos quemados, ni todos los streams/capítulos/adjuntos).
- **Office / LibreOffice** — para exportar una copia limpia de un documento y quitar cambios registrados, comentarios y los campos de autor de `docProps`.
- **ripgrep / scripts / revisión manual** — para logs, donde el dato sensible es *contenido*, no cabecera, y debe leerlo y depurarlo un humano (ver §7).

Una matriz de referencia de dónde vive cada riesgo, qué lo aborda hoy, y cuánto puedes confiar en el resultado:

| Tipo de archivo | Herramientas hoy | Confianza | NO resuelve |
|---|---|---|---|
| JPEG / HEIC | exiftool, mat2, re-codificar | Alta | pistas visibles en el cuadro, ruido del sensor |
| PNG / capturas | exiftool, mat2, recortar/re-codificar | Alta (cabecera), Humana (contenido) | todo lo visible en la captura |
| PDF | qpdf, pdftotext, aplanar | Media | redacción mala, capas, adjuntos |
| DOCX | Document Inspector de Office, LibreOffice | Media | objetos incrustados, revisiones enterradas |
| Logs | ripgrep / scripts / manual | Requiere revisión humana | secretos sensibles al contexto |
| Video / audio | ffmpeg, exiftool | Media | voces, entorno, pistas de sonido, streams/capítulos/subtítulos |

Las dos últimas columnas llevan la lección real: no todos los formatos son igual de limpiables, y toda herramienta deja algo que no resuelve. Quitar la cabecera de un JPEG es fiable para la *cabecera*; un PDF o un DOCX esconden historia que la limpieza por campos omite; un log es *contenido*, así que solo un humano que lo lea es de fiar. **Herramienta ≠ seguridad.**

**Ejemplos mínimos de comandos.** Son puntos de partida, no recetas listas —los flags exactos dependen del formato, y las operaciones destructivas varían. **Ejecútalos siempre sobre una copia y vuelve a escanear el resultado.**

```sh
# INSPECCIONAR (solo lectura, seguro):
exiftool archivo.jpg                    # mostrar todos los metadatos
mat2 --show archivo.pdf                 # mostrar lo que mat2 considera revelador
pdftotext archivo.pdf -                 # volcar el texto del PDF (¿sigue ahí lo "redactado"?)

# LIMPIAR (escribe; trabaja sobre copias):
exiftool -all= archivo.jpg              # quita todos los metadatos (deja respaldo _original). Puede quitar también perfiles de color, etc.; verifica el resultado visual
mat2 archivo.pdf                        # escribe archivo.cleaned.pdf, el original intacto
ffmpeg -i in.mp4 -map_metadata -1 -c copy out.mp4   # quita metadatos comunes del contenedor; verifica streams/capítulos/subtítulos aparte

# VERIFICAR (solo lectura): re-ejecuta los comandos de INSPECCIONAR sobre la copia limpia.
```

> **Una regla dura sobre los limpiadores online.** Las webs de "quitar metadatos" son aceptables **solo para archivos de baja sensibilidad**. Subir un archivo a un limpiador web puede quitar sus metadatos locales, pero entrega el *archivo entero* a un tercero —contenido incluido. Para cualquier cosa personal, legal, periodística, laboral o que involucre a una fuente, **usa herramientas locales/offline**. Quitar una etiqueta de GPS entregando la foto completa a un servidor desconocido no es una victoria.

Una precaución que las propias herramientas declaran sin rodeos: un *escaneo* limpio no significa un archivo *seguro*. La documentación de mat2 advierte que aunque su `--show` no encuentre nada, el archivo puede seguir llevando metadatos —no hay forma fiable de enumerar todos los campos en formatos complejos. Así que limpia de forma defensiva aunque un escaneo parezca vacío, y verifica siempre el resultado. Esta humildad es la postura correcta, y conduce directo a la siguiente sección.

---

## 7. Qué no puede limpiar una herramienta

Esta es la sección que separa la guía honesta de la falsa sensación de seguridad. Una herramienta de metadatos quita *metadatos*. No quita información sensible que vive en el **contenido** del archivo, y no entiende qué es sensible *para ti*. Los límites son reales y hay que nombrarlos:

**Contenido sensible en la propia imagen.** Ninguna herramienta que quite EXIF notará que la foto muestra el número de tu casa, una placa de calle, el reflejo de tu cara en una ventana, una credencial, o una pantalla de fondo. Los píxeles son el contenido; la herramienta solo toca la cabecera.

**Datos "redactados" visualmente pero recuperables.** Un rectángulo negro dibujado sobre texto en un PDF, o un desenfoque reversible, no es redacción —el texto o los píxeles subyacentes a menudo siguen ahí, seleccionables o recuperables. Un limpiador de metadatos no arregla esto. La redacción de verdad es eliminar el dato, luego aplanar.

**Contenido oculto o tapado en documentos.** Texto escondido detrás de imágenes, texto blanco sobre blanco, contenido en revisiones anteriores, porciones de imagen recortadas-pero-incrustadas (algunos formatos guardan los píxeles recortados), y notas del orador en presentaciones. La limpieza de metadatos por campos deja todo esto.

**El acto de compartir en sí.** Los metadatos de *transmisión* —tu dirección IP al subir, las cabeceras de correo, la cuenta desde la que enviaste, la marca de tiempo del envío— están fuera del archivo por completo. Limpiar el archivo no hace nada sobre el canal. (Aquí esta guía cede el paso a la OPSEC de red.)

**Correlación entre archivos.** Incluso archivos perfectamente limpios pueden vincularse por su *estilo* de contenido, por la hora a la que los envías, o por la cuenta que usas. Una herramienta limpia un archivo; no gestiona tu comportamiento a través de muchos.

**Ruido del sensor — una huella que los limpiadores de metadatos no abordan.** Muchos sensores de imagen dejan patrones de ruido específicos del dispositivo (PRNU — Photo Response Non-Uniformity) que *a veces* pueden usarse para identificar la cámara de origen: el análisis forense puede vincular imágenes a la cámara concreta que las tomó, incluso después de quitar todos los metadatos, porque el patrón vive en los datos de píxel, no en la cabecera. Con qué fiabilidad depende de la compresión, del pipeline de procesamiento de la cámara, de cuántas imágenes haya disponibles y de la capacidad del analista —es una técnica real, no magia, y no está garantizada. Los limpiadores de metadatos no la abordan en absoluto. Para la mayoría de los modelos de amenaza no importa; para una fuente cuyas fotos no deben atarse a su dispositivo, es razón para usar un dispositivo de captura dedicado cuyas salidas no se reutilicen entre identidades (una cámara dedicada igual crea patrón si cada foto remite siempre a la misma identidad), o para aceptar que re-codificar no vuelve una imagen verdaderamente anónima en su origen.

**Qué es sensible es tu juicio, no el de la herramienta.** Una herramienta puede marcar el GPS como ALTO porque el GPS siempre ubica. No puede saber que cierta marca de tiempo de aspecto corriente te coloca en un lugar donde dijiste que no estabas. La revisión final es humana.

El encuadre honesto: una herramienta de metadatos es necesaria y valiosa, y **no es suficiente**. Ayuda a inspeccionar y quitar muchas formas comunes de metadatos ocultos en los formatos soportados; el contenido visible, la historia enterrada, el canal de transmisión y tu propio juicio son tuyos.

---

## 8. Checklist antes de enviar archivos

Una pasada corta y repetible. Adapta el umbral a tu modelo de amenaza (§4).

**Antes de enviar cualquier archivo:**
- [ ] ¿De verdad necesito enviar *este* archivo, o puedo enviar menos (un recorte, un resumen, una exportación nueva)?
- [ ] ¿Lo escaneé y *vi* qué metadatos contiene —no lo asumí?
- [ ] ¿Hay GPS, un nombre real, o credenciales/tokens? (Si sí → quitar antes de enviar.)
- [ ] ¿Estoy trabajando sobre una **copia**, con el original preservado intacto?
- [ ] ¿Volví a **escanear la copia limpia** para confirmar que los metadatos se fueron de verdad?

**Para imágenes / capturas:**
- [ ] ¿GPS quitado (o re-codificado para estar seguro)?
- [ ] ¿La imagen visible muestra algo identificable (placas, caras, pantallas, credenciales)?
- [ ] Para capturas: ¿revisé el *cuadro entero* —pestañas, notificaciones, reloj, usuarios, barras laterales?

**Para PDF / documentos:**
- [ ] ¿Campos de autor / modificado-por / empresa revisados y limpiados?
- [ ] ¿Cambios registrados y comentarios eliminados?
- [ ] ¿Lo "redactado" realmente eliminado y aplanado, no solo tapado?

**Para logs:**
- [ ] ¿Usuarios, rutas de archivo, hostnames internos, IP depurados?
- [ ] ¿Tokens, claves, IDs de sesión eliminados?
- [ ] ¿Marcas de tiempo revisadas (zona horaria / patrón de actividad aceptable de revelar)?

**Siempre:**
- [ ] ¿Estoy a punto de subir esto a un limpiador online? Si el archivo es sensible, **detente** —usa herramientas locales/offline.
- [ ] Original guardado offline; solo la copia limpia compartida.
- [ ] Si la herramienta no reportó nada, ¿traté el archivo como *no probado limpio* en vez de seguro (DESCONOCIDO, §4)?

---

## 9. Casos prácticos

Un conjunto de escenarios cortos e ilustrativos que muestran cómo cambia el umbral con el modelo de amenaza. Son ejemplos, no recetas.

### Un periodista enviando fotos

Un reportero fotografía un documento en la ubicación de una fuente y envía las imágenes a un editor. El contenido visible es solo el documento —pero el **GPS del EXIF revelaría la ubicación de la fuente**, y el modelo de dispositivo y las fechas podrían vincular estas fotos con el resto de su trabajo. Umbral: ALTO en todo. La jugada correcta es re-codificar las imágenes con un flujo que no copie metadatos al resultado, verificar que ningún GPS sobrevive, comprobar que nada en el *cuadro* (una vista por la ventana, un reflejo, una dirección visible) identifique el lugar, y enviar solo las copias limpias. Aquí los metadatos no son un detalle menor de higiene; una sola coordenada podría poner en peligro a una persona.

### Un activista compartiendo un PDF

Un organizador prepara un PDF de un folleto en una suite ofimática y lo comparte públicamente. Los campos de **productor y autor del PDF llevan su nombre real**, y el documento se construyó a partir de un borrador anterior. Umbral: ALTO en los campos de identidad, y un riesgo real de historia enterrada. La jugada: limpiar los metadatos de autor/productor, y —como puede persistir contenido de revisiones anteriores— aplanar el documento (exportar a un PDF nuevo) en vez de confiar en la limpieza por campos. Luego confirmar que el nombre se fue del archivo nuevo. El folleto visible se pretende público; la identidad del autor, no.

### Un consultor enviando logs

Un consultor de seguridad pega un fragmento de log en un informe para demostrar un hallazgo. El log contiene los **hostnames internos e IP del cliente, el nombre de usuario de la propia máquina del consultor en las rutas, y un token de sesión vivo**. Umbral: ALTO —y el peligro está en el *contenido*, no en una cabecera, así que ninguna herramienta de metadatos por sí sola lo resuelve. La jugada: leer el log como texto, depurar usuarios, rutas, hostnames internos, IP y cualquier credencial, reemplazarlos con marcadores claramente señalados, y conservar solo lo que demuestra el punto. Este es el caso que más fiablemente quema a la gente técnica, precisamente porque confían en un archivo "limpio" sin leerlo.

### Un usuario común compartiendo capturas

Alguien hace una captura de un chat para compartir un mensaje gracioso con un amigo. La captura también captura **un banner de notificación con el nombre de otro contacto y una vista previa del mensaje, el reloj del sistema, y el título de una pestaña que revela qué estaba haciendo**. Umbral: mayormente BAJO, pero la captura incidental es la fuga. La jugada: recortar a solo la parte relevante antes de compartir, echar un vistazo al cuadro entero por nombres y vistas previas, y —si se comparte más allá del amigo— quitar los metadatos de fecha de la propia captura. Bajo riesgo, pero el hábito de *mirar el cuadro entero* es lo que evita el accidente ocasional de alto riesgo.

### Un desarrollador abriendo un reporte de bug open-source

Alguien adjunta logs y capturas a un issue público de GitHub para pedir ayuda con un bug. El pegado filtra, de una sola vez, **su nombre de usuario del directorio home (visible en cada ruta de archivo), el contenido de un archivo `.env`, un token de API vivo, un hostname interno, y el nombre de un repositorio privado** —nada de lo cual pretendía publicar, y todo ahora en un issue público, indexado y archivado para siempre. Umbral: ALTO, y es el *contenido* de los logs y el *cuadro* de las capturas lo que filtra, no las cabeceras. La jugada: antes de publicar, leer cada línea; reemplazar usuarios, rutas, hostnames, nombres de repo y correos con marcadores claramente señalados; eliminar cualquier token, clave o contenido de `.env` (y rotar cualquier credencial ya expuesta); y recortar las capturas al error, revisando el cuadro entero. Los rastreadores de bugs públicos son para siempre —trata cualquier cosa que pegues ahí como publicada.

### Quien busca empleo compartiendo un CV en PDF

Alguien exporta su CV a PDF y lo sube a portales de empleo y lo envía por correo a reclutadores. El PDF lleva, en sus metadatos, **los campos de autor y empresa (a veces el nombre de un empleador actual), el origen de la plantilla, un nombre de usuario viejo, y el historial de modificaciones del documento** —y si se construyó a partir de una versión anterior, posiblemente texto de puestos o notas de salario que creía haber borrado. Umbral: MEDIO en los campos de identidad, con un riesgo real de contenido de borrador enterrado. La jugada: limpiar los campos de autor/empresa/productor, luego exportar un PDF nuevo (o imprimir a PDF) tras eliminar los datos ocultos para descartar el historial de revisiones, y volver a escanear para confirmar; luego leer el documento visible una vez más por si algo se arrastró de un borrador anterior. Un CV está hecho para compartirse —que es exactamente por qué sus campos ocultos merecen una mirada.

---

## Referencias y verificación

Esta guía separa a propósito la toma de decisiones de las herramientas. El comportamiento de las herramientas cambia con el tiempo; verifica los comandos y los formatos soportados contra la documentación oficial antes de confiar en ellos para flujos de alto riesgo. Las referencias de abajo son fuentes oficiales o primarias de las herramientas y la orientación mencionadas, al momento de escribir.

- **ExifTool** — sitio oficial y documentación: `https://exiftool.org/` (existe un mirror en SourceForge si el sitio principal no está disponible).
- **mat2** — repositorio y documentación oficiales: `https://0xacab.org/jvoisin/mat2` (desarrollado con el apoyo del proyecto Tails).
- **Document Inspector de Microsoft** — "Quitar datos ocultos e información personal inspeccionando documentos, presentaciones o libros", en el sitio de soporte oficial de Microsoft (`support.microsoft.com`).
- **FFmpeg** — documentación oficial: `https://ffmpeg.org/documentation.html`.
- **Redacción de PDF** — consulta la documentación oficial de tu software de PDF para su función de *redacción* específicamente (p. ej., las herramientas de redacción de Adobe Acrobat), y ten en cuenta la orientación de fuentes como la NSA sobre sanitización de documentos: la redacción de verdad elimina el contenido subyacente; no es lo mismo que dibujar encima. Verifica intentando extraer el contenido supuestamente eliminado (ver §6).

Los comandos aquí se escribieron como ejemplos, no como garantías; pruébalos contra tus versiones instaladas. Trata siempre la documentación oficial como la fuente de verdad por encima de cualquier guía, incluida esta. Si un comando o comportamiento descrito aquí no coincide con lo que hace tu versión instalada, confía en tu versión y en los documentos oficiales.

---

> **El hilo, una vez más.** Los metadatos filtran identidad porque viajan de forma invisible y por defecto. Herramientas como exiftool y mat2 ayudan a inspeccionar y quitar muchas formas comunes de metadatos ocultos. Pero el contenido visible, la historia enterrada, el ruido del sensor en los píxeles, el canal de transmisión y el juicio de qué es sensible *para ti* siguen siendo trabajo humano. Limpia el archivo, sí. Pero revísalo como la persona cuya identidad depende de ello —porque a veces así es.

---

*Esta guía es agnóstica de herramientas: enseña el marco de decisión, y señala herramientas offline establecidas como implementaciones de hoy. Aplica el marco de modelado de amenazas del Manual de Seguridad Operacional a un riesgo concreto y cotidiano. De naturaleza defensiva; no es asesoría legal.*