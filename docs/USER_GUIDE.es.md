# DBApguin - Guía de usuario

**Para DBApguin 1.0.3**

*Esta guía se ha traducido del inglés con ayuda de IA. Si algo no está claro o difiere, prevalece el original en inglés (Ayuda → Guía de usuario, con la interfaz en inglés).*

DBApguin es una consola de escritorio para tareas de DBA en PostgreSQL. Esta
guía cubre todas las funciones desde el punto de vista de quien las usa y se
distribuye con la aplicación, así que siempre describe la versión que tienes
delante.

Ábrela cuando quieras desde **Ayuda → Guía de usuario**.

## 1. Instalación

### Linux

1. Descarga `DBApguin-<versión>-x86_64.AppImage`.
2. Hazlo ejecutable: `chmod +x DBApguin-*.AppImage`
3. Doble clic o ejecútalo desde un terminal.

El AppImage es autónomo (~27 MB) y guarda sus datos en `~/.local/share/DBApguin/`.

Si la ventana integrada no aparece, el AppImage muestra un diálogo con instrucciones de instalación para cada gestor de paquetes importante (apt, dnf, pacman, zypper, apk, emerge). El diálogo selecciona solo la pestaña de tu distribución y ofrece un botón «Copiar comando». Para Debian/Ubuntu el comando es:

```
sudo apt-get install python3-gi python3-gi-cairo gir1.2-gtk-3.0 gir1.2-webkit2-4.1
```

DBApguin es una aplicación de escritorio independiente: si WebKit2GTK no está disponible, NO abrirá en silencio una pestaña en tu navegador. Instala el paquete y vuelve a lanzar el AppImage.

### Windows

1. Descomprime el zip de DBApguin donde quieras (Escritorio, `C:\Tools\`, etc.).
2. Abre la carpeta.
3. Doble clic en `DBApguin.exe`.

El primer arranque puede mostrar SmartScreen («Windows protected your PC»). Pulsa «More info» y luego «Run anyway». DBApguin todavía no está firmado para Windows.

Si falta Microsoft Edge WebView2 Runtime, la aplicación abre un diálogo con tres vías de instalación: (1) lanzar el `MicrosoftEdgeWebView2Setup.exe` incluido en la carpeta de la aplicación, (2) descargar el bootstrapper en línea desde `developer.microsoft.com/microsoft-edge/webview2/`, o (3) desplegar el instalador independiente mediante GPO / SCCM / Intune en entornos gestionados. Cada bloque tiene un botón «Copiar» para rutas y enlaces.

La aplicación no se instala en el sistema; se ejecuta desde su carpeta. Para quitarla, borra la carpeta.

El diálogo de configuración lee su idioma de tus preferencias de DBApguin (`app_prefs.json` -> `language`); en el primer arranque sigue la configuración regional del sistema (croata para `hr_*`, inglés en los demás casos).

## 1.4 Versiones de PostgreSQL admitidas

DBApguin apunta a **PostgreSQL 13 y posteriores**. Es la versión contra la que se prueba y para la que está escrita cada comprobación.

También funciona contra servidores más antiguos, y la regla es **haz lo que la versión permita, no rechaces**. Una comprobación que pueda omitir una columna que el servidor no tiene, o saltarse una sección que necesita una vista inexistente, hace justo eso e imprime una nota breve de lo que omitió. Solo una comprobación cuyo tema *entero* es una función que no existe - la replicación lógica, que llegó completa en PostgreSQL 10 - muestra una tarjeta "Necesita PostgreSQL N" en lugar de ejecutarse. En concreto, en un servidor tan antiguo como **9.4**: la lista de sesiones, el live load y la vista de bloqueo funcionan - la vista de bloqueo recurre a un self-join de pg_locks donde falta `pg_blocking_pids()` (9.6), y la lista de sesiones muestra el indicador `waiting` donde faltan los eventos de espera (9.6). `Blocking pairs`, `Long transactions`, `Connection limits`, `Autovacuum activity`, `Replication`, `Table sizes`, `Replication slots`, las estimaciones de bloat, replica identity y el resumen de salud construyen una consulta acorde a la versión y anotan qué es más grueso. La particionalización declarativa es 10+, así que en un servidor más antiguo el árbol de esquema muestra los hijos de herencia como tablas bajo la tabla de la que heredan, nunca como particiones.

**PostgreSQL 10, 11 y 12** son totalmente conectables y la mayor parte de la aplicación funciona sin cambios; algunos recuadros del panel que leen vistas de estadísticas más nuevas quedan en blanco en silencio.

Por debajo de **PostgreSQL 10** estás más allá de lo probado. Conectar funciona, el editor SQL / explorador de esquemas / herramientas de copia están bien, y las comprobaciones que pueden adaptarse lo hacen; pero es best-effort, y el aviso al conectar lo dice. Por debajo de **9.4** los catálogos han divergido lo suficiente como para que DBApguin no afirme funcionar en absoluto.

Nunca ves un `column "..." does not exist` en crudo de una comprobación: cuando la causa es que el servidor es más antiguo de lo que la comprobación espera, la comprobación lo dice con palabras.

### PostgreSQL 19

DBApguin también está medido contra PostgreSQL 19: cada comprobación se
ejecuta en él, con y sin las extensiones contrib, como superusuario y como
rol normal. Una beta o una release candidate lo indica al conectar, porque los
detalles aún pueden cambiar antes de la versión final. Una versión mayor más
nueva que 19 conecta y funciona, con una nota de que es más nueva que todo lo
medido.

En 19 la aplicación usa lo que trajo la versión:

- **REPACK** en el panel de Mantenimiento y en los consejos de bloat. Reescribe
  una tabla y devuelve el espacio, y con CONCURRENTLY lo hace mientras siguen
  las lecturas y escrituras. Los servidores anteriores mantienen VACUUM FULL y
  CLUSTER.
- **La cola de autovacuum del propio servidor** en el estado de vacuum: qué
  tablas tomará autovacuum a continuación y por qué, incluida la prevención
  del wraparound.
- **Estadísticas de bloqueos** en Locks: esperas por tipo de bloqueo desde el
  último reinicio y cuántas veces la vía rápida se quedó sin huecos.
- **Cómo se inició un vacuum y con qué intensidad trabaja** (aggressive,
  failsafe, para evitar el wraparound), en la carga en vivo y en el asesor de
  vacuum.
- **Horas de reinicio por índice** en Índices sin uso, para que un índice
  cuyos contadores se acaban de reiniciar no se dé por no usado.
- **Ejecuciones con plan genérico y con plan personalizado** por sentencia en
  Top consultas.
- **EXPLAIN ANALYZE con E/S**: las peticiones que hizo un escaneo, sus esperas
  y cuánto leyó por adelantado.
- **Replicación**: errores y conflictos por suscripción, cuántas veces la
  decodificación lógica superó su memoria y el estado de recuperación de una
  réplica.
- **Slots lógicos con wal_level = replica**: 19 sube el nivel de WAL por sí
  solo, así que el diálogo de slots lo permite y dice lo que cuesta.
- **Desactivar restricciones**: el gestor de restricciones puede poner claves
  foráneas (desde 18) y checks (desde 19) en NOT ENFORCED en lugar de
  eliminarlas.
- **Consejos de ajustes según los valores por defecto de 19**: JIT
  desactivado, tabla de bloqueos dimensionada de otra forma, registro de
  esperas de bloqueo activado, avisos en inicios de sesión MD5, RADIUS fuera
  de pg_hba.conf.

Una copia de seguridad comprueba primero que el pg_dump de este equipo no sea
más antiguo que el servidor, el estado habitual de un equipo el día en que
sale una versión mayor nueva, e indica el paquete cliente que hay que
instalar. Antes de actualizar, ejecuta **¿Listo para PostgreSQL 19?** (ver [la
comprobación de preparación](#ready-for-postgresql-19)).

## 1.5 Requisitos en el servidor



DBApguin funciona con un clúster PostgreSQL corriente sin preparación especial. Algunas funciones avanzadas requieren extensiones o GUCs activados en el servidor. La forma más rápida de saber qué falta es el asesor de configuración integrado:

> Menú superior -> Estado del clúster -> «Asesor de configuración de DBApguin»

Sondea el clúster e imprime, por función, un estado (OK / PARCIAL / FALTA) más el SQL exacto o la línea de postgresql.conf que activa lo que falta. En nubes gestionadas (RDS, Aurora, Cloud SQL, Azure, Supabase, Neon) el consejo cambia al grupo de parámetros del proveedor, porque ALTER SYSTEM está bloqueado allí.

Referencia rápida, ordenada por impacto:

| Lo que quieres usar | Requisito en el servidor | Cómo activarlo |
|--|--|--|
| Consultas más costosas, historial de consultas, informes al estilo AWR | extensión pg_stat_statements cargada vía `shared_preload_libraries` | Añade `pg_stat_statements` a `shared_preload_libraries` + reinicio, luego `CREATE EXTENSION pg_stat_statements` en cada base que quieras medir |
| Captura de planes durante el trazado | extensión auto_explain + `log_min_duration` >= 0 | Añade `auto_explain` a `shared_preload_libraries` + reinicio, luego `SET auto_explain.log_min_duration = 500` (o 0 para todo) |
| Query_id en pg_stat_activity, correlación de ASH con pg_stat_statements | `compute_query_id = on` | `ALTER SYSTEM SET compute_query_id = on; SELECT pg_reload_conf();` (PG14+) |
| Tiempo de E/S realista en pg_stat_statements / EXPLAIN (BUFFERS) | `track_io_timing = on` | `ALTER SYSTEM SET track_io_timing = on; SELECT pg_reload_conf();` |
| Texto completo de la consulta en pg_stat_activity y ASH | `track_activity_query_size >= 4096` | `ALTER SYSTEM SET track_activity_query_size = 4096;` y luego REINICIAR |
| Top SQL completo en AWR (sin sentencias expulsadas entre instantáneas) | `pg_stat_statements.max` suficiente para las sentencias distintas | `ALTER SYSTEM SET pg_stat_statements.max = 10000;` y REINICIO - la comparación AWR dice cuándo se expulsaron sentencias |
| Estadísticas de llamadas a funciones | `track_functions = 'pl'` | `ALTER SYSTEM SET track_functions = 'pl'; SELECT pg_reload_conf();` |
| Instantánea de la caché en db-cache-advisor | extensión pg_buffercache | `CREATE EXTENSION pg_buffercache;` (superusuario) |
| Reescritura de tablas en línea (Mantenimiento -> pg_repack) | extensión pg_repack + binario cliente | Instala el paquete pg_repack en el servidor y en el equipo con DBApguin, luego `CREATE EXTENSION pg_repack;` |
| Cola del log del servidor en db-errors | `logging_collector = on`, `log_destination` incluye stderr, `pg_read_server_files` concedido | Solo servidores propios. Las nubes gestionadas bloquean pg_read_file. |
| Detalle de esperas por bloqueo en el log | `log_lock_waits = on`, `deadlock_timeout = '1s'` | `ALTER SYSTEM SET log_lock_waits = on;` |
| Log de consultas lentas | `log_min_duration_statement >= 0` | `ALTER SYSTEM SET log_min_duration_statement = '1s';` (o `'100ms'` más estricto) |
| Auditoría en el servidor (complementa la del cliente) | extensión pgaudit o `log_statement = 'ddl'` | Añade `pgaudit` a `shared_preload_libraries`, reinicia, `CREATE EXTENSION pgaudit;` |
| Que un no superusuario pueda ejecutar las comprobaciones | Concede `pg_monitor` al rol | `GRANT pg_monitor TO <rol>;` (PG10+) |

Si solo vas a activar una cosa, activa `pg_stat_statements`. La mayoría de las funciones de DBApguin se vuelven mucho más útiles en cuanto está cargado.

## 2. Primera configuración

En el primer arranque verás a la izquierda una lista de Servidores vacía.

1. Pulsa `+ Nueva` en la barra de Servidores.
2. Rellena:
   - Alias (nombre visible)
   - Grupo (por defecto «default»)
   - Color opcional (ayuda a distinguir producción de pruebas de un vistazo)
   - Host, Puerto, Usuario, Contraseña, Base de datos, sslmode
   - sslrootcert opcional (se sube desde Ajustes -> Certificados)
3. Pulsa `Guardar y conectar` o solo `Guardar`.

La conexión aparece en el árbol de Servidores. Un clic la abre. El árbol despliega el grupo padre y se desplaza hasta la fila.

Las contraseñas se guardan cifradas con una clave propia de cada instalación. El archivo de la clave está junto a `connections.json` en tu directorio de datos.

### Para qué sirve el certificado, y qué activa realmente TLS

Son dos ajustes distintos, y el certificado es el segundo, no el primero.

**`sslmode` es lo que activa el cifrado.** Está en la propia conexión:

| `sslmode` | Qué obtienes |
| --- | --- |
| `disable` | Sin cifrado |
| `prefer` (por defecto) | Cifrado si el servidor lo ofrece, texto plano si no - en silencio |
| `require` | El cifrado es obligatorio, pero no se verifica nada: se acepta cualquier certificado, incluido uno que un atacante acabe de generar |
| `verify-ca` | El certificado del servidor debe encadenar con una raíz en la que confías |
| `verify-full` | …y el nombre de host al que llamaste debe coincidir con el certificado |

En `require` es donde se para la mayoría, y es justo el ajuste que parece seguro
y no lo es: cifra el tráfico y luego se lo entrega a quien haya respondido.

**El certificado es la raíz contra la que comprueban `verify-ca` y
`verify-full`.** libpq busca una en `~/.postgresql/root.crt`. Si el certificado
de tu servidor lo emitió una CA pública que ya está ahí, no necesitas subir
nada - pones `sslmode` y listo. Subes un certificado cuando el emisor *no* es
ya de confianza: tu propia PKI interna, o el paquete de CA de un proveedor
cloud (AWS, Google y Azure publican el suyo).

Es decir:

1. Sube el PEM en **Ajustes → Certificados**. Se copia a
   `<DBCHECKS_HOME>/certs/` con permisos 0600.
2. Abre la conexión y pon **sslmode** en `verify-ca` o `verify-full`.
3. Elige el archivo subido en **Certificado raíz TLS** en esa misma conexión.

Subirlo sin el paso 2 no cambia nada: el archivo queda guardado, referenciado y
nunca consultado, porque con `prefer` o `require` no hay verificación que lo
consulte.

## 2.3 Aceptar el contrato de licencia

La primera vez que se ejecuta, DBApguin muestra el contrato de licencia de
usuario final y no se abre hasta que lo aceptas. No hay forma de saltarlo - sin
botón de cerrar, sin Esc, sin clic fuera - porque detrás todavía no hay nada que
usar.

El panel muestra el contrato en sí, no un resumen, leído del mismo archivo que
más tarde muestra Ayuda → Contrato de licencia. Lleva un **selector de idioma**,
y junto a los botones hay enlaces al aviso de privacidad y a la lista de
componentes de terceros.

**Rechazar y salir** no registra nada y cierra la ventana. Rechazar no instala ni
registra nada.

**Acepto** registra dos cosas en `app_prefs.json`, solo en tu equipo: qué versión
aceptaste y cuándo. No se transmite nada. La versión guardada es la razón por la
que se pregunta una vez y no en cada arranque - y por la que se volverá a
preguntar si el contrato cambia, porque aceptar una versión no es aceptar lo
que diga una posterior.

Para releerlo cuando quieras: **Ayuda → Contrato de licencia**.

## 2.4 El aviso de inicio

En la primera ejecución la aplicación muestra un aviso breve con el logo: qué
hace, y que sus vistas previas son una orientación y no una red de seguridad.
Está ahí porque esta herramienta envía `DROP` y `TRUNCATE` a producción cuando tú
se lo pides, y eso merece decirse una vez antes del primer uso.

El aviso también lleva un **selector de idioma**, porque el primer arranque es
cuando esa elección importa y en Ajustes todavía no mira nadie. Cambiarlo cambia
al instante el aviso que estás leyendo, que es la única manera de confirmar que
elegiste bien. El mismo ajuste vive después en **Ajustes → Idioma**.

Marca **No mostrar de nuevo** y deja de aparecer. Para recuperarlo:
**Ajustes → Aviso de inicio**.

Desactivarlo oculta el recordatorio. No cambia las condiciones a las que apunta:
esas están en Ayuda → Contrato de licencia.

## 2.5 Condiciones de uso

DBApguin es **gratuito para todos**, tanto para uso personal como comercial,
con todas las funciones.

Lo que eso significa en la práctica:

- **Todas las funciones están disponibles desde el primer arranque**, en tantos
  equipos como quieras: los asesores, el análisis de carga de trabajo, las
  instantáneas AWR, el historial de consultas, el análisis de índices y de
  hinchazón, las herramientas de comparación y migración: todo.
- **No se comprueba nada a través de la red**, así que nada llama a casa.
  La aplicación
  solo habla con los servidores de base de datos que tú configures y, si
  instalas plugins, con aquello para lo que esos plugins estén configurados
  (capítulo 7, Plugins).
- **Sin cuenta, sin inicio de sesión, sin registro.**

Lo único que se te pide es aceptar el acuerdo de licencia en el primer arranque
(sección 2.3).

En resumen: úsalo libremente, no lo vendas y no lo presentes como obra propia.
Gratuito no es lo mismo que código abierto: el código fuente no se publica. El texto completo está en **Ayuda -> Acuerdo de licencia**.

DBApguin ha sido creado por Darko Lovrenčec.

## 3. Barra lateral de Servidores

El panel izquierdo tiene tres secciones:

### 3.1 Servidores (arriba)

Lista cada conexión guardada, agrupada por grupo. Botones de la barra:

- `+ Nueva`: registrar una conexión
- `Editar`: editar la conexión activa (alias, grupo, color, host, contraseña)
- `Contraer todo`: contraer cada grupo
- `Expandir todo`: expandir cada grupo
- `Más` (...): abrir el gestor completo de conexiones (importar, exportar, reorganizar en masa)

Menú contextual por conexión:
- Conectar / Ir a la pestaña
- Editar
- Refrescar conexión (volver a introducir la contraseña, reconectar)
- Fijar / Soltar pestaña
- Desconectar
- Abrir editor SQL / psql / Sesiones / Panel / Mantenimiento / Servidor (solo en la conexión activa)
- Submenú de comprobaciones del clúster
- Crear base de datos / tabla / rol
- Probar conexión
- Duplicar
- Eliminar (aviso con «Deshacer» durante 6 segundos)

Puntos de estado junto a cada conexión:
- Verde relleno: es la pestaña activa
- Verde hueco: pestaña abierta, pero estás mirando otra
- X roja: conexión perdida (el servidor la cerró)
- Sin punto: no conectado

### 3.2 Bases de datos (visible solo al estar conectado)

Lista cada base del clúster. Un clic cambia la base activa. Campo de filtro arriba.

**Clic derecho en el encabezado BASES DE DATOS** muestra el menú **Database** de la barra, entrada por entrada - conexiones, trabajos en segundo plano, comparación, copia y restauración, crear, clonar, renombrar, privilegios y eliminar, extensiones - seguido de `Refrescar la lista de bases`. Crear, clonar, renombrar y eliminar aparecen en gris, con el motivo, cuando tu rol o el proveedor no los permiten.

**Clic derecho en una fila** para todo lo que afecta a esa base: editor SQL,
sesiones, panel, mantenimiento, todo el catálogo de comprobaciones, creación de
objetos, copia y restauración, clonar, renombrar, privilegios y eliminar.

#### Extensiones

`Extensiones` abre una de tres listas, y la que sueles querer es la tercera:

- `Extensiones instaladas…` - lo que esta base ya ha creado
- `Extensiones disponibles (no instaladas)…` - lo que trae el **servidor** y para
  lo que nadie ha ejecutado aún `CREATE EXTENSION`
- `Gestionar extensiones…` - todo, con filtro y contador

Todo lo que trae el servidor aparece aquí, esté creado o no. Desde la misma lista
la instalas, cambias su versión, la mueves a otro esquema o la eliminas. Las
extensiones que el servidor no tiene no pueden aparecer: primero hay que
instalarlas en la máquina, normalmente con un paquete.

### 3.2.5 Roles y usuarios

Está junto a Bases de datos y no dentro de una, porque los roles son de todo el
clúster: no pertenecen a una base, y anidarlos bajo una diría lo contrario.

**Clic derecho en el encabezado ROLES Y USUARIOS**:

- `+ Crear rol / usuario…`
- `Resumen de roles y usuarios` - la auditoría completa
- `Comparar roles / usuarios…` entre dos servidores
- `Exportar TODOS los roles como .sql…`
- `Refrescar roles`

**Clic en un rol** para auditar sus permisos. **Clic derecho en un rol** para sus
acciones: privilegios, `search_path`, pertenencias, crear uno similar, copiar
privilegios, exportar DDL, eliminar.

Los usuarios con login muestran 👤, los roles de grupo 👥.

#### Exportar un rol {#role-export}

**Export DDL** (un rol) y **Export ALL roles** escriben un script que crea el
rol en otro lugar: `CREATE ROLE` con cada atributo (dentro de un bloque que
omite un rol que ya existe), su comentario, sus membresías, sus ajustes
`ALTER ROLE ... SET` - también por base de datos - y, marcado por defecto,
**cada privilegio que tiene en el clúster**: sobre bases de datos,
tablespaces y parámetros del servidor (PostgreSQL 15+), y en cada base sobre
esquemas, tablas y sus columnas, secuencias, funciones y procedimientos,
tipos y dominios, lenguajes, foreign-data wrappers y servidores y objetos
grandes, además de los privilegios por defecto que da o recibe (`ALTER
DEFAULT PRIVILEGES`). Cada base se lee por separado y su parte empieza con
`\connect`, así que el archivo es un script de psql: `psql -f role.sql`
contra el destino, como superusuario. Una base que la exportación no pudo
leer - una a la que el rol con el que estás conectado no puede conectarse -
se nombra al final del archivo y en un aviso, así que una parte que falta
nunca pasa en silencio. **Lo que posee** añade `ALTER ... OWNER TO` para sus
esquemas, tablas, vistas, secuencias y rutinas, para cuando esos objetos ya
están en el destino. Los hashes de contraseña son una opción aparte. Medido:
un rol con privilegios de todo tipo en dos bases, exportado, eliminado con
`DROP OWNED` y `DROP ROLE`, y el archivo ejecutado con psql - cada privilegio
y cada propiedad volvió como estaba.

### 3.3 Explorador de esquema (visible solo al estar conectado)

El explorador de esquema forma parte del árbol **Servidores**. Bajo la conexión en la que estás, **Bases de datos** lista todas las bases del servidor con un recuento. Al desplegar una se abre en sus esquemas, cada uno con el número de objetos que contiene, y cada esquema en sus objetos, agrupados por tipo.

Cada esquema muestra los mismos 40 tipos: tablas, tablas particionadas, vistas, vistas materializadas, tablas foráneas, secuencias, índices, funciones, procedimientos, agregados, funciones de ventana, disparadores, disparadores de eventos, reglas, políticas RLS, tipos, dominios, tipos de rango, tipos multirrango, extensiones, lenguajes, operadores, clases de operadores, familias de operadores, conversiones de tipo (casts), intercalaciones, conversiones de codificación, configuraciones, diccionarios, analizadores y plantillas de búsqueda de texto, envoltorios de datos foráneos, servidores foráneos, mapeos de usuario, publicaciones, suscripciones, estadísticas extendidas, privilegios predeterminados, objetos grandes y tablespaces. Un tipo sin nada dentro sigue en la lista, en gris, para que "no hay ninguno" nunca parezca "no se muestra". Los objetos que no pertenecen a ningún esquema - disparadores de eventos, lenguajes, casts, envoltorios de datos foráneos, servidores foráneos, mapeos de usuario, publicaciones, suscripciones, privilegios predeterminados sin IN SCHEMA, objetos grandes y tablespaces - aparecen bajo **(cluster)**. Los esquemas de sistema `pg_catalog` e `information_schema` muestran sus tablas, vistas, índices, rutinas y tipos; los operadores, casts y objetos similares que trae PostgreSQL quedan fuera del árbol.

Las relaciones se abren como PostgreSQL las guarda. Una tabla particionada se abre en sus particiones; una partición que a su vez está particionada queda en su sitio con sus propias particiones debajo, a cualquier profundidad. Una tabla se abre en las tablas que heredan de ella. Cada una aparece una vez, bajo su padre, no otra vez entre las tablas. Una partición que vive en otro esquema que su padre se muestra bajo el padre con su esquema, y en su propio esquema arriba, con el padre nombrado. Cada tabla, vista y vista materializada se abre además en sus partes - índices, restricciones, disparadores, reglas, políticas RLS, las secuencias de sus columnas, objetos de estadísticas y las publicaciones en las que está -, leídas al abrirla, así que un esquema de miles de tablas no carga más lento por ellas. Lo que PostgreSQL copió de una tabla particionada a una partición (un índice, una clave, un disparador de fila) indica allí *de* qué original viene y no está en las listas de índices y disparadores del esquema, que muestran lo declarado; esa copia no se puede eliminar por sí sola. Una secuencia que pertenece a una columna lo dice (*pertenece a* o *identity*).

Los objetos que pertenecen a una extensión quedan fuera de cada tipo - la extensión misma sigue en Extensiones - y el esquema dice cuántos dejó fuera. Un clic en esa línea los muestra, cada uno marcado con su extensión; otro clic los oculta.

Un clic en un esquema muestra sus tablas con filas vivas y muertas, tamaños, número de índices, escaneos y el último vacuum y analyze; el filtro encima de esa lista la acota por nombre. Un clic en un objeto lo abre en su propia pestaña (para una tabla: DDL, Relaciones, Almacenamiento, Asesor, Datos). Clic derecho para todo lo demás - ver [Menús de objeto](#object-menus). El cuadro de búsqueda encima del árbol encuentra conexiones por alias, host, usuario o base; no busca dentro del árbol.

## 4. Pestañas de trabajo

Cada pestaña lleva su propia conexión, su posición en el esquema y el estado del editor SQL. Las pestañas sobreviven a los reinicios de la aplicación.

- `+ Nueva conexión` (barra superior izquierda): abre una pestaña sin conexión.
- Clic derecho en una pestaña: Cambiar / Refrescar conexión / Fijar / Guardar como conexión (para pestañas sin guardar) / Desconectar / Cerrar.
- Las pestañas fijadas sobreviven a «Cerrar todo».

#### Crear una conexión

El diálogo Nueva conexión contiene los datos de conexión y, debajo, el nombre,
el grupo y la etiqueta de color. El desplegable de grupos lista los que ya
existen y ofrece **Crear grupo nuevo** y **Crear subgrupo**: una conexión puede
archivarse donde corresponde mientras se crea, en vez de caer en `default` y
tener que moverla después.

Tres botones cierran el diálogo, y los tres hacen lo mismo con la conexión:

| | |
|---|---|
| **Probar** | comprueba las credenciales y se queda donde está |
| **Conectar** | la abre en una pestaña de trabajo sin guardarla |
| **Guardar y conectar** | escribe la conexión guardada y luego la abre |

«La abre» significa lo mismo en ambos casos: una pestaña de trabajo propia salvo
que la actual siga vacía, etiquetada con la conexión guardada para que la barra
de pestañas de arriba la muestre, marcada como activa en la barra lateral, y
cada pestaña de herramienta abierta después pertenece a esa conexión. «Guardar y
conectar» antes solo escribía la fila y se detenía ahí, dejando que la pestaña
que estuviera delante mostrara una conexión que no sabía que tenía.

Cuando hay más pestañas de las que caben, aparecen flechas en ambos extremos y
un **deslizador fino** bajo las pestañas. El deslizador solo aparece cuando hay
adónde desplazarse; arrástralo o pulsa en la parte vacía para pasar página. Tiene
su propio carril, así que nunca tapa una pestaña ni su botón de cerrar.

#### Rol, servicio y túnel SSH {#conn-role-service-ssh}

Tres ajustes del diálogo de conexión que la mayoría de las conexiones dejan
vacíos, con los nombres que usa pgAdmin.

- **Rol** - la sesión ejecuta `SET ROLE` a él justo después de cada conexión, en
  cada conexión que la aplicación abre para este servidor: las del pool, una
  reconexión, las comprobaciones, los recolectores en segundo plano, una
  transacción SQL mantenida. Todo corre entonces como ese rol, sus privilegios
  deciden lo que la aplicación ofrece y la cabecera lo nombra. El usuario de
  inicio de sesión debe ser miembro del rol; si el servidor lo rechaza, la
  conexión falla con el nombre del rol y el motivo del servidor, y nada corre
  como el usuario de inicio de sesión en su lugar. Un rol elegido en la lista de
  roles de la cabecera sigue mandando mientras esté elegido.
- **Servicio** - un nombre de servicio de libpq. Host, puerto, base de datos,
  usuario y contraseña que deje vacíos en el diálogo salen de ese servicio, y lo
  que sí rellene tiene prioridad. libpq busca el servicio en el archivo indicado
  en `PGSERVICEFILE`, si no en `.pg_service.conf` en su directorio personal (en
  Windows `%APPDATA%\postgresql\.pg_service.conf`), y después en
  `pg_service.conf` en el directorio indicado en `PGSYSCONFDIR`.
- **Avanzado: túnel SSH** (plegado hasta que una conexión lo usa) - para una base
  de datos accesible solo a través de un host de salto. Indique el host SSH, el
  puerto (22 si está vacío), el usuario y, si hace falta, el archivo de clave. La
  aplicación arranca el programa `ssh` de su equipo con un reenvío de puerto
  local, así que se aplican su configuración de ssh, `known_hosts` y
  `ssh-agent`. Inicia sesión solo con clave y nunca pide ni guarda una
  contraseña SSH; una clave con frase de paso debe estar cargada en `ssh-agent`.
  El host del diálogo es entonces el de la base de datos tal como lo ve el host
  de salto, a menudo `localhost`. Un host de salto cuya clave aún no ha aceptado
  se rechaza con el comando que debe ejecutar una vez en un terminal para
  comprobarla y aceptarla. El túnel se cierra con la última pestaña que lo usa y
  al salir de la aplicación.

## 5. Vistas principales

La barra de pestañas superior cambia de vista. Cada vista trabaja contra la conexión activa de la pestaña de trabajo activa.

En cualquier lista desplegable se puede buscar. Haga clic en ella, o simplemente empiece a escribir mientras tiene el foco, y se abre un campo de búsqueda encima: la lista se reduce con cada tecla, encuentra cualquier parte de una entrada sin distinguir mayúsculas ni acentos ("sifra" encuentra "šifra"), y cada palabra escrita tiene que aparecer. Las flechas arriba y abajo mueven, Intro elige, Escape cierra y deja la elección como estaba, y Tab toma la entrada resaltada y sigue. Las listas de solo dos o tres entradas abren la lista del sistema, donde una letra escrita salta a la primera entrada que empieza por ella.

### 5.1 Comprobaciones

La vista de entrada. La barra de menú superior expone el catálogo por grupos:

- Estado del clúster: db-health, db-locks, db-blocking, db-long-tx, db-idle-in-tx, db-conn-limits, db-vacuum, db-autovacuum, db-checkpoints, db-replication, db-repl-slots, db-logical-replication
- Conexiones y consultas: db-connections, db-users, db-active-queries, db-long-queries, db-stat-statements, db-query-history, db-temp-queries, db-temp-files
- Almacenamiento y rendimiento: db-sizes, db-schema-sizes, db-table-sizes, db-index-sizes, db-bloat, db-bloat-indexes, db-bloat-breakdown, db-pgstattuple, db-buffer-cache, db-tablespaces
- Ajuste y diagnóstico: db-settings, db-cache-hit, db-cache-advisor, db-index-usage, db-index-unused, db-index-advisor, db-duplicate-indexes, db-table-stats, db-replica-identity, db-workload-report, db-ash, db-awr-snapshot, db-trace-session, db-errors
- Seguridad / permisos: db-roles-list, db-role-perms (elegir objetivo), db-ownership (elegir ámbito)
- También accesible: db-sequences, db-seq-usage, db-seq-sync (herramientas de secuencias en el navegador de esquema), db-setup-advisor (menú Ayuda)

**Tres informes responden a «adónde ha ido el espacio», en tres niveles.**
`db-sizes` da cada base de datos del clúster; `db-schema-sizes` desglosa la
base **conectada** por esquemas - total, con los datos y los índices por
separado, la parte que ocupa cada esquema, el número de objetos y su tabla
mayor; `db-table-sizes` baja hasta la tabla concreta; `db-index-sizes` baja
hasta el índice concreto. Cómo leer los cuatro, y qué incluye y qué no cada
total, está en *Adónde va el espacio* más abajo.

Tamaños de esquema cubre sólo la base a la que está conectado, y eso es un
límite, no un descuido: `pg_namespace` es por base de datos, así que listar
los esquemas de todas exigiría una conexión a cada una. El total abarca todo
lo que en el esquema ocupa disco - tablas, su TOAST, sus índices, vistas
materializadas y secuencias - cada byte una sola vez.

Cada comprobación abre su propia pestaña de resultado. Las pestañas sobreviven al cambio de vista, así que puedes comparar varias en paralelo. Cada pestaña tiene dos paneles:

- Estructurado: tablas ricas, tarjetas de asesor, detalles
- Salida cruda: informe de texto simple

Muchas comprobaciones incluyen secciones de «asesor» con hallazgos marcados por gravedad (OK / INFO / AVISO / CRIT) y sugerencias concretas del siguiente paso.

#### Sumar lo que muestra un informe

Cada tabla de informe se ordena por columna y se filtra escribiendo. Dos cosas
más responden a la pregunta por la que suelen abrirse estos informes: «¿merece
la pena?».

**Un total bajo la tabla.** Toda columna cuyos valores se lean como tamaños o
como números simples se suma, y la suma sigue a cada filtro. Reduzca un informe
de bloat a un esquema y el total dirá cuánto espacio devolvería reindexarlo;
redúzcalo a una tabla y verá si conviene el trabajo sobre todo el esquema o si
casi todo lo carga una sola tabla.

Los números se leen de vuelta desde lo impreso, porque en la mayoría de estas
comprobaciones eso es todo lo que hay: el propio formato de tamaños de
PostgreSQL y nada debajo. La suma es por tanto una suma de los valores tal como
se muestran, redondeados a la unidad impresa: cuarenta índices redondeados al
megabyte pueden desviarse decenas de megabytes. Suficiente para decidir, no una
cifra para citar.

Los porcentajes, las proporciones y las medias nunca se suman: su suma es un
número que no significa nada, e imprimirlo invitaría a leerlo. Lo dice el
nombre de la columna: un informe de bloat imprime `bloat_pct` como un simple
`44,9`, sin ningún signo en la celda.

La cuadrícula de resultados del editor SQL y la pestaña Datos del explorador de
esquemas suman solo tamaños. Un informe sabe cuáles son sus columnas; un
`SELECT` que acaba de escribir no lo sabe, y una suma bajo una columna de
identificadores es ruido.

**Elegir valores de una columna.** La flecha junto al filtro de cada columna
abre la lista de valores que existen realmente en esa columna, con su recuento.
Marcar tres esquemas cuyos nombres no tienen nada en común es algo que un
filtro por subcadena no puede hacer, y los informes de almacenamiento están
llenos de ello. El filtro escrito y la lista marcada se aplican los dos.

Una columna con más de doscientos valores distintos lo dice en vez de dibujar
mil casillas.

#### De qué base de datos habla un informe

Cada uno de estos informes se refiere a una base de datos, y la línea sobre los
resultados dice a cuál: el nombre de la comprobación, la base y la conexión en
la que se ejecutó. Viene de la pestaña, no de la conexión viva, así que una
pestaña que se ejecutó contra `shop` sigue diciendo `shop` aunque se cambie la
conexión por debajo.

#### Un índice, y qué hacer con él

Los informes nombran un índice inflado o sin uso. Al pulsar el nombre se abre
el **asesor de índices** para ese índice concreto, y lo mismo ocurre en cada
índice de la lista del explorador de esquemas.

Responde a la pregunta que el informe deja abierta. La estimación de bloat es
la misma aritmética que la del informe desde el que ha llegado, acotada a ese
índice, de modo que los dos no pueden discrepar. Junto a ella: cuántas veces se
ha escaneado desde el último reinicio de estadísticas, su tamaño frente al de
su tabla, y una comprobación contra todos los demás índices de la misma tabla:
otro índice sobre exactamente las mismas columnas con el mismo método, o uno
cuyas columnas iniciales son la lista completa de este, en cuyo caso el
planificador puede usar el más ancho para todo lo que este responde.

Los índices que respaldan restricciones se leen de otra manera, y por eso
«nunca escaneado» no es por sí solo un motivo para eliminar nada. Un índice que
respalda una PRIMARY KEY, un UNIQUE o un EXCLUDE impone la restricción lea o no
lea una consulta, y `DROP INDEX` lo rechaza; lo mismo vale para la identidad de
réplica de una tabla, que la replicación lógica necesita. El asesor lo dice y
no ofrece la eliminación.

Lo que ofrece ejecutar está escrito para un índice y ninguno más:

| | |
|---|---|
| `REINDEX INDEX CONCURRENTLY` | construye un reemplazo al lado y los intercambia: sin bloqueo largo, necesita espacio para ambas copias, no puede ir en una transacción (PostgreSQL 12+) |
| `REINDEX INDEX` | mantiene un bloqueo ACCESS EXCLUSIVE sobre la tabla durante toda la reconstrucción |
| `DROP INDEX CONCURRENTLY` | se ofrece solo donde el índice se puede eliminar |
| la definición | consérvela antes de eliminar: es la única copia |

Cada uno abre el mismo diálogo en el que abre toda sentencia generada por esta
aplicación: verla, copiarla, guardarla o abrirla en el editor SQL. Las dos
reconstrucciones ofrecen además **Ejecutar como trabajo en segundo plano**, que
las coloca en Trabajos en segundo plano con su salida en directo, de modo que
una reconstrucción de veinte minutos no depende de que una ventana siga
abierta.

Antes de esto, el único REINDEX que la aplicación podía emitir era por tabla,
por esquema o por base de datos, así que un informe que nombraba un índice
inflado solo podía atenderse reconstruyendo todos los índices de su tabla.

#### Comprobaciones destacadas {#notable-checks}

**db-health** (primera parada recomendada): revisión del pulso en una sola pantalla - conexiones, idle-in-tx, transacciones activas largas, bloqueos, acierto de caché, retraso de replicación, archivado de WAL, bloat, vacuum pendiente, secuencias, índices sin usar. El detalle por categoría saca las filas problemáticas.

**db-cache-advisor**: tasas de acierto de heap/índices/toast, tablas, índices y TOAST calientes con remedio concreto, comprobación del tamaño de shared_buffers frente al clúster, sensatez de effective_cache_size, instantánea de pg_buffercache si la extensión está instalada.

**db-index-advisor**: análisis de seq scans con columnas candidatas por tabla. Cruza con pg_stat_statements para las consultas calientes, señala huecos en la columna principal, oportunidades de índices parciales y la cobertura de FK compuestas. Genera sentencias CREATE INDEX listas para copiar.

**db-vacuum**: estado de vacuum/analyze por tabla con asesor de autovacuum. Muestra filas vivas y muertas, mod_pct, etiqueta de calor (muy caliente / caliente / media / por defecto), marcas (riesgo de WRAPAROUND, autovacuum desactivado, solo inserciones) y las reloptions «recomendadas» frente a las «actuales» por candidata. Desde 1.0.x de octubre de 2026 también mira en la otra dirección: con qué frecuencia se limpia cada tabla (ejecuciones al día y el intervalo entre ellas, con su duración en PostgreSQL 18), una tabla limpiada cada hora o más a menudo - y si eso está bien (ver [cuándo está bien limpiar a menudo](#vacuum-bloat-and-table-statistics)), una limpiada una y otra vez sin efecto porque algo sostiene un snapshot más antiguo que sus filas muertas (nombrado, con lo que hay que terminar), una tabla cuyo vacuum es siempre agresivo y - en la sección *Autovacuum cost and I/O* - los ajustes de ritmo del autovacuum y, en PostgreSQL 16 y posterior, su parte de las lecturas y escrituras del clúster según `pg_stat_io`. Una recomendación nunca endurece los umbrales de una tabla que ya se limpia cada hora. También dice cuándo vacuum no puede llegar a una tabla: una sesión tiene un bloqueo que VACUUM no puede compartir (SHARE UPDATE EXCLUSIVE o más fuerte - un ANALYZE largo, CREATE INDEX CONCURRENTLY, ALTER TABLE, LOCK TABLE) en una transacción abierta un minuto o más, sobre una tabla que ha pasado su disparador. Un autovacuum normal se salta esa tabla y lo vuelve a intentar un naptime después, así que nada de lo que se dice de ella ocurre; el hallazgo nombra la sesión (pid, usuario, aplicación, cliente, cuánto lleva abierta su transacción, su sentencia) y va primero. Un escritor inactivo en una transacción READ COMMITTED también cuenta como poseedor del snapshot: entre sentencias no tiene snapshot, pero su ID de transacción retiene el horizonte.

**db-stat-statements**: las 25 consultas más costosas con indicadores por fila (inestabilidad del plan, volcado de work_mem con bytes concretos por llamada, acierto de caché bajo, lenta en cada llamada, camino del peor caso). El widget de consultas principales permite copiar el texto completo.

**db-replica-identity**: tablas sin PRIMARY KEY y tablas con REPLICA IDENTITY no predeterminada. Cruza con pg_publication_tables para distinguir CRÍTICO (está en una publicación, fallará en UPDATE/DELETE) de meramente informativo. Recomienda ALTER en este orden: ADD PRIMARY KEY, REPLICA IDENTITY USING INDEX, REPLICA IDENTITY FULL.

**db-errors**: vigilante de errores consciente de la nube. Siempre muestrea los contadores de pg_stat_database en una ventana de 5 segundos (rollbacks, interbloqueos, conflictos, sessions_fatal/killed/abandoned en PG14+). En clústeres propios además lee los últimos 64 KB del log del servidor mediante `pg_current_logfile()` + `pg_read_file()` y analiza las líneas ERROR/FATAL/PANIC. En nubes gestionadas se omite la lectura del log, con indicaciones propias del proveedor (consola de RDS, log_fdw, Cloud Logging, Azure Server Logs, Supabase / Neon / Heroku).

**db-roles-list**: listado de roles y pertenencias más un barrido de seguridad: superusuario + login, BYPASSRLS sin superusuario, CREATEROLE en PG <16, login sin límite de conexiones y sin VALID UNTIL, roles de login caducados, hashes MD5 de contraseñas, REPLICATION + LOGIN, PUBLIC con CONNECT sobre bases, PUBLIC con CREATE sobre esquemas, rutinas SECURITY DEFINER propiedad de un superusuario con ejecución para PUBLIC.

**db-role-perms** (elegir rol objetivo): 19 secciones (resumen, pertenencias, privilegios sobre base/esquema/tabla/columna/secuencia/función/tipo/fdw/servidor/lenguaje, privilegios por defecto, RLS) con una columna de interpretación por fila («vía PUBLIC», «WITH GRANT OPTION», «sin USAGE en el esquema X: la concesión no surte efecto», etc.) y un bloque de hallazgos que cruza secciones (superusuario efectivo por cadena, herencia de BYPASSRLS / CREATEROLE / REPLICATION, permisos de tabla sin USAGE de esquema, etc.).

**db-ownership** (elegir ámbito): listados de esquemas / tablas / secuencias / funciones / tipos / FDW / lenguajes / publicaciones / suscripciones más hallazgos (objetos de la aplicación propiedad de un superusuario, rutinas SECURITY DEFINER propiedad de un superusuario, OID de propietario colgando tras un DROP ROLE sin REASSIGN, esquema public con propietario que no es superusuario, esquemas con tres o más propietarios distintos).

**db-settings**: guía curada de unos 70 parámetros de pg_settings relevantes para el ajuste, con la razón del DBA, una regla práctica y distintivos de estado (aviso / peligro). Editable desde la vista Servidor.

**db-setup-advisor**: sondea cada ajuste del servidor del que dependen las funciones de DBApguin (extensiones, GUCs, permisos de rol), informa OK / PARCIAL / FALTA por función e imprime el SQL exacto o la línea del grupo de parámetros que cierra cada hueco. Es consciente del proveedor: en nubes gestionadas el consejo dice «defínelo en el grupo de parámetros» en lugar de sugerir ALTER SYSTEM. Ejecútalo primero al configurar DBApguin contra un clúster nuevo, y de nuevo tras cualquier cambio en postgresql.conf para comprobar.

La lista de arriba dice qué produce cada comprobación. Abajo está cómo leer
las que necesitan interpretación, y los errores que existen para atrapar.

#### El resumen de salud {#the-health-overview}

`db-health` es una sola pantalla: una línea por categoría - conexiones, idle-in-transaction, transacciones largas, bloqueos, acierto de caché, retraso de replicación, archivado de WAL, retraso de vacuum, espacio desperdiciado, atraso de vacuum, secuencias cerca de su límite, índices sin uso, durabilidad, particiones - cada una verde, ámbar o roja. Ejecútala primero; a menudo es la única comprobación que necesitas. Un hallazgo que trata de una tabla la nombra en la columna **table** - la peor cuando cuenta varias - y ese nombre es un enlace: abre la tabla en el explorador de esquemas.

Ámbar significa «merece un vistazo», rojo significa «la producción está afectada ahora». Pulsa una categoría para abrir sus filas: los bloqueos se despliegan en pares bloqueador/en espera, idle-in-transaction en las sesiones y cuánto lleva cada una fijando su snapshot. Una línea roja sin nada en el desglose la disparó un total, no una fila - por ejemplo un acierto de caché arrastrado hacia abajo en toda la base. La trampa es leer el ámbar como ruido: un espacio desperdiciado ámbar es un VACUUM que puedes programar para esta noche; la misma tabla roja el mes que viene es `pg_repack` en un sistema ocupado.

**El retraso de vacuum y el espacio desperdiciado son dos filas separadas porque miden dos cosas distintas.** *VACUUM LAG (dead tuple %)* es `n_dead_tup / (n_live_tup + n_dead_tup)` - la fracción de filas de una tabla que están muertas. Dice que el autovacuum va por detrás. **No** es una medida de espacio: una tabla puede estar al 4% muerta y aun así ocupar un terabyte de espacio libre que nunca se devolvió. *WASTED SPACE (est.)* es ese terabyte - bytes estimados que la tabla retiene pero no usa, heap más TOAST más cada índice, la misma estimación combinada que usan `db-bloat-breakdown` y la pestaña Advisor. Una tabla aparece como `4.3% dead tuples but 1.3 TB wasted (39x its live data)` cuando ambas cosas son ciertas a la vez. La fila se activa con cualquiera de las dos medidas: 30 % de filas muertas (CRIT al 50 %) o, sea cual sea la proporción, 1.000.000 de filas muertas en una tabla (CRIT a 10.000.000) - el 29 % de una tabla de mil millones de filas sigue siendo un vacuum muy atrasado.

**WAL ARCHIVING** dice si el archivador está enviando WAL ahora mismo, a partir de `pg_stat_archiver` y los ajustes de archivado - la misma lectura que hacen la alarma de la cabecera y las tarjetas de archivado de Key settings. *Fallando* (rojo) significa que el último intento falló y no se ha archivado nada desde entonces: la fila nombra el segmento que falló y cuándo, el último archivado y cuántos segmentos esperan en `pg_wal`. `archive_mode` activado sin `archive_command` ni `archive_library` es ámbar - no se archiva nada y cada segmento terminado se queda en `pg_wal` - y rojo cuando esperan 10 GB. `archive_command` y `archive_library` definidos a la vez es rojo: desde PostgreSQL 16 el archivador se niega a funcionar, mientras sus estadísticas parecen no cambiar. El detalle muestra el comando tal como lo tiene el servidor y qué hacer: leer su propio error en el log del servidor (`archive command failed with exit code`), ejecutarlo a mano para el segmento indicado y nunca borrar archivos de `pg_wal` - en cuanto el comando funcione, el archivador se pone al día solo. Contar los segmentos en espera requiere PostgreSQL 12 y superusuario o `pg_monitor`; sin ellos la cifra no aparece.

**DURABILIDAD** lee los ajustes de los que depende que una transacción confirmada sobreviva a una caída. Es roja con `fsync = off` o `full_page_writes = off` - los dos ajustes con los que una caída del sistema operativo o un corte de luz pueden corromper el propio clúster, no solo perder sus últimas transacciones - y con un `synchronous_standby_names` que pide más standbys síncronos de los que hay conectados mientras los commits los esperan (`synchronous_commit` en `on`, `remote_write` o `remote_apply`): cada COMMIT se cuelga entonces, y la fila dice cuántas sesiones están esperando ahora (evento de espera `SyncRep`) y desde hace cuánto. Con el `synchronous_commit` del servidor en `local` u `off`, el mismo standby ausente es ámbar: los commits no esperan ahora, pero una sesión que lo vuelva a poner en `on` sí. `synchronous_commit = off` y las sumas de verificación de datos desactivadas son azules (INFO): lo primero puede perder las últimas transacciones ya notificadas como confirmadas (hasta tres `wal_writer_delay`) pero no corrompe nada, un intercambio que a menudo se hace a propósito; lo segundo significa que una página dañada por el disco se lee sin error - activar las sumas de verificación exige el clúster parado (`pg_checksums --enable`, PostgreSQL 12+). Las tarjetas de Ajustes clave de esos mismos cinco ajustes dan el mismo veredicto. En un standby la parte del standby síncrono se omite: solo se aplica a un primario.

**PARTICIONES** es el margen de las tablas particionadas por rango de la base conectada: ámbar cuando un último rango termina en menos de 7 días o la partición DEFAULT de una tabla RANGE contiene filas, roja cuando el último rango ya terminó - consulta [Margen de particiones](#partition-runway).

La fila **MIGRACIONES** aparece cuando la base de datos tiene la tabla de historial de una herramienta de migraciones: roja para una migración fallida, ámbar o roja para un bloqueo que dejó su proceso (ver *Historial de migraciones*).

#### Conexiones, sesiones y carga {#connections-sessions-and-load}

Estas leen `pg_stat_activity` y `pg_stat_statements` desde distintos ángulos.

`db-connections` es el recuento por estado - active, idle, idle in transaction y waiting. **Idle in transaction** es el número a vigilar: esas sesiones mantienen una transacción abierta, fijan el snapshot más antiguo que vacuum puede quitar y a menudo retienen bloqueos, y un recuento en aumento suele ser una aplicación que olvidó el commit o un pool sin timeout de idle-in-transaction. `db-users` desglosa las mismas conexiones por rol y aplicación, que es como se encuentra *qué* servicio las filtra.

`db-conn-limits` es `max_connections` frente a lo usado, con el desglose por estado y el margen de `superuser_reserved_connections`. Solo los backends de cliente cuentan contra el límite - los workers de fondo no -, así que el número aquí puede ser menor que un recuento en bruto de `pg_stat_activity`. La alarma de conexiones de la cabecera cuenta igual. **Waiting** es una sentencia activa retenida por algo que no es su cliente - un bloqueo, un LWLock, E/S; una sesión inactiva también muestra un wait event (`Client: ClientRead`, espera su siguiente orden) y no se cuenta. Cuando está cerca del techo, la solución casi nunca es un `max_connections` mayor (cada slot cuesta memoria); es un pooler de conexiones, o encontrar las sesiones idle-in-transaction de arriba.

`db-active-queries` es lo que se está ejecutando ahora mismo, las más largas primero, con el PID del backend y el evento de espera. `db-long-queries` es la misma lista filtrada a las sentencias que llevan más de cinco segundos - las que vale la pena interrumpir o explicar. Ambas son una instantánea: una consulta que terminó entre dos refrescos ya no está.

`db-stat-statements` es la vista acumulada - las sentencias principales por tiempo total desde el último `pg_stat_statements_reset()`, cada una normalizada de modo que `WHERE id = 1` y `WHERE id = 2` son una fila. Las banderas de aviso por fila señalan inestabilidad del plan, vuelco de `work_mem` (con los bytes por llamada), un ratio de acierto de caché bajo y sentencias lentas en cada llamada individual, no de media. El tiempo total es la columna por la que ordenar: una consulta que tarda 3 ms un millón de veces al día es un coste mayor que una que tarda 20 segundos dos veces. El widget copia el texto completo de la sentencia. Consulta `db-query-history` (sección 6.6) para los mismos datos cruzados con un reloj.

Tras las sentencias principales viene **Bloqueo optimista - escrituras obsoletas**: cada UPDATE y DELETE de la vista cuyo WHERE compara una columna de versión con un parámetro - el `@Version` de Hibernate, el `lock_version` de Rails, el `xmin` o `ConcurrencyStamp` de Entity Framework, un `version = $n` escrito a mano, una comprobación de `updated_at` - con `stale_%` = 1 - rows / calls: la proporción de sus escrituras que encontraron la fila cambiada (o borrada) y no escribieron nada. La aplicación recarga y reintenta cada una, un bucle que el lado de la base de datos nunca ve de otro modo. Una sentencia es ámbar cuando al menos el 5% de al menos 100 llamadas no escribió nada, o cuando lo hicieron 1.000 llamadas sea cual sea la proporción; la misma sentencia entre las 25 principales lleva la misma marca. Una sentencia que escribe más filas de las veces que se llama (un lote con condición de versión) se lista sin proporción, porque rows / calls no es entonces una fila por llamada.

`db-temp-queries` y `db-temp-files` tratan de los vuelcos a disco. `db-temp-files` es en vivo - ficheros que se están escribiendo ahora mismo, agrupados por PID, para que pilles la consulta que vuelca mientras ocurre. `db-temp-queries` es acumulado de `pg_stat_statements` - los que vuelcan de forma crónica, que es un `work_mem` demasiado bajo para el sort o el hash de esa consulta, o un orden de join que el planificador acertó mal. Un poco de temp es normal en una consulta de informes; la misma sentencia volcando gigabytes en cada ejecución es la que hay que arreglar.

#### Bloqueos, contención y transacciones atascadas {#locks-blocking-and-stuck-transactions}

`db-blocking` es la primera que abrir en un incidente de «la app está congelada»: empareja cada sesión bloqueada con la sesión exacta que la bloquea (vía `pg_blocking_pids()`), de modo que la cabeza de la cadena - la transacción a la que esperan todas las demás - es una fila, con su consulta y cuánto lleva reteniendo el bloqueo. Cancela o termina esa una y la cola se vacía.

`db-locks` es la vista en bruto de `pg_locks` - cada bloqueo retenido y en espera, no solo los que bloquean. Úsala cuando `db-blocking` esté vacía pero algo siga pareciendo atascado, o para ver *qué* modo de bloqueo interviene (un `ShareLock` esperando a un `RowExclusiveLock` es un insert de clave foránea contra una fila que se actualiza en otro sitio; una petición de `AccessExclusiveLock` detrás de un `SELECT` largo es una sentencia DDL o un `VACUUM FULL` que congelará la tabla hasta que termine la lectura).

`db-long-tx` lista transacciones abiertas más de un minuto, hagan lo que hagan. Una transacción larga que *trabaja* es una consulta lenta; una que está **idle in transaction** es un fallo de la aplicación, y es peor que una consulta lenta porque retiene su snapshot y sus bloqueos mientras no hace nada. `db-idle-in-tx` es ese segundo caso por sí solo, con cuánto lleva cada sesión aparcada a mitad de transacción. En cualquier caso el coste es el mismo: `VACUUM` no puede limpiar ninguna versión de fila más nueva que la transacción abierta más antigua, en todo el clúster - así que un `BEGIN` olvidado en un portátil puede hinchar todas las tablas del servidor.

La vista **Sessions** lleva el ángulo transaccional más allá de las comprobaciones. El panel de detalle de una sesión muestra *Edad del snapshot (backend_xmin)* - cuántas transacciones atrás está fijado su snapshot - y, cuando aplica, una línea roja: **esta sesión define ahora mismo el horizonte xmin del clúster**; autovacuum está bloqueado en todas partes hasta que termine su transacción. La lista de sesiones en vivo etiqueta esa misma sesión, y la vista Blocking convierte el grafo de espera de bloqueos en un árbol, de modo que la única transacción que retiene a todas las demás es una sola fila.

#### Transacciones preparadas (de dos fases) {#prepared-two-phase-transactions}

`db-prepared-xacts` cubre el único tipo de transacción que no tiene sesión alguna. `PREPARE TRANSACTION 'gid'` escribe la transacción en disco y retorna; un `COMMIT PREPARED 'gid'` o `ROLLBACK PREPARED 'gid'` posterior la termina. Entre medias mantiene cada bloqueo que tenía y fija el horizonte xmin del clúster - y nada en `pg_stat_activity` la muestra, porque no es una conexión. Un gestor de transacciones externo (la JTA de un servidor de aplicaciones Java, Windows MSDTC, un driver XA) que prepara una transacción y luego se cae o pierde la pista la deja preparada para siempre. Es el incidente clásico de «la base se hincha sin parar y todas las sesiones parecen bien».

La comprobación lista cada una - `gid`, propietario, base, hace cuánto se preparó, la edad de su transacción y las tablas que bloquea - y marca todo lo de más de diez minutos como probablemente huérfano. Resuélvela conectado a **su propia base**: `COMMIT PREPARED 'gid'` si el trabajo debe quedar, `ROLLBACK PREPARED 'gid'` si no.

**La mayoría de servidores no pueden tener transacciones preparadas en absoluto.** `max_prepared_transactions` es `0` por defecto, y la comprobación lo dice con claridad en vez de mostrar una tabla vacía - súbelo por encima de cero solo si algo necesita de verdad `PREPARE TRANSACTION`. Cuando está activado, una transacción preparada que está bloqueando una sesión viva ahora aparece en la vista Blocking como su propio nodo (etiquetado `prepared 'gid'`, con las dos sentencias de resolución para copiar), y en `db-locks` sus bloqueos retenidos se atribuyen al gid en lugar de desaparecer.

#### Historial de migraciones {#migration-history}

`db-migrations` (menú Salud del clúster) lee las tablas de historial que las herramientas de migración guardan en la base de datos conectada - Flyway (`flyway_schema_history`, antes de Flyway 5 `schema_version`), Liquibase (`databasechangelog`, `databasechangeloglock`), Alembic (`alembic_version`), golang-migrate y Rails (`schema_migrations`, distinguidas por la columna `dirty` de golang-migrate), Django (`django_migrations`), Prisma (`_prisma_migrations`), knex (`knex_migrations`, `knex_migrations_lock`) y Entity Framework Core (`__EFMigrationsHistory`) - en cualquier esquema. Para cada una dice qué se aplicó por último y marca los estados que detienen el siguiente despliegue, con la fila que lo muestra y cómo resolverlo:

- **CRIT - una migración fallida:** Flyway `success = false` (una migración que no pudo correr en una transacción, como `CREATE INDEX CONCURRENTLY`, falló a medias; Flyway se niega a migrar hasta `flyway repair`), golang-migrate `dirty = true` (arregla el esquema a mano y luego `migrate force`), Prisma una migración que empezó y ni terminó ni se revirtió, hace diez minutos o más (`prisma migrate resolve`).
- **Un bloqueo que dejó un proceso que murió:** Liquibase `databasechangeloglock.locked = true` es un aviso cuando lleva **5 minutos** retenido y crítico a los **60 minutos** (una migración de datos larga lo retiene con motivo, así que la fila dice quién lo tomó y cuándo; `liquibase releaseLocks` lo libera); knex `is_locked = 1` es un aviso (no guarda la hora).
- **INFO:** migraciones de Flyway aplicadas fuera de orden (`outOfOrder=true` - permitido, pero esta base de datos las ejecutó en un orden que una instalación nueva no seguiría), más de un head de Alembic, changesets de Liquibase registrados como `FAILED` con `failOnError=false`.

Una tabla que este login no puede leer se nombra como encontrada y no se adivina. El resumen de salud lleva lo peor de esto como su fila **MIGRACIONES**, y solo cuando la base de datos tiene una tabla así.

#### Tablas de cola y LISTEN/NOTIFY {#queue-tables-and-listen-notify}

`db-queues` (menú Conexiones y consultas) encuentra las tablas usadas como colas de trabajos: las que nombran las sentencias con `FOR UPDATE SKIP LOCKED` en `pg_stat_statements` (solo esta base de datos) y las tablas de las bibliotecas que guardan sus trabajos en PostgreSQL - pgmq, PgQ, que, graphile-worker y River. Para cada una: los trabajos en espera (solo los que ya tocan, no los programados para después), cuánto ha esperado el más antiguo, los trabajos fallidos donde la biblioteca los registra, filas vivas y muertas, tamaño y el último vacuum. Una tabla encontrada solo por sus sentencias no tiene un estado "en espera" conocido - muchas guardan los trabajos terminados -, así que su fila más antigua se muestra y no se califica.

- **Las filas muertas** las salta cada extracción de la cola, por eso una tabla de cola es la primera en sufrir cuando vacuum se retrasa o algo fija el horizonte xmin. Aviso al **50%** de la tabla con al menos **10.000** filas muertas, o con **1.000.000** por sí solas; crítico al **90%** con **100.000**, o con **10.000.000**. El texto da la proporción y la cantidad.
- **Espera:** aviso cuando el trabajo pendiente más antiguo ha esperado **5 minutos**, crítico a **1 hora**. Los consumidores de PgQ se califican igual por su retraso.
- **LISTEN/NOTIFY:** `pg_notification_queue_usage()` (PostgreSQL 9.5+) dice cuán llena está la única cola de notificaciones del clúster. Solo se recorta hasta lo que ha leído el oyente más lento, y una sesión lee solo entre transacciones, así que un oyente inactivo dentro de una transacción la deja crecer hasta que cada `NOTIFY` falla en el commit. Aviso al **10%** de su máximo o **512 MB**, crítico al **50%** (donde PostgreSQL empieza a avisar en su log) o **4 GB**; el máximo es 8 GB, o `max_notify_queue_pages` desde PostgreSQL 17. Se listan las sesiones cuya última sentencia fue `LISTEN`, con su estado - PostgreSQL no tiene una vista de oyentes, y es lo más cerca que se llega. Las mismas líneas encienden la lámpara **Cola LISTEN/NOTIFY** de la cabecera, que aparece en un clúster donde alguien escucha.

Los recuentos están acotados (un millón de filas) y cada consulta tiene cinco segundos, así que un retraso enorme sin índice se informa como no medido en vez de leerse entero.

#### Los asesores de caché e índices {#the-cache-and-index-advisors}

`db-cache-hit` es el número único: la fracción de lecturas de bloque servidas desde `shared_buffers` desde el último reset de estadísticas, para esta base de datos. Es una media acumulada sobre toda la vida de las estadísticas, así que un informe frío puntual de hace una hora sigue tirándolo hacia abajo ahora - por sí solo no te dice casi nada. `db-cache-advisor` es la versión sobre la que vale la pena actuar.

`db-cache-advisor` informa de los ratios de acierto de heap, índice y TOAST, luego las tablas e índices que hacen más lecturas físicas, `shared_buffers` frente al tamaño de los datos y una comprobación de sensatez de `effective_cache_size`. Un ratio por debajo de ~99% solo importa si los fallos están concentrados - una tabla de informes fría leída una vez al día no vale la pena afinar. Actúa sobre la lista de objetos calientes, no sobre el número principal.

`db-index-advisor` encuentra tablas con alta proporción de escaneos secuenciales y lista columnas candidatas a partir de los predicados en `pg_stat_statements`, incluidos huecos de columna líder en índices existentes y claves foráneas sin índice que las cubra. Escribe el `CREATE INDEX` de cada una. Dos trampas: una tabla pequeña está pensada para escanearse secuencialmente, así que ignora los candidatos por debajo de unos pocos miles de filas; y todo índice cuesta al escribir, así que añade el que necesita la consulta caliente, no todos. Para evaluar un índice concreto antes de construirlo, usa la prueba de índice (sección 6.5.5), que lo planifica con HypoPG.

Con instantáneas AWR de la base de datos guardadas, `db-index-advisor` añade tres secciones de historial tras un bloque **Evidencia** que nombra las instantáneas y los días de ASH usados. **Escaneos secuenciales en las ventanas guardadas**: por tabla, escaneos y filas leídas al día, filas por escaneo, y la primera mitad del historial frente a la segunda; *en aumento* significa que la segunda mitad es al menos el doble que la primera y al menos 100 al día, o al menos 10.000 más al día, y una tabla aparece cuando está en aumento o sus escaneos secuenciales leen al menos 100.000 filas al día. Junto a cada una, las sentencias que la nombran y más leen del disco. **Sentencias que más leen del disco**: `shared_blks_read` de `pg_stat_statements` sumado sobre las ventanas, por llamada, con la ventana más activa y cuántas veces ASH pilló la sentencia esperando I/O (relacionadas por query id desde PostgreSQL 14). **Sin escaneos en la ventana guardada**: los escaneos de cada índice sumados sobre las ventanas (un reinicio de estadísticas entremedias empieza una cuenta nueva, no se pierde nada), con la ventana indicada. Con al menos **7 días** un índice sin escaneos es un *candidato a revisar*, por debajo la ventana es *demasiado corta para juzgar*, y cada fila lleva la advertencia de que un informe mensual, trimestral o anual puede ser su único lector. Ahí no se ofrece `DROP`, y las claves primarias, los índices únicos y los de claves foráneas nunca son candidatos: se listan aparte como conservados, con el motivo.

#### Mantenimiento de índices {#index-housekeeping}

Tres comprobaciones encuentran índices que cuestan escrituras y disco sin devolver nada. Las tres leen `pg_stat_user_indexes`, así que sus recuentos de escaneo son **desde el último reset de estadísticas** - un `pg_stat_statements_reset()` o un `pg_stat_reset()` la semana pasada hace que «nunca escaneado» cubra solo una semana.

`db-index-usage` es el cuadro completo: cada índice con su recuento de escaneos, tamaño y la proporción de escaneos frente a escrituras en la tabla. `db-index-unused` es el subconjunto que importa - índices con cero escaneos - y ya excluye aquellos para los que un cero es normal: los que respaldan una `PRIMARY KEY`, una `UNIQUE` o una `EXCLUDE`, y la replica identity de una tabla. Esos hacen cumplir algo lea o no una consulta, y `DROP INDEX` los rechaza, así que una lista de borrado que los incluyera sería peligrosa. También deja fuera todo índice sobre una columna de **clave foránea**: se lee como nunca escaneado porque `DELETE` y `UPDATE` en la tabla padre lo usan internamente, y sin él cada uno de ellos recorre la tabla hija bajo un bloqueo. Esos se listan aparte, cada uno con la clave foránea que respalda, y nunca se ofrecen para eliminar - ni aquí, ni en el resumen de salud, ni en el bloat de índices, ni en el Advisor de la tabla. Lo que queda es un **candidato a revisar, no un veredicto**: la comprobación escribe el `DROP INDEX CONCURRENTLY` de cada uno y dice cuánto tiempo cubren sus contadores, porque un índice usado solo por un informe mensual, trimestral o anual se lee como sin uso todo el tiempo entre medias - confírmalo en un periodo que incluya uno.

`db-duplicate-indexes` encuentra índices que se repiten entre sí. Los **idénticos** - misma tabla, columnas, clases de operador, expresiones y predicado - se muestran como un grupo, y la comprobación nombra el que **se queda** y por qué: el que hace cumplir o sirve algo (una clave primaria, una restricción `UNIQUE` o `EXCLUDE`, un índice único, la replica identity, el índice de una clave foránea), si no, el más escaneado. Los demás se marcan como **redundantes - idénticos a** el que se queda. Cuando dos de ellos hacen cumplir algo (dos restricciones `UNIQUE` sobre las mismas columnas), lo dice en su lugar: quitar uno significa cambiar una restricción. También lista un índice simple cuyas columnas son las **columnas iniciales de uno más amplio** con las mismas clases de operador - el más amplio responde a las mismas búsquedas, algo más grande de leer; los índices únicos y de restricción nunca aparecen ahí, y cuando el más estrecho respalda una clave foránea, el más amplio la sigue cubriendo. La comprobación no escribe ningún `DROP` - un índice redundante cuesta escrituras y disco, y la decisión es suya. Guarde la definición antes de quitar uno - es la única copia.

Para actuar sobre un solo índice que nombran los informes - el `REINDEX` o `DROP` exacto, su historial de escaneos y un cotejo con todos los demás índices de la misma tabla - pulsa su nombre para abrir el asesor por índice (ver *Un índice, y qué hacer con él* arriba).

#### Adónde va el espacio: los informes de tamaño {#where-the-space-goes}

Cuatro informes responden a «qué está usando el disco», cada uno un nivel más
fino que el anterior. Son pura medición - sin asesor, sin recomendación -, así
que la única forma de leerlos mal es malinterpretar qué incluye un número.

`db-sizes` - **cada base de datos del clúster.** `pg_database_size()` por
base, más el número de conexiones vivas (solo backends de cliente - los
workers de autovacuum y los procesos de fondo también llevan un `datname`, y
contarlos haría que una base ociosa pareciera ocupada), el ratio de acierto de
caché desde el último reset de estadísticas y el uso acumulado de ficheros
temporales. Las bases plantilla quedan fuera. Es el único informe aquí que ve
a través de las bases; la conexión en la que estás no afecta a sus números.

`db-schema-sizes` - **la base conectada, desglosada por esquema.** Total por
esquema, bytes de datos de tabla y de índice por separado, el porcentaje de la
base de cada esquema, el número de objetos y la tabla individual mayor de cada
uno. El total cuenta una sola vez todo lo que ocupa disco en el esquema:
tablas, su TOAST, sus índices, vistas materializadas, secuencias. No puede
cruzar a otras bases porque `pg_namespace` es por base de datos.

`db-table-sizes` - **la tabla individual**, las 25 mayores por tamaño total.
`total` es `pg_total_relation_size()` - heap **más** su tabla TOAST y su
índice TOAST **más** cada índice sobre ella **más** los mapas de espacio libre
y de visibilidad. `table_only` es solo el heap; `indexes` es
`pg_indexes_size()`. Así que `total` menos `table_only` menos `indexes` es el
TOAST y los mapas, que en una tabla con columnas `text`/`jsonb` anchas puede
ser la mayor parte del tamaño. Las particiones se agregan a la raíz de
partición - una tabla particionada aparece como una fila con un recuento
`partitions`, no como un muro de filas hijas con el padre a cero. `live_rows`
y `dead_rows` son **estimaciones** del recolector de estadísticas, no un
`count(*)`; `stats_age` es hace cuánto que el último `ANALYZE` los refrescó, y
un `stats_age` grande significa que los recuentos de filas están obsoletos en
la dirección en que la tabla se ha movido.

`db-index-sizes` - **el índice individual**, los 50 mayores por tamaño de
índice, cada uno con el tamaño de su tabla padre al lado, de modo que un
índice que ha crecido más que los datos que indexa salta a la vista. Ese ratio
es la señal: un índice btree mayor que su tabla o indexa claves muy anchas o
está hinchado (ver la sección siguiente), y en ambos casos merece una mirada.
Un índice GIN o GiST que legítimamente empequeñece su tabla es normal -
almacenan mucho más de una entrada por fila.

La trampa común a los cuatro: son **tamaños en disco**, que incluyen tuplas
muertas y espacio libre que un `VACUUM FULL` o `pg_repack` recuperaría. Una
tabla que marca 4 GB no son 4 GB de datos si la mitad es bloat. Contrasta un
tamaño sorprendente con `db-bloat-breakdown` o `db-pgstattuple` antes de
concluir que necesitas un disco mayor.

#### Bloat: la estimación y el número real {#bloat-the-estimate-and-the-real-number}

«Bloat» es el espacio que una relación ocupa en disco por encima de lo que sus
filas vivas necesitan - tuplas muertas que autovacuum no ha recuperado y
páginas que quedaron medio vacías tras borrados y actualizaciones. Tres
informes lo abordan, de rápido-y-aproximado a lento-y-exacto, y cada uno de
los basados en estimación - el desglose por tabla, la clasificación de
índices, la pestaña Advisor y la fila *WASTED SPACE* del resumen de salud -
se calcula con el mismo código y llega al mismo número para la misma
relación, porque todos llaman a un único módulo compartido en lugar de
calcularlo cada uno por su cuenta. **Lee las estimaciones como «merece
investigarse», nunca como una medición.**

`db-bloat-breakdown` es por donde empezar. Por tabla, lee tamaños reales
(`pg_relation_size`, `pg_indexes_size`, `pg_total_relation_size`, y
`pg_total_relation_size` sobre la relación TOAST) y los combina con un
tamaño *esperado* («compacto») calculado a partir de `pg_stats` (número de
columnas, ancho medio, fracciones de nulos) para el heap, el mismo modelo
por entrada para cada índice btree, y la propia proporción de tuplas muertas
de la tabla TOAST - un único `est_waste` / `est_waste_pct` para **toda la
tabla**, heap más TOAST más cada índice sumados, no el heap solo. Eso
corrige el caso de una tabla que antes mostraba «0% de bloat» aquí mientras
solo sus índices acumulaban cientos de gigabytes de desperdicio estimado: la
cifra ahora los incluye. Imprime una columna `recommendation` derivada de
esos mismos totales que muestra, de modo que el consejo y la cifra ya no
pueden contradecirse. Lee esa columna como **un empujón, no una
instrucción** - no sabe si tienes `pg_repack` instalado. Una línea «VACUUM
FULL or pg_repack» en una tabla de 20 MB la satisface un `VACUUM` normal; la
misma línea en una tabla ocupada de 200 GB significa una reconstrucción en
línea y un plan de mantenimiento.

**Que la estimación de un componente no sea fiable no vacía toda la fila.**
La fórmula del tamaño compacto de un solo componente - normalmente el heap
de una tabla grande o mucho tiempo sin analizar - puede salir *mayor* que
el tamaño real de ese componente: estadísticas obsoletas, o filas
inusualmente anchas o muy toasteadas para que encaje el modelo por columna
de `pg_stats`. Eso no es «sin desperdicio», es «la estimación de este
componente no aplica aquí» - y no descarta lo que los *otros* componentes
(TOAST, cada índice) sí conocen. `est_waste` pasa a ser un **límite
inferior**: marcado con `≥` e indicando qué falta, p. ej.
`≥ 448 GB (heap unknown)`, en vez de un cero inventado o un «n/a» en
blanco - el total real es esa cifra o mayor, nunca menos. `est_waste_pct`
lleva la misma marca `≥` cuando corresponde. `ANALYZE`, o `db-pgstattuple`
más abajo, es el camino para superarlo específicamente en el componente que
falta - la nota de la fila indica cuál y por qué: nunca analizada,
estadísticas obsoletas hace N días con qué proporción de filas ha cambiado
desde entonces (ambas apuntan a `ANALYZE`), o estadísticas actuales que aun
así no encajan con la forma de la tabla (apunta a `db-pgstattuple` en su
lugar, porque otro `ANALYZE` no cambiaría nada). Esta misma regla - los
componentes conocidos siguen visibles, los que faltan se nombran, nada se
deja en blanco - se aplica en todo sitio donde una desglose suma varios
componentes, incluida la fila *WASTED SPACE* del resumen de salud.

**`est_waste_pct` y `dead_pct` son cifras distintas a propósito.**
`est_waste_pct` es espacio: bytes que la tabla (heap, TOAST, índices)
retiene pero no usa. `dead_pct` es `n_dead_tup / (n_live_tup +
n_dead_tup)` - un recuento de tuplas, y solo una medida de cuánto va por
detrás autovacuum. Una tabla muy castigada por `UPDATE` puede estar al 4%
muerta y al 90% desperdiciada a la vez. `idx_to_heap` es bytes totales de
índice sobre bytes de heap, donde bastante por encima de 1 merece una
auditoría de índices no usados/duplicados.

`db-bloat-indexes` responde una pregunta distinta: no «qué tabla está peor»
sino «qué índice concreto, en cualquier lugar del clúster, merece un
`REINDEX` ahora mismo» - la misma aritmética por índice que las filas de
índice del desglose, pero clasificada a nivel de clúster en vez de agrupada
por tabla. Asume `fillfactor` 90 y cubre **solo btree** - los índices GIN,
GiST, BRIN, hash y de expresión reciben un tamaño real y ninguna estimación,
que es el estimador retirándose, no un índice limpio. **`VACUUM` nunca
compacta un índice**; la solución que nombra es `REINDEX` (`... CONCURRENTLY`
en PostgreSQL 12+, comprobado contra el servidor conectado para que a uno
más antiguo no se le diga que ejecute una sentencia que no tiene), una línea
por índice. Una tabla construida a propósito con `fillfactor` por debajo de
100, para abaratar las actualizaciones HOT, también mostrará ese espacio
libre como «bloat» aquí - es el espacio que pediste, no desperdicio.

`db-pgstattuple` es el número real, para ambas mitades. Lee **cada página**
de cada tabla mediante la extensión `pgstattuple` para los porcentajes
verdaderos de tuplas muertas y espacio libre, y luego `pgstatindex()` sobre
los índices btree de esa tabla para una cifra real de densidad de página
hoja por índice - sin modelo de `pg_stats` en ningún lado, un escaneo de
página real. Es exacto y tan lento como implica ese escaneo: el coste es el
tamaño de la tabla en E/S, cada vez. Úsalo para confirmar una tabla concreta
que la estimación marcó - o una que marcó como no fiable -, no para
inspeccionar una base de datos. Los parámetros de rango de tamaño son un
presupuesto de tiempo: el valor por defecto salta las tablas por debajo de
1 MB (el bloat ahí es un error de redondeo) y por encima de 5 GB (súbelo
cuando quieras concretamente la cifra exacta de una tabla grande y puedas
esperar). La extensión también ofrece `pgstattuple_approx`, que salta las
páginas que el mapa de visibilidad marca como todo-visible y así corre mucho
más rápido en una tabla bien vacuumada a costa de algo de precisión -
conviene saberlo cuando el escaneo completo es demasiado lento.

**Un orden práctico:** `db-bloat-breakdown` para ver la forma de todo esto en
cada tabla, `db-bloat-indexes` para una lista ordenada de sospechosos a nivel
de clúster, `db-pgstattuple` en la una o dos tablas sobre las que de verdad
vas a actuar (o cualquier fila marcada como no fiable). Luego el asesor por
índice (arriba) para la sentencia `REINDEX` o `VACUUM` exacta.

#### Shared buffers: qué hay realmente en caché {#shared-buffers-what-is-actually-cached}

`db-buffer-cache` abre `pg_buffercache` y muestra qué relaciones ocupan ahora
mismo el pool de shared buffers de PostgreSQL - las 30 primeras por número de
buffers, con el tamaño total de cada relación, su parte de `shared_buffers` y
cuánto de la propia relación está en caché.

**El coste es real y conviene conocerlo antes de ejecutarlo en producción.**
La vista `pg_buffercache` recorre todo el pool de buffers; en un servidor con
`shared_buffers` de varios gigabytes eso son millones de buffers examinados
bajo bloqueos de partición, y se nota como un breve pico de latencia para
todos los demás. Ejecútalo cuando lo necesites, no en un horario, y
preferiblemente en un momento tranquilo. En PostgreSQL 16 y posteriores, las
funciones más baratas `pg_buffercache_summary()` y
`pg_buffercache_usage_counts()` dan la foto general sin el recorrido buffer a
buffer - usa esas si solo quieres «cómo de lleno está el caché».

Cómo leer los dos porcentajes:

- **`pct_of_relation`** - cuánto de esta relación está en caché. Una tabla de
  búsqueda pequeña y caliente al 100% es exactamente lo correcto. Una tabla de
  200 GB al 3% también es correcto - no querrías que desalojara todo lo demás.
  Este número solo es un problema cuando una tabla que *esperas* que se sirva
  de memoria está baja.
- **`pct_of_buffers`** - la parte de esta relación en todo el pool. Es una
  alarma solo cuando una o dos relaciones lo dominan y las tablas que te
  importan quedan sin servir. Un pool sano suele ser una cola larga.

Si la parte alta de la lista son tablas TOAST e índices que no reconoces, es
normal - las páginas de índice y los trozos de TOAST se cachean como cualquier
cosa. La comprobación sirve para responder a «por qué mi conjunto de trabajo
no se queda en memoria», y la respuesta suele ser o que `shared_buffers` es
demasiado pequeño para el conjunto de trabajo o que un trabajo por lotes tira
repetidamente de una tabla fría grande a través de él.

#### TimescaleDB {#timescaledb}

`db-timescaledb` lee la extensión TimescaleDB de la base de datos a la que
está conectado. Para todo lo que sigue necesita TimescaleDB 2; con 1.x
instalado lo dice, y el navegador de esquemas sigue mostrando las
hipertablas y sus chunks.

**Edición.** La primera línea da la versión y el ajuste de licencia
(`timescaledb.license`). Con `apache` - la edición solo Apache - el propio
servidor rechaza la compresión, los agregados continuos, las políticas de
retención, compresión, refresco y reordenación y los trabajos de usuario;
las hipertablas, los chunks, `drop_chunks` y `time_bucket` funcionan. El
panel muestra esas acciones desactivadas y dice por qué. Si en el servidor
hay instalada una versión más nueva que en la base de datos, se nombra con
el `ALTER EXTENSION timescaledb UPDATE` que la aplica.

**En el navegador de esquemas** las hipertablas tienen una categoría propia;
cada una dice cómo se divide en chunks (columna de tiempo e intervalo de
chunk, y las particiones de espacio si las hay), cuántos chunks tiene y
cuántos están comprimidos; su número de filas es la suma de sus chunks,
porque la tabla propia de la hipertabla está vacía. Al abrirla muestra su
tamaño, la proporción de compresión y sus chunks, los más recientes primero
(los 500 más recientes; el resto se cuenta). Un chunk se abre como la tabla
que es, y su menú lo comprime o descomprime. Los esquemas de TimescaleDB
(`_timescaledb_internal`, `_timescaledb_catalog`, `timescaledb_information`
y los demás) y las relaciones que gestiona - chunks, datos comprimidos, las
tablas y vistas detrás de los agregados continuos - se ocultan como otros
objetos de extensiones, tras el mismo interruptor "mostrar".

**Qué juzga la comprobación**, con los números con que juzga:

- **Un trabajo que sigue fallando** - dos ejecuciones fallidas seguidas
  desde su último éxito (contadas en `timescaledb_information.job_errors`;
  antes de 2.9, que no tiene esa vista, solo se conoce la última ejecución).
  Una alarma.
- **Un trabajo que no se ejecutó según lo programado** - su próximo inicio
  quedó más de 5 minutos en el pasado. El planificador inicia un trabajo en
  segundos cuando hay un proceso en segundo plano libre, así que minutos de
  retraso significan que no lo hay. Una alarma.
- **El planificador no está en marcha** para esta base de datos mientras hay
  trabajos programados - normalmente `timescaledb.max_background_workers` es
  demasiado bajo (necesita un proceso por base de datos con TimescaleDB más
  uno por trabajo que se ejecute a la vez) o `max_worker_processes` está
  agotado. Una alarma.
- Los trabajos de mantenimiento propios de TimescaleDB (telemetría,
  retención del historial de trabajos) se informan cuando fallan, nunca como
  alarma: la telemetría falla en cualquier servidor sin acceso a internet.
- **Compresión por detrás de su política** - chunks más antiguos que el
  `compress_after` de la política que siguen sin comprimir, cada uno con un
  botón Comprimir.
- **Retención por detrás de su política** - chunks más antiguos que
  `drop_after` que siguen ahí.
- **Un agregado continuo que se queda atrás** - el tiempo desde su marca de
  agua (lo materializado) hasta ahora, frente a lo que su política de
  refresco debería mantener: `end_offset` más la programación. Se marca a
  partir del doble de eso y de 15 minutos, para que una política que
  refresca cada minuto no se marque por un retraso momentáneo. Sin política
  se dice en todo caso, como advertencia cuando el agregado es
  `materialized_only` (las consultas no ven entonces nada posterior a la
  marca de agua).
- **Intervalo de chunk frente a memoria** - la regla de Timescale: los
  chunks en los que se escribe, con sus índices, de todas las hipertablas
  juntas deberían caber en un 25% de la memoria principal, la parte que
  `timescaledb-tune` da a `shared_buffers`. La comprobación mide el
  intervalo terminado más reciente de cada hipertabla escrita en los últimos
  7 días y compara su suma con `shared_buffers`; por encima, cada hipertabla
  recibe un intervalo reducido para que quepa. También se marca lo
  contrario: más de 1000 chunks cuyo intervalo guarda menos del 1% de
  `shared_buffers` o menos de 10 MB - cada chunk es una tabla que el
  planificador considera. Un intervalo nuevo (`set_chunk_time_interval`) se
  aplica a los chunks creados desde entonces.

Una advertencia o una alarma en la lista sobre el panel tiene el botón
habitual "Añadir a la lista de tareas", y las alarmas encienden la luz de
TimescaleDB en la cabecera (la misma lectura que Health muestra como
TRABAJOS DE TIMESCALEDB).

**Acciones.** Desde el panel: añadir o quitar una política de compresión o
de retención, comprimir un chunk, refrescar un agregado continuo para una
ventana, ejecutar un trabajo ahora, cambiar su programación, pausarlo y
reanudarlo; desde el menú de un chunk: comprimirlo o descomprimirlo. Cada
una muestra primero su sentencia y la que la deshace - quitar una política
guarda su ajuste en la sentencia que la vuelve a poner - y ambas van al
registro de auditoría. Solo el propietario de la hipertabla, del agregado o
del trabajo, o un miembro de ese rol, puede hacerlo; el panel pregunta
`pg_has_role(owner, 'USAGE')` y desactiva lo que no puede hacer, y el
servidor vuelve a preguntar.

**El resto de la aplicación en una base de datos con TimescaleDB:**

- La comparación compara una hipertabla como tabla y además por cómo se
  divide en chunks y se comprime, así que una tabla normal y una hipertabla,
  o dos intervalos de chunk, difieren. Los chunks no se comparan: se nombran
  por orden de creación, y `_hyper_1_5_chunk` en un lado es otro intervalo
  de tiempo que el mismo nombre en el otro. Un intervalo distinto recibe un
  `set_chunk_time_interval`; convertir una tabla en hipertabla queda como
  sentencia comentada, porque mueve cada fila.
- La restauración envuelve un volcado con TimescaleDB como exige
  TimescaleDB: `timescaledb_pre_restore()` antes,
  `timescaledb_post_restore()` después - también cuando la restauración
  falla, para que la base de datos nunca quede con sus trabajos detenidos.
  En un destino vacío primero se crea la extensión. El destino debe tener la
  versión de TimescaleDB de la que procede el volcado. Se avisa al restaurar
  solo una parte de ese volcado o con `--clean`: una hipertabla sin sus
  filas de catálogo ni sus chunks vuelve como una tabla normal vacía.
- Cambiar el propietario de una hipertabla mueve con ella sus chunks (lo
  hace TimescaleDB). La copia de las definiciones tomada antes de un cambio
  guarda la hipertabla, no sus chunks, y la sentencia que la vuelve a
  convertir en hipertabla.
- El mapa de relaciones dibuja una hipertabla como una sola tarjeta con su
  número de chunks y enlaza un agregado continuo con la hipertabla que
  agrega.
- Otras comprobaciones que listan tablas (tamaños, bloat, vacuum) nombran
  junto a cada chunk la hipertabla o el agregado continuo.

#### Vacuum, bloat y estadísticas de tablas {#vacuum-bloat-and-table-statistics}

`db-vacuum` muestra, por tabla, cuándo se pasó vacuum y analyze por última vez, el número de tuplas vivas y muertas, el porcentaje modificado desde el último analyze y una etiqueta de calor, con un conjunto recomendado de reloptions de autovacuum junto al actual. Las etiquetas son lo que más importa: **riesgo de WRAPAROUND** es para actuar ya; **autovacuum desactivado** en una tabla ocupada suele ser un error olvidado; una tabla **solo-inserción** necesita `autovacuum_vacuum_insert_threshold`, ya que el umbral de vacuum por defecto nunca se dispara para ella.

`db-autovacuum` es la vista en vivo: qué workers de autovacuum están corriendo ahora mismo y qué hace cada uno. Vacío es el estado normal. Un worker que lleva mucho en la misma tabla grande, o `max` workers todos ocupados mientras el atraso de vacuum en `db-vacuum` sigue creciendo, significa que autovacuum no da abasto - sube `autovacuum_max_workers` o `autovacuum_vacuum_cost_limit`, o baja el scale factor por tabla en los peores casos.

`db-bloat-breakdown` separa el bloat de heap, TOAST e índice por tabla, los combina en un único `est_waste` para toda la tabla y nombra el siguiente paso - VACUUM, REINDEX, `pg_repack` o nada. Su columna `recommendation` es un empujón derivado de esos mismos totales que muestra, no una instrucción: *Bloat: la estimación y el número real* (arriba) cubre qué tiene y qué no tiene en cuenta, y cómo encajan a su alrededor la clasificación de índices a nivel de clúster (`db-bloat-indexes`) y el escaneo exacto (`db-pgstattuple`).

La fila *WASTED SPACE (est.)* del **resumen de salud** usa esta misma estimación combinada de heap más TOAST más índices, agregada sobre la base de datos, y marca una tabla según **lo que se dispare primero**: **20 GB** absolutos de desperdicio estimado en una tabla (rojo a **200 GB**), o **25% o más** de la tabla desperdiciado cuando eso sea al menos **1 GB**. El número absoluto existe porque el 40% de una tabla de 200 MB es tarea doméstica, mientras que el 6% de una tabla de 20 TB es más de un terabyte de disco y horas añadidas a cada restauración. Una tabla con un componente no fiable (normalmente el heap) **no** se elimina de esta fila - sus demás componentes conocidos siguen contando, como límite inferior marcado con `≥`, con la misma regla que sigue la fila por tabla en `db-bloat-breakdown`; ahí se indica qué componente falta y por qué. El desglose lista las peores tablas y la pestaña Advisor por tabla desglosa cada una por componente. `pg_repack` y `VACUUM FULL` devuelven el espacio de heap/TOAST al sistema operativo; un `VACUUM` normal solo lo libera para reutilizarlo dentro de la misma tabla; un índice necesita `REINDEX`.

La **pestaña Advisor** del navegador de esquemas (elige una tabla -> Advisor) es la versión por tabla de todo esto en un solo lugar. Muestra un **desglose de bloat** por componente - heap, TOAST, cada índice - con tres números cada uno: qué hay en disco (exacto), cuánto de eso es **dato útil** estimado y cuánto es **desperdicio** estimado. El *tamaño* de un índice no es desperdicio; el desperdicio de un índice es el espacio que un `REINDEX` nuevo no necesitaría. Un TOAST grande no es problema por sí solo - el desperdicio dentro de él sí. El «desperdicio» está anclado al tamaño real en disco (la misma aritmética que `db-bloat-breakdown`), así que se mantiene correcto aunque `pg_class.relpages` esté desactualizado, y la fila de un componente muestra `?` en lugar de una cifra cuando sus datos de entrada no encajan. La fila **total** nunca hace eso: se marca con `≥` e indica el componente que falta (con una razón breve y acciones `ANALYZE`/Escaneo preciso cuando aplican) cada vez que un componente no se pudo estimar, en vez de mostrar un cero inventado o una cifra en blanco. Las estimaciones asumen un fillfactor y la cifra de TOAST es solo una proporción de tuplas muertas, así que ejecuta *Escaneo preciso (pgstattuple)* para las cifras reales antes de actuar sobre algo grande. El bloque **Análisis y sugerencias** de abajo convierte los números en consejos condicionales con SQL listo para copiar: el desperdicio reutilizable con autovacuum al día se deja en paz; un fichero inflado recibe `pg_repack` o `VACUUM FULL`; si autovacuum se queda atrás se enlaza a los ajustes por tabla que el asesor de autovacuum calculó a partir de los números de esta tabla; los índices inflados reciben un `REINDEX INDEX CONCURRENTLY` por índice (o `REINDEX` simple en PostgreSQL < 12, señalado); el desperdicio de TOAST se destaca porque un `VACUUM` simple no lo compacta.

La pestaña Advisor son cinco secciones, en las mismas tarjetas que usa el resultado de una comprobación, con una tira bajo el título que salta a cualquiera de ellas: **Tabla** (sus cifras), **Vacuum y autovacuum**, **Índices**, **Mantenimiento** y **Análisis profundo**. La sección de vacuum empieza por cómo ha tratado el vacuum a esta tabla - cuántas ejecuciones desde el reinicio de las estadísticas y cuántas al día, el intervalo entre ellas, en PostgreSQL 18 cuánto duran al día, la proporción de filas muertas y el snapshot más antiguo que algo sostiene en la base de datos - y dice, con las mismas palabras que `db-vacuum`, cuándo una tabla se limpia cada hora o más a menudo, cuándo sus vacuums no consiguen nada porque una transacción, una transacción preparada, un slot de replicación o un standby sostienen un snapshot más antiguo que sus filas muertas (y qué terminar), cuándo cada vacuum suyo es agresivo y cuándo el autovacuum funciona sin freno. Una recomendación nunca endurece los umbrales de una tabla que ya se limpia cada hora. Después vienen las líneas de disparo - cuándo se activa el autovacuum hoy y tras la recomendación -, los cuatro parámetros y los botones que los aplican. Cuando un bloqueo mantiene a vacuum lejos de la tabla, ese hallazgo va primero y nombra la sesión que lo tiene.

**Cuándo está bien limpiar a menudo.** Una tabla limpiada cada hora o más a menudo - al menos 24 ejecuciones al día *y* al menos 50 en la ventana de estadísticas; cada cuarto de hora es «muy frecuente» - no se señala como problema solo por la cuenta. Se sopesan cuatro cosas más, y el hallazgo dice cuál de cinco veredictos se aplica:

- **El disparador.** Autovacuum se lanza cuando las filas muertas superan `threshold + scale_factor × filas` (primero el ajuste propio de la tabla, luego el del servidor; desde PostgreSQL 18 limitado por `autovacuum_vacuum_max_threshold`). El hallazgo dice qué parte decide - en una tabla pequeña el umbral fijo (50 + 0.2 × 120 filas es sobre todo el 50), en una grande el factor de escala - y qué parte de la tabla es el disparador.
- **El trabajo que encuentra cada ejecución.** Filas actualizadas o borradas por ejecución (desde PostgreSQL 13 también las insertadas, frente al disparador de inserciones). Una ejecución *limpia* cuando eso llega a **la mitad del disparador o 10.000 filas**; está *vacía* cuando queda **por debajo del 10% del disparador y de 1.000 filas**. Un `VACUUM` manual programado es el caso vacío habitual: se ejecuta haya cambiado algo o no.
- **Lo que cuesta una ejecución.** En PostgreSQL 18 se mide por tabla. Antes de 18 se estima a partir del freno: las páginas que lee una ejecución (las que el mapa de visibilidad no marca como all-visible, las que tocaron los cambios y todos los índices cuando la ejecución limpia índices - desde PostgreSQL 14 se omite cuando menos del 2% de las páginas tienen entradas de índice muertas), cada lectura contada como lectura de disco a `vacuum_cost_page_miss`, cada página ensuciada a `vacuum_cost_page_dirty`, al ritmo de `cost_limit` unidades por `cost_delay`. El límite de coste se reparte entre los workers de autovacuum que trabajan a la vez, salvo que la tabla tenga sus propios ajustes de coste, así que la estimación lo divide entre los workers que trabajan ahora. Un `VACUUM` manual corre por defecto con `vacuum_cost_delay` 0 - sin freno - y su duración no se puede estimar así. En PostgreSQL 18 la estimación queda como explicación: las ejecuciones duran más o menos lo que permite el freno (limitadas por el freno: un límite de coste mayor las acortaría), menos (las páginas venían de la caché) o más (esperan a disco, bloqueos o CPU). El hallazgo da también el techo de lectura y escritura que implica el freno. *Barato* es **como mucho 5 segundos por ejecución y 15 minutos al día**.
- **El WAL que escribe.** Siempre una estimación - ninguna versión de PostgreSQL cuenta el WAL por tabla: unos 100 bytes por página ensuciada, más una imagen de página completa para una página no registrada desde el último checkpoint. Una página que la carga cambió después del último checkpoint no se vuelve a copiar, así que una tabla limpiada *más* a menudo de lo que ocurren los checkpoints paga muchas menos imágenes de lo que sugiere su número de ejecuciones; el hallazgo compara el intervalo entre ejecuciones con el intervalo real de checkpoints y dice qué caso se aplica. Desde PostgreSQL 14 la estimación se limita al WAL medido del clúster y se compara con él. Grande es **1 GB al día y el 10% del WAL del clúster** (aviso desde **10 GB y 30%**). `wal_compression` reduce las imágenes; el hallazgo lo menciona sin modelarlo.

Los veredictos: **ok** - barato, limpia, filas muertas como mucho 1,5 × el disparador (o menos de 1.000), WAL no grande: se muestra como OK, las ejecuciones al día en verde, nada que cambiar. **idle** - se limpia más a menudo de lo que cambia la tabla (una nota cuando las ejecuciones son baratas; un aviso cuando son muy frecuentes y no baratas, o de coste desconocido): programe el `VACUUM` manual con menos frecuencia; para las ejecuciones propias de autovacuum, mire el disparador de inserciones y la edad de los ID de transacción. **costly** - ejecuciones caras en una tabla de al menos 100.000 filas cuyo disparador es menos del 1% de ella, con el ritmo de escritura y el umbral y el factor de escala que la limpiarían más o menos cada hora - esos pasan a ser los valores recomendados que llevan **Apply** y la sentencia ALTER; o ejecuciones bien en todo salvo en su WAL. **busy** - la frecuencia viene del ritmo de escritura, no de los ajustes: limpiar menos dejaría crecer el bloat, y cada ejecución se abarata con más actualizaciones HOT (fillfactor), menos índices o un freno más suave. **unknown** - no se puede decir el coste. Una recomendación nunca endurece los umbrales de una tabla limpiada a menudo, sea cual sea el veredicto; una cara se afloja en cambio hasta el disparador de una vez por hora, y el motivo aparece junto a los valores recomendados.

**Cómo se cuenta «por día».** Cada cifra diaria son las ejecuciones desde el inicio de la ventana de estadísticas de la tabla divididas por su duración - un promedio sobre esa ventana -, y la tarjeta y el hallazgo dicen dónde empieza, por ejemplo *since 2026-10-03 14:20 CET (25.0 days, this database's statistics reset)*. El inicio es, en este orden: en PostgreSQL 19, el momento de reinicio propio de la tabla; si no, el último reinicio de estadísticas de esta base de datos; para una base de datos nunca reiniciada (PostgreSQL 15 y posteriores dejan ese momento vacío), el reinicio más antiguo de las estadísticas del propio servidor; si no hay ninguno, el arranque del servidor. PostgreSQL mueve el momento de reinicio de la base de datos también cuando se reinician los contadores de *una sola* tabla (`pg_stat_reset_single_table_counters`), así que los contadores de las demás tablas pueden ser anteriores. Un reinicio borra además los momentos del último vacuum y analyze de una tabla, así que una tabla con un momento anterior al reinicio de la base de datos no se reinició con ella: recibe **sin tasa** en lugar de una cifra inflada por la ventana corta, y la sección *Autovacuum cost and I/O* cuenta esas tablas en una línea. Una ventana de menos de una hora da una proyección, mostrada sin color. Un sexto veredicto, **past**, es una tabla frecuente en promedio cuyo último vacuum fue hace al menos diez de sus intervalos medios y tres horas: el promedio viene de actividad anterior, así que no se marca nada ni se retiene ninguna recomendación por ello; las instantáneas AWR miden la tasa actual por ventana. En PostgreSQL 15 y posteriores, una base de datos creada después del inicio de las estadísticas del servidor se divide por la ventana más larga, así que su tasa puede salir baja, nunca alta.

Autovacuum visita cada base de datos una vez por `autovacuum_naptime`, así que puede limpiar una tabla como mucho con esa frecuencia; una tabla limpiada en la mayoría de esas visitas vence casi cada vez, y el hallazgo lo dice. Cuando los workers de autovacuum están saturados - en PostgreSQL 18 ocupados **al menos el 50% de su capacidad y 6 horas-worker al día** (aviso desde **80% y 12 horas**), medido con el tiempo que las tablas de esta base pasaron en autovacuum y autoanalyze, un límite inferior porque otras bases suman lo suyo; en 19 también todos los workers ocupados mientras hay tablas pendientes - las tablas pendientes esperan su turno, y el consejo es más workers o un freno más suelto, no otros umbrales.

Cada hallazgo termina diciendo cuán sólidas son sus cifras: PostgreSQL 18 y posteriores miden la duración de cada tabla; 16 y 17 solo miden la I/O del clúster; 14 y 15 lo estiman todo por tabla; en 13 el hallazgo es solo la frecuencia.

La sección **Maintenance actions** de la pestaña Advisor también muestra cualquier VACUUM / ANALYZE / REINDEX / VACUUM FULL que se esté ejecutando **ahora mismo** sobre esa tabla, leído del servidor por id de relación - así que aparece tanto si lo inició la app, como otra herramienta, o si la ventana se acaba de abrir. Cada uno muestra progreso en vivo desde `pg_stat_progress_*` y un **Stop** que lo cancela en el servidor; detener una operación que inició otra persona pide confirmación primero. En PostgreSQL 12+ va primero el más seguro `REINDEX CONCURRENTLY` (que no toma un bloqueo `ACCESS EXCLUSIVE`); por debajo de 12 se muestra deshabilitado con el motivo.

**Según el historial guardado (AWR y ASH).** Todo lo anterior lee los contadores desde el último reinicio de estadísticas: una media de todo ese tiempo. Cuando hay instantáneas AWR guardadas de la base de datos conectada, `db-vacuum` y la pestaña Advisor también las leen: por tabla, las filas actualizadas, borradas e insertadas en cada ventana entre dos instantáneas, cuántas veces corrió vacuum y a qué distancia estaba la tabla de su disparador en cada instantánea, con los ajustes vigentes entonces (las instantáneas de una versión anterior no tienen estas cifras). A partir de la tasa media dimensionan un disparador de más o menos **una ejecución por hora**, nunca más holgado que el 20% de la tabla. El disparador actual se **ajusta** cuando es al menos el **doble**, deja esperar al menos **10.000** filas muertas y la tabla cambia al menos 500 filas por hora; se **relaja** cuando es como mucho la mitad en una tabla con vacuum cada hora o más a menudo (al menos 10 ejecuciones en el historial); lo que quede dentro de un factor dos **encaja**. Desde PostgreSQL 13 una tabla con sobre todo inserciones (el 90% de sus escrituras) recibe el par de inserción de la misma forma. Estos valores sustituyen a la heurística por tamaño en la recomendación que llevan **Apply** y la sentencia ALTER, y el motivo junto a ellos nombra las tasas y la ventana más activa. Una tabla que estuvo por encima de su disparador al menos **una hora en dos o más instantáneas** se señala (un aviso a partir de **seis horas**), y entonces no se ofrecen umbrales nuevos: no es el disparador lo que decide cuándo se hace vacuum. Cuando hay muestras ASH guardadas, dicen por qué: un worker de autovacuum en la tabla **esperando un bloqueo** (al menos el 10% de sus muestras y al menos 5, o 60 muestras por sí solas), con la sesión que lo mantenía; dormido en su **freno de coste** (`VacuumDelay`: al menos la mitad de sus muestras y al menos 30, o 600 por sí solas); o **todos los workers de autovacuum ocupados a la vez** (al menos la mitad de las muestras con un worker trabajando y al menos 60, o 1.800 por sí solas). Con menos de una hora de historial no se dimensiona nada. Un bloque **Evidencia** lista las instantáneas y ventanas usadas, de cuándo a cuándo, y los días de ASH, o dice que no hay historial y que el consejo es el de siempre.

`db-table-stats` son los contadores de escaneos secuenciales frente a por índice por tabla, más `last_seq_scan` / `last_idx_scan` en PostgreSQL 16+. Una tabla con millones de escaneos secuenciales y un contador que sube se está escaneando en producción ahora - contrástala con `db-index-advisor`.

#### Secuencias, checkpoints y tablespaces {#sequences-checkpoints-and-tablespaces}

`db-sequences` y `db-seq-usage` listan las secuencias por lo cerca que están de su máximo, con la columna propietaria y cada default que las referencia. Una secuencia `integer` al 80% es un cambio a `bigint` que quieres planificar, no descubrir cuando `nextval` falla.

El máximo que cuenta es el menor de dos: el `MAXVALUE` propio de la secuencia y el mayor valor que cabe en la columna que alimenta (32.767 para `smallint`, 2.147.483.647 para `integer`). Antes de PostgreSQL 10 una secuencia no tenía tipo - todas eran `bigint` -, así que cada `serial` creado entonces es una columna `integer` alimentada por una secuencia `bigint`, y un esquema actualizado conserva esa forma. Medida solo contra la secuencia, esa columna marca un 0% de uso cuando le quedan unos miles de ids; la siguiente inserción más allá del borde falla con *integer out of range*. `db-sequences`, `db-seq-usage` y la fila SEQUENCES del resumen de salud miden contra la columna y califican por dos cosas, basta cualquiera:

- **la parte del rango usada**: ámbar desde el 80%, rojo desde el 95%;
- **el tiempo que queda**: los ids restantes divididos por las inserciones al día en la tabla de la columna (su contador de inserciones en la ventana de estadísticas - la misma ventana que indica el Advisor de tablas): ámbar por debajo de 180 días, rojo por debajo de 30. Una ventana de menos de una hora convierte el ritmo en una proyección: el tiempo se muestra, se dice que lo es y no cambia el color.

Cada secuencia que se agota nombra las columnas de clave foránea que apuntan a su columna y son más estrechas que `bigint`: pasar la columna a `bigint` traslada el problema a ellas, así que cambian en la misma migración. Cuando la columna ya es `bigint`, una columna de clave foránea más estrecha es el techo: el primer id por encima de su máximo no se puede referenciar, así que el espacio restante se mide contra ella y se nombra. `ALTER COLUMN ... TYPE bigint` reescribe toda la tabla bajo un bloqueo `ACCESS EXCLUSIVE` - nadie la lee ni la escribe hasta que termina; el camino sin corte es una nueva columna `bigint` mantenida al día por un trigger, un relleno por lotes y un intercambio.

`db-seq-sync` escribe un `SELECT setval(...)` para cada secuencia, dejando cada una justo por encima del mayor valor que hay ahora en su columna propietaria. Ejecútalo tras un cambio de logical replication o una carga con `pg_dump --data-only`, donde las filas llegan pero las secuencias se quedan en 1 y el siguiente `INSERT` choca con una clave existente. Es un script generado como cualquier otro - revísalo y luego ejecútalo en el destino.

`db-checkpoints` muestra cómo se disparan los checkpoints: **temporizados** (bien, guiados por `checkpoint_timeout`) frente a **solicitados** (forzados por la presión de `max_wal_size`), el intervalo medio entre ellos y el tiempo de escritura y sincronización. Una proporción alta de solicitados significa que `max_wal_size` es demasiado bajo para el ritmo de escritura - subirlo cambia espacio en disco por mucho menos E/S de checkpoint.

`db-tablespaces` lista cada tablespace con ubicación, propietario, tamaño, opciones (overrides de `seq_page_cost`, `random_page_cost`, `effective_io_concurrency`) y las bases que lo usan, más el contenido de cada tablespace en la base conectada. La sección de contenido por base está limitada a la conexión actual por la misma razón que los tamaños de esquema - el catálogo que asocia objetos a tablespaces es por base de datos. Compruébalo antes de mover un objeto grande, para confirmar que el destino tiene sitio y está en el disco que esperas; un `MOVE` escribe una segunda copia completa antes de descartar la primera, así que el destino necesita espacio libre igual al tamaño del objeto.

#### Margen de particiones {#partition-runway}

`db-partition-runway` (Almacenamiento) lista cada tabla particionada de la base conectada: su estrategia y clave, cuántas particiones, dónde termina el último rango y cuánto falta para eso, la partición DEFAULT y las filas que contiene, y si pg_partman gestiona la tabla. Existe por el incidente que ocurre a medianoche del día 1: una tabla particionada por mes cuya siguiente partición nadie creó. Desde el momento en que termina el último rango, un INSERT para "ahora" falla con *no partition of relation found for row* o, si hay una partición DEFAULT, acaba en ella - y una partición DEFAULT con filas de un rango impide más tarde crear la partición de ese rango (*updated partition constraint for default partition would be violated*) hasta que las filas se sacan a mano.

- Una tabla es **ámbar** cuando su último rango termina dentro de los días que se le dan a la comprobación (7 por defecto, y Salud usa 7), y **roja** cuando ya terminó. El margen se mide para una clave RANGE cuya primera columna es `date`, `timestamp` o `timestamptz`, también cuando la clave es una expresión como `(created_at::date)`; una clave numérica (un rango de ids) no tiene calendario que medir, y la fila lo dice. Un último rango hasta `MAXVALUE` nunca se agota.
- **Las filas en una partición DEFAULT** son ámbar en una tabla RANGE, sea cual sea su número - una fila basta para impedir la partición que debería contenerla - y azules en una tabla LIST, donde un cajón de sastre es un diseño habitual. La fila da el número y la proporción de la tabla.
- **pg_partman**: si la extensión está instalada en la base, la comprobación lee su configuración - el premake y el intervalo de cada tabla que gestiona - y qué ejecuta su mantenimiento (el worker `pg_partman_bgw`, un trabajo de `pg_cron`). Una tabla que gestiona pg_partman es ámbar cuando su margen ha caído por debajo de lo que su premake mantiene por delante: una tabla diaria con premake 4 siempre tiene de tres a cuatro días, así que el aviso de 7 días nunca callaría para ella. El mantenimiento no se está ejecutando entonces, o - con `infinite_time_partitions = off` - solo crea particiones futuras cuando han llegado filas nuevas.
- Una tabla cuyo último rango terminó hace tiempo y que ya no recibe filas nuevas (un archivo) también es roja: el catálogo no distingue un archivo de una tabla cuyas inserciones fallan, y la fila lo dice.

Nada recorre una tabla: los límites salen del catálogo y los recuentos de filas de las estadísticas del planificador, así que son estimaciones.

#### Salud de la replicación {#replication-health}

`db-replication` es la vista del primario sobre sus réplicas: el estado de cada standby y el retraso en bytes y en tiempo en las fases write, flush y replay. El retraso de `replay` es lo que una réplica de lectura sirve realmente por detrás; un retraso creciente de `write`/`flush` apunta a la red o al disco del standby, un retraso creciente de `replay` a una consulta larga en el standby que retiene el replay. En un standby sin réplicas propias aguas abajo la comprobación simplemente lo dice.

`db-replica-identity` encuentra tablas en las que la replicación lógica se romperá: sin clave primaria, o con una `REPLICA IDENTITY` no predeterminada. Una tabla sin PK que está **en una publicación** es CRÍTICA - el siguiente `UPDATE` o `DELETE` falla en el suscriptor - mientras que una que no lo está es solo informativa. Escribe el `ALTER TABLE` por prioridad: clave primaria, si no `REPLICA IDENTITY USING INDEX`, si no `FULL`.

`db-repl-slots` lista los slots físicos y lógicos con su marca `active` y el WAL que cada uno retiene. **Un slot inactivo llena el disco** - retiene WAL para siempre mientras nada lo consume. Elimina los slots que dejó una réplica retirada.

Crear un slot **lógico** espera a que termine cada transacción que tiene un id de transacción cuando empieza - en cualquier base de datos del clúster, incluida una transacción preparada olvidada. El diálogo Create slot lista esas transacciones antes de crearlo, y la creación espera como máximo el tiempo que elijas (30 segundos si no eliges otro); cuando se agota, el slot no se crea y la respuesta nombra lo que seguía esperando. Una transacción de solo lectura no lo retiene. Sin `pg_read_all_stats` las sesiones de otros usuarios solo se ven en parte, y el diálogo avisa de que la lista puede estar incompleta.

El chip **REPLICA** de la cabecera en un standby dice *al día* cuando todo lo recibido se ha reproducido, por mucho tiempo que haga desde la última confirmación en el primario - el tooltip da ese tiempo como el que el primario lleva en calma. Un retraso en segundos solo aparece mientras queda WAL por reproducir; un standby sin receptor de WAL (recuperación desde archivo) no muestra cifra de retraso, porque no puede saber si llega más.

`db-logical-replication` es el lado del suscriptor: el retraso de `pg_stat_subscription` y el estado de sincronización por tabla de `pg_subscription_rel`. Una tabla atascada en `COPY` o `SYNC` suele significar que la copia inicial encontró un error - revisa el log del suscriptor.

#### El vigilante de errores {#the-errors-watcher}

`db-errors` muestrea `pg_stat_database` durante una ventana de cinco segundos e informa la tasa por segundo de rollbacks, deadlocks y conflictos, más `sessions_fatal` / `killed` / `abandoned` en PostgreSQL 14+. En un clúster autogestionado donde `pg_read_file` está permitido también lee los últimos 256 KB del log del servidor - el archivo `jsonlog` (PostgreSQL 15+) o el `csvlog` cuando el servidor escribe uno, si no el log simple - y lee las entradas `ERROR` / `FATAL` / `PANIC` como entradas, cada una con sus líneas `DETAIL`, `HINT`, `CONTEXT` y `STATEMENT`. En una nube gestionada omite el log e indica dónde lo guarda el proveedor.

Del log informa:

- **Errores por SQLSTATE:** cada clase con su cantidad, el minuto más cargado, primera y última aparición, cuántos orígenes y el principal, y qué significa para las clases que tienen que ver con concurrencia y reintentos - `40P01` deadlock, `40001` fallo de serialización, `23505` violación de unicidad, `55P03` bloqueo no disponible, `57014` sentencia cancelada, `42P10` `ON CONFLICT` sin índice único correspondiente, `53300` demasiadas conexiones, `25P02` sentencias enviadas a una transacción abortada.
- **Errores por minuto**, por SQLSTATE.
- **Tormentas de reintentos:** el mismo SQLSTATE desde el mismo origen **10 veces o más en un minuto** - algo reintenta en bucle. Una cantidad absoluta, no una proporción: un bucle de reintentos es un problema en un servidor tranquilo y en uno cargado. El origen es la aplicación cuando el log la nombra (`jsonlog`, `csvlog` o `%a` en `log_line_prefix`), si no la forma de la sentencia.
- **Deadlocks, ambos lados:** para cada deadlock, cada proceso con su pid, qué esperaba, quién lo retenía, la sentencia que ejecutaba y cuál canceló PostgreSQL.

El log simple lleva lo que pone `log_line_prefix`; la comprobación lee el prefijo del servidor y encuentra `%e` (el SQLSTATE), `%a`, `%u` y `%d` dondequiera que estén. Sin `%e` el SQLSTATE se lee del texto en inglés del mensaje y la comprobación lo dice: añade `%e` (y `%a`) a `log_line_prefix`, o usa `jsonlog` o `csvlog`.

Lee primero las tasas: una tasa constante de deadlocks es un error de orden de bloqueo en la aplicación; un pico de conflictos en una réplica son consultas que chocan con el replay. El log es el detalle, los contadores son la alarma.

#### Los asesores de configuración y ajustes clave {#the-setup-and-key-settings-advisors}

`db-setup-advisor` comprueba todo lo del servidor de lo que dependen las comprobaciones de DBApguin - las extensiones `pg_stat_statements`, `auto_explain`, `pg_buffercache` y `pg_repack`, GUCs como `track_io_timing`, `compute_query_id`, `log_lock_waits` y los permisos `pg_monitor` / `pg_read_server_files` que necesita un no-superusuario - e informa de cada uno OK / PARCIAL / FALTA con la solución exacta. **Ejecútalo primero contra un clúster nuevo** y de nuevo tras cualquier cambio de configuración. En una nube gestionada el consejo dice «defínelo en el grupo de parámetros» en lugar de `ALTER SYSTEM`.

`db-settings` es una guía curada de ~70 parámetros relevantes para el tuning con una regla general y una insignia de aviso/peligro cada uno. Es una referencia, no una alarma: una insignia significa «este valor es inusual», no «esto está roto». Estos parámetros se editan desde la vista Servidor. Cinco tarjetas tratan de si los datos confirmados sobreviven a una caída y dan el mismo veredicto que la fila DURABILIDAD del resumen de salud: `fsync` y `full_page_writes` apagados son peligro, `synchronous_commit = off` es info (un intercambio deliberado, no un fallo), `data_checksums` (solo lectura) apagado es info, y `synchronous_standby_names` muestra cuántos de los standbys síncronos que pide están conectados - peligro mientras los commits esperan a uno que no lo está.

**Calculado para este servidor** abre Key settings: los quince parámetros que un ajustador como DBtune cambia sin reiniciar (`work_mem`, `maintenance_work_mem`, `effective_cache_size`, `random_page_cost`, `seq_page_cost`, `effective_io_concurrency`, `default_statistics_target`, los tres límites de workers paralelos, `max_wal_size`, `min_wal_size`, `checkpoint_completion_target`, `bgwriter_delay`, `bgwriter_lru_maxpages`), cada uno con un valor calculado para esta máquina, la medición o fórmula en la que se basa, reload o reinicio, y **Aplicar**. La RAM y las CPU salen de `/proc` y de los límites de cgroup cuando el servidor puede leer archivos (un superusuario o `pg_read_server_files`); un contenedor muestra las del host, así que introduzca lo que el servidor puede usar y pulse **Recalcular**. El disco se supone SSD salvo que elija HDD. Una medición decide el nivel siempre que la hay: `work_mem` solo es un aviso cuando los sorts se vuelcan de verdad (al menos 100 archivos temporales Y 1 GB desde el reinicio de las estadísticas), `max_wal_size` cuando al menos una quinta parte de los checkpoints Y cinco de ellos los forzó el volumen de WAL (o a partir del ritmo de WAL: tres `checkpoint_timeout` de WAL), el background writer cuando los backends escribieron ellos mismos al menos una cuarta parte de los buffers sucios Y 10.000 de ellos. **Aplicar los N a revisar** ejecuta exactamente las sentencias listadas tras una confirmación y vuelve a leer el servidor. Son puntos de partida para la máquina, no una medición de su carga: cambie algunos, observe las mismas cifras y quédese con lo que ayudó. La ventana DBApguin setup advisor (Ayuda) muestra el número con un botón a esta sección.

**Con historial lee lo que hizo el servidor, no solo lo que tiene.** Cuando ASH o los snapshots AWR programados han funcionado para este servidor (desde cualquiera de sus bases de datos), la sección empieza con **En qué se basa**: los días y muestras de ASH, cuántas sesiones estuvieron realmente activas a la vez (p95 y pico), la parte del tiempo activo esperando a la E/S y la hora más ocupada; las ventanas de AWR, lo volcado por día, los checkpoints forzados, el WAL de la ventana más ocupada frente a la media, el tiempo medio de lectura y el peor acierto de caché; las sentencias que vuelcan y cuánto escribe una llamada; y lo que se ejecuta ahora. La recomendación los usa donde son más fuertes que una regla: el techo de `work_mem` cuenta las sesiones que ASH vio a la vez (el doble del p95 o el pico) en lugar de todas las conexiones, pero solo sube `work_mem` hasta lo que necesitan las sentencias que vuelcan, el doble de lo que una llamada escribió a disco; una sentencia que necesita más de lo que permite la RAM aparece con su propio `SET work_mem`. `max_wal_size` sigue a la ventana AWR más ocupada, el tipo de disco al tiempo de lectura medido (`pg_stat_io` con `track_io_timing`), `effective_io_concurrency` y `shared_buffers` (reinicio) pasan a aviso cuando ASH muestra una quinta parte del tiempo activo Y 600 muestras esperando a la E/S, o AWR un acierto de caché por debajo del 99 % en al menos un millón de lecturas. Sin historial lo dice y ofrece **Iniciar ASH** e **Iniciar snapshots AWR** (cada 15 minutos); cuanto más haya, más cerca está la recomendación de esta carga. Los snapshots AWR ahora también registran `pg_stat_wal` (PostgreSQL 14+) y `pg_stat_io` (16+); a los snapshots anteriores simplemente les falta.

#### ¿Listo para PostgreSQL 19? {#ready-for-postgresql-19}

**Ajuste y diagnóstico -> ¿Listo para PostgreSQL 19?** pregunta a este
servidor por cada punto de la lista de incompatibilidades de PostgreSQL 19 y
responde a cada uno:

- **BLOCKS** - pg_upgrade se niega o 19 no arranca: una base de datos en la
  codificación eliminada MULE_INTERNAL, un salto de línea en el nombre de una
  base de datos, rol o tablespace, un índice con las clases de operadores
  inet/cidr de btree_gist, RADIUS en pg_hba.conf, escape_string_warning
  configurado en algún sitio.
- **CHANGES** - la actualización pasa y algo se comporta distinto:
  standard_conforming_strings desactivado, contraseñas MD5 (en 19 cada inicio
  de sesión avisa), max_locks_per_transaction fijado a mano (19 necesita el
  doble para la misma capacidad), JIT en uso (desactivado por defecto en 19),
  postgres_fdw (las transacciones de solo lectura pasan ahora al lado remoto).
- **INFO** - extensiones de terceros que necesitan una versión para 19 antes
  de pg_upgrade, y herramientas de monitorización que leen columnas
  renombradas.

Varios puntos necesitan superusuario; sin él dicen «no se pudo comprobar» en
lugar de dar todo por bueno. El punto de índices lee la base de datos a la que
estás conectado, así que ejecuta la comprobación en cada base de datos. En un
servidor que ya ejecuta 19, las mismas filas muestran lo que aún difiere de
los valores por defecto de 19.

#### Observabilidad al estilo Oracle

DBApguin incluye comprobaciones que hacen para PostgreSQL lo que en Oracle hacen AWR / ASH / SQL Trace:

**ASH y AWR tienen una pestaña cada uno.** El menú *Ajuste y diagnóstico* abre *Historial de sesiones activas (ASH)* e *Instantáneas e informes AWR*, una pestaña por servidor. La cabecera de la pestaña muestra qué hace el recolector (muestrea o toma instantáneas, desde cuándo, cada cuánto, qué conserva; en ámbar y *en pausa* mientras el servidor es inaccesible) con **Iniciar**, **Detener** y los ajustes tras el engranaje; las vistas son la subnavegación de la pestaña (ASH: En vivo, Informe, Días guardados; AWR: En vivo, Instantáneas, Informe, Comparar, Captura, Tendencia), y las entradas de una vista, la ventana y la base de datos del informe o las dos instantáneas a comparar (cuatro para dos periodos), están sobre el resultado en lugar de en un diálogo. Una fecha se elige con el selector de fecha y hora, cuyo calendario subraya los días con muestras. Los parámetros `action` descritos abajo son lo que ejecutan las vistas; siguen funcionando desde scripts y enlaces antiguos y abren la misma pestaña. Al conectar con un servidor cuyo muestreo o programador está en marcha, su pestaña se abre sola.

**db-workload-report (captura en vivo al estilo AWR)** - la ejecuta AWR "Durante un tiempo", hasta una hora, con *también muestrear las sesiones cada 5 s* (su resultado sigue apareciendo en la pestaña AWR bajo *Captura*): una ventana (10 s a 60 min, por defecto 5 min) y un intervalo de muestreo. Toma al principio la misma instantánea que toma una instantánea AWR, muestrea pg_stat_activity cada pocos segundos entre medias, toma la segunda instantánea al final y lee la ventana exactamente como un informe AWR lee dos instantáneas: **Resumen de la ventana** (con las muestras, el pico de sesiones activas y de todas, y cuántos usuarios y aplicaciones se vieron), **Hallazgos** con su evidencia, **SQL principal a revisar** con el porqué de cada una, luego **Sesiones activas a lo largo del tiempo** (un gráfico de las muestras por clase de espera), **Eventos de espera**, **Pares de bloqueo observados**, **Transacción abierta más larga vista**, las clasificaciones de SQL principal, las tablas por DML, la actividad de fondo y de checkpoints y los ajustes para un mejor análisis. Vea [Cómo leer un informe AWR / de carga](#reading-an-awr-report). Úsalo cuando la carga está ocurriendo ahora mismo y quieres un informe de ella; el informe se guarda en [Informes guardados](#saved-reports).

**db-awr-snapshot (almacén persistente de instantáneas + comparación arbitraria + planificador)**: seis acciones mediante el parámetro `action`. Tres manuales, tres para el planificador.

  Acciones manuales:

  - action=take: captura pg_stat_statements + pg_stat_database + pg_stat_user_tables + pg_stat_user_indexes + la configuración de autovacuum + bgwriter / checkpointer en un archivo JSON bajo `$DBCHECKS_HOME/awr_snapshots/<huella-del-servidor>/snap-<id>.json`. Se guarda además un comentario opcional. La mayor parte de una instantánea es la copia de pg_stat_statements; las filas por tabla y por índice que leen los consejos de autovacuum e índices añaden unos 320 bytes por tabla y 160 bytes por índice, así que una base con mil tablas añade alrededor de medio megabyte a cada instantánea. La huella es `host__puerto__base`, así que las instantáneas de clústeres distintos nunca se mezclan. El conjunto de columnas sigue a la versión del servidor - `pg_stat_database.sessions_*` desde PostgreSQL 14, `pg_stat_checkpointer` desde 17, antes `pg_stat_bgwriter` - de modo que una instantánea es válida desde PostgreSQL 13, y comparar dos versiones mayores distintas simplemente omite las columnas que le faltan a una de ellas.
  - action=list (**Snapshots guardados**): cada snapshot guardado para este servidor - día, hora (UTC), comentario, sentencias, tamaño - con **Comparar con el anterior** en cada fila, **Comparar los dos elegidos**, **Exportar** (los elegidos, o todos, a un zip donde diga el diálogo de guardar), **Importar...**, **Borrar** los elegidos y **Borrar todo lo anterior a N días**. Borrar pregunta antes. Trabaja solo con los archivos, así que el historial de un servidor apagado se puede seguir viendo, limpiando o llevando a otro sitio.

    **Exportar e importar.** Una exportación es un zip: los archivos tal cual y un pequeño manifiesto que nombra ASH o AWR y el servidor. La importación los devuelve a ese servidor - conéctate a él para ver sus informes -, acepta solo los nombres de archivo propios del almacén (un zip con cualquier otra cosa se rechaza entero, sin escribir nada) y nunca sobrescribe un archivo que ya está: dice cuáles conservó.
  - action=compare: compara dos instantáneas cualesquiera por id y muestra las mismas secciones al estilo AWR. La «ventana» puede abarcar horas o días porque los datos están en disco. Toma una antes de un despliegue / lote / migración, otra después, y compara cuando quieras.

    **A través de un reinicio.** Un reinicio limpio conserva los contadores de PostgreSQL - los escribe al apagarse y los vuelve a leer al arrancar -, así que una comparación que lo cruza cubre toda la ventana y solo dice que el servidor se reinició. Una caída, una recuperación al arrancar o un failover los ponen a cero, igual que pg_stat_reset(). Cada snapshot registra cuándo arrancó el servidor; donde un contador es menor en el snapshot más reciente, la comparación lo cuenta desde cero (el valor nuevo es lo ocurrido desde el reinicio) en vez de mostrar un número negativo, empieza con un hallazgo que lo dice y, tras una caída, toma la ventana desde el arranque del servidor. La tendencia hace lo mismo en cada ventana. El AWR de Oracle tiene el mismo límite - sus contadores vuelven a empezar con la instancia y rechaza un informe que cruce un reinicio; lo que ambos conservan es la serie de snapshots.

  - action=compare-periods (la vista **Comparar**): dos periodos uno al lado del otro, como los presenta el informe *Compare Periods* de Oracle - el periodo 1 de la instantánea A1 a B1, el periodo 2 de A2 a B2; la instantánea más antigua de cada par se toma como su inicio. Cada periodo se calcula exactamente como compare calcula su ventana, también a través de un reinicio o un restablecimiento, con la misma elección de **Base de datos**. Abierta sin elegir nada, la vista toma las tres instantáneas más recientes como dos ventanas consecutivas - la tercera más reciente a la segunda, luego la segunda a la más reciente - y las compara; con menos de tres dice que una comparación necesita tres y no ejecuta nada. Los periodos pueden solaparse o venir en cualquier orden; el informe lo dice. Vea [Cómo leer una comparación de periodos](#reading-a-period-comparison).


  Acciones del planificador (el equivalente del AWR automático que en Oracle mueve `DBMS_SCHEDULER`):

  - action=auto-start: lanza un hilo en segundo plano que toma una instantánea cada `interval_minutes` (por defecto 60, rango 5-1440) y borra solo las anteriores a `retention_days` (por defecto 30, rango 1-365). El hilo usa su propia conexión psycopg2, así que nunca deja sin recursos al pool del usuario. Es idempotente (volver a ejecutarlo actualiza los parámetros del hilo vivo). La primera instantánea se toma de inmediato: `auto-start` espera su resultado e informa de lo que ocurrió de verdad, así que un planificador que no puede tomar ni una instantánea lo dice en vez de «arrancado». Si esa primera instantánea falla, el hilo no se queda arriba. Una vez en marcha, un servidor que deja de responder no lo termina: tras tres instantáneas fallidas seguidas el planificador se pausa, sigue intentando reconectar (cada 30 segundos, luego con menos frecuencia) y continúa solo en cuanto el servidor vuelve; el panel en vivo muestra la pausa en ámbar.
  - action=auto-stop: detiene el hilo del planificador. `auto-stop` informa de lo que capturó mientras corría; si nunca tomó una instantánea, lo dice con claridad en vez de «los archivos se quedan en disco». La marca `enabled` guardada en `auto_config.json` pasa a false.
  - action=auto-status: encabeza con el veredicto - sano, en pausa, fallando, abortado o sin ejecutarse - y luego el estado del hilo vivo (intervalo, retención, número de instantáneas, id y hora de la última, archivos podados en total, último error si lo hubo) y la configuración guardada. Tras reiniciar la aplicación el planificador vuelve solo la próxima vez que abras ese servidor; auto-start solo hace falta si lo detuviste.

  Un planificador automático en marcha también aparece en la lista **Background jobs**, igual que el muestreador ASH, para ver de un vistazo si está sano sin abrir la comprobación.

  **Planificador, en vivo** (en `auto-status` y tras `auto-start`) es el mismo panel que tiene ASH y se actualiza cada 5 segundos mientras la pestaña está abierta: el estado, el intervalo y la retención, cuánto lleva en marcha, los snapshots tomados, hace cuánto fue el último (en amarillo cuando lleva medio intervalo de retraso), cuándo toca el siguiente, lo que hay en disco y de cuándo a cuándo, y lo que la retención ha eliminado. El diálogo muestra solo los campos que usa la acción elegida.

  Sobre la persistencia: la configuración del planificador se guarda junto a las instantáneas y sobrevive a los reinicios. Un planificador que estaba en marcha al cerrar la aplicación no arranca al inicio (entonces no se descifra nada); vuelve en el momento en que abres de nuevo ese servidor en la aplicación, con la conexión con la que lo abres y el intervalo y la retención guardados. Hasta entonces `auto-status` lo muestra como activado pero sin ejecutarse. `auto-stop` es lo que lo apaga del todo.

**db-ash (muestreador del historial de sesiones activas)**: muestreador continuo en segundo plano. Cuatro acciones.

  - action=status: encabeza con el veredicto - en marcha y sano, en pausa (el servidor dejó de responder; sigue reconectando y continúa solo), fallando o detenido - y luego desde cuándo, cuántas muestras y filas lleva, el último error si lo hubo, y el inventario en disco.
  - action=start: lanza un hilo demonio que consulta pg_stat_activity cada **interval** segundos (1 por defecto; 2, 5, 10, 30 o 60) en una conexión propia (separada del pool del usuario), descarta los backends inactivos y añade cada fila activa a un archivo JSONL diario bajo `$DBCHECKS_HOME/awr_snapshots/<huella>/ash/ash-YYYY-MM-DD.jsonl`. Es idempotente (repetirlo no hace nada). El conjunto de columnas por muestra sigue a la versión del servidor - `query_id` es de PostgreSQL 14+, así que en 13 simplemente no se registra y el informe lo señala una vez en lugar de fallar.
  - **interval** y **retention days** se eligen al iniciar y se guardan por servidor para el siguiente inicio. La retención es de 30 días salvo que elija otra, y 0 lo conserva todo; los archivos diarios más antiguos que la retención se eliminan al iniciar el muestreador y una vez por hora mientras funciona - solo días enteros, nunca parte de uno. Un intervalo más largo escribe menos y se pierde esperas cortas; un segundo es lo que hace un ASH clásico. El muestreador conserva los ajustes con los que arrancó: iniciarlo de nuevo con otros valores lo dice y no cambia nada hasta que se detenga.
  - action=stop: detiene el muestreador e informa de lo que capturó. Si nunca se escribió nada, lo dice, en vez de «los archivos JSONL se quedan en disco». Tras tres muestras fallidas seguidas el muestreador se pausa en vez de detenerse: sigue intentando reconectar (cada 30 segundos, luego una vez por minuto) y se reanuda solo cuando el servidor vuelve a responder; `status` y el panel en vivo muestran la pausa en ámbar con el último error. Un muestreador que dejaste en marcha también vuelve solo tras reiniciar la aplicación, la próxima vez que abras ese servidor; `stop` es lo que lo apaga.
  - action=report: agrega cualquier rango de fechas en el reloj propio del servidor en cuatro secciones ASH clásicas: histograma de estados, top 20 de eventos de espera, sesiones más frecuentes por número de muestras, consultas más frecuentes por número de muestras, pares de bloqueo observados. **Informe desde** e **Informe hasta** se eligen con el selector de fecha y hora de la aplicación: opciones rápidas (ahora, hoy 00:00, hace 1 h, hace 24 h, ayer), un calendario y campos de hora y minuto; Vaciar deja un límite abierto.
  - action=data (**Días guardados**): cada archivo diario guardado para este servidor con su tamaño, hoy marcado; **Informe** en una fila abre el informe de ese día, **Informe de los días elegidos** va del primero al último elegido. Exportar, importar y borrar como en los snapshots AWR guardados, más abajo.

  Un muestreador en marcha aparece en la lista **Background jobs** con su estado, para que un muestreador que se ha parado en silencio sea visible sin abrir la comprobación.

  **Muestreador, en vivo** (en `status` y tras `start`) se actualiza cada 5 segundos mientras la pestaña está abierta: el estado, el intervalo y la retención, cuánto lleva en marcha, muestras y filas escritas, cuántas sesiones estaban activas en la última muestra, hace cuánto fue (en amarillo cuando lleva tres intervalos de retraso), lo que hay en disco y cuántos días abarca, y lo que la retención ha eliminado.

  Espacio: una fila por sesión activa y muestra, unos 450 bytes con su texto de consulta. Medido con una muestra por segundo y dos sesiones activas de media: 2,8 MiB por hora, unos 67 MiB al día; con una muestra cada 5 segundos, la quinta parte. El disco crece con el número de sesiones activas y la longitud de sus sentencias, no solo con el tiempo, por eso la retención es de 30 días salvo que elija otra. En el servidor una muestra es una lectura de pg_stat_activity, unos 0,3 ms. JSONL mantiene el formato trivial de inspeccionar con `jq`.

**db-trace-session (trazado al estilo Oracle)**: muestreador por backend con tres modos de objetivo y dos filtros. *Ajuste y diagnóstico* > *Traza de sesión* abre su pestaña, una por servidor, con tres vistas. **En vivo** muestra lo que hace ahora el trazado en marcha: a quién sigue, cuánto lleva de su ventana, las muestras y las sentencias capturadas hasta ahora, cada sesión seguida con su estado, su espera y su sentencia, y las esperas más vistas; se actualiza cada dos segundos, la pestaña se abre en ella mientras hay un trazado en marcha para este servidor, y un trazado iniciado desde el formulario corre en ella y termina en Informe. **Nueva** es el formulario, sobre el resultado en lugar de en un diálogo: a quién seguir (cada sesión de un usuario, cada sesión cuyo application_name coincide con un patrón o un backend elegido de la lista de los conectados ahora), la ventana (30 s, 1, 2, 5, 10 o 30 minutos, o cualquier número de segundos de 10 a 1800; 2 minutos por defecto), la categoría de sentencias y el filtro del texto de la consulta, y luego **Iniciar traza**. Antes de empezar, la traza pregunta, como ASH y AWR, por las extensiones y los ajustes que lee cuando falta alguno que necesita, salvo que esa pregunta esté desactivada; la vista Nueva dice en una línea cuántos están en su sitio, y **Comprobar ajustes...** muestra la misma ventana en cualquier momento, cuyo Iniciar inicia la traza. Mientras dura, la cabecera dice a quién traza y cuánto de la ventana ha pasado, y **Detener** la termina antes: el informe cubre lo capturado hasta entonces. La pestaña muestra entonces **Informe**, con la fila de exportación (texto, CSV, JSON, PDF). El informe termina con **Ajustes para un mejor análisis**, la tabla con la que termina el informe ASH (ver [Cómo leer un informe ASH](#reading-an-ash-report)), encabezada por lo que decide cuánto ve una traza: `auto_explain` con `log_min_duration` y `log_analyze` (el plan que realmente se usó va al log del servidor), `log_min_duration_statement`, `log_lock_waits`, `compute_query_id` y `track_activity_query_size`. **Historial** lista las trazas guardadas para este servidor, las más recientes primero: cuándo empezó cada una, a quién siguió, la ventana y cuánto duró de verdad (las detenidas van marcadas), cuántas sentencias capturó y su tamaño. **Abrir** muestra una como su informe, de solo lectura y exportable en los mismos cuatro formatos; **Eliminar** la quita tras una pregunta. Las trazas se guardan por servidor hasta que se eliminan (las 500 más recientes), por muchas otras comprobaciones que se ejecuten mientras tanto. **Exportar todo**, o las elegidas, las escribe en un zip; **Importar...** lee un zip así y nunca sobrescribe una traza que ya está aquí; la exportación de datos de la app las lleva con las instantáneas AWR y los días ASH del servidor. Detener termina antes la *Captura* de la pestaña AWR de la misma manera.

  - target_mode=pid: traza un solo backend por su pid (elegido de la lista de los backends conectados ahora).
  - target_mode=application_name: traza todo backend cuyo application_name case con una expresión regular POSIX. Útil cuando el pool rota los pid pero el nombre de la aplicación no cambia.
  - target_mode=user: traza todo backend de un rol concreto. Para la pregunta «qué está haciendo ahora este usuario de informes».
  - duration_seconds: de 10 a 1800 segundos (por defecto 120). El muestreador va a 2 Hz durante toda la ventana.
  - category_filter: todas / dml / ddl / select / utility. Descarta las filas cuyo texto no encaje con la categoría elegida.
  - query_filter: expresión regular opcional sobre el texto, sin distinguir mayúsculas. Vacío = sin filtro.

  El trazado además comprueba si `auto_explain` está cargado e indica las órdenes SET exactas para activar la captura de planes en la sesión objetivo antes de que empiece la carga. Salida: secuencia de consultas (transiciones por tiempo), histograma de eventos de espera de los backends objetivo, diferencia por consulta desde pg_stat_statements. En PostgreSQL 13 la columna `query_id` aún no existe; el trazado funciona sin ella y lo indica.

### 5.2 SQL {#the-sql-editor}

Editor SQL con resaltado de sintaxis PostgreSQL y autocompletado propio. Barra de herramientas:

- Ejecutar (`Ctrl/Cmd + Enter`, `F5`): ejecutar todo lo del editor
- Sentencia (`Ctrl/Cmd + Mayús + Enter`): ejecutar solo la selección o la sentencia bajo el cursor
- Validar (`Ctrl/Cmd + E`): analizar y planificar sin ejecutar
- Explain: EXPLAIN / EXPLAIN ANALYZE / BUFFERS
- Exportar CSV: vuelve a ejecutar la consulta en el servidor, de modo que el
  archivo contiene todas las filas y no solo la página en pantalla
- Guardar como…: escribe el resultado que está viendo como JSON, JSON Lines,
  `INSERT` de SQL, tabla Markdown o texto alineado
- Limpiar historial

Bajo el editor está el panel de salida con:
- Rejilla de resultados (ver más abajo)
- Historial en vivo y de estadísticas (plegable)
- Tiempo y número de filas por ejecución

**Consultas guardadas.** **Save query** (Ctrl+S) guarda el texto del editor
con un nombre; la lista **saved** de al lado abre una en una pestaña. **SQL
→ Consultas guardadas…** (o *manage saved queries…* al final de esa lista)
abre su propio diálogo: una lista que se busca por nombre, nota y SQL, y
para la elegida su nombre, una nota y el SQL para editar, **Abrir en el
editor**, **Duplicar** y **Eliminar**. Un cambio se guarda solo después de
que **Revisar y guardar** lo haya mostrado lado a lado con lo guardado -
antes a la izquierda, después a la derecha, una línea que solo tiene un lado
en rojo o verde, las palabras cambiadas marcadas - y se pulse **Guardar
cambios**. Cada cambio guardado conserva la versión anterior (las últimas
25): **Versiones anteriores** compara cualquiera con la actual del mismo
modo y puede devolverla a los campos, para revisarla y guardarla. Una
pestaña abierta desde una consulta guardada lo recuerda: **Save query** allí
muestra el cambio lado a lado y ofrece **Actualizar** o **Guardar como
nueva**. Las consultas guardadas no tienen límite; la lista **history** de
al lado conserva las últimas 30 consultas ejecutadas. El espacio de trabajo
por conexión conserva tus pestañas abiertas entre reinicios.

#### EXPLAIN como árbol {#explain-as-a-tree}

Explain y Explain Analyze se abren como un **árbol** por defecto: una fila por nodo del plan, sangrada bajo su padre, cada una mostrando la estimación de coste del planificador, filas estimadas y (para Analyze) filas reales, el tiempo propio de este nodo y su tiempo total incluyendo a sus hijos, y el número de bucles (loops). Dos cosas se destacan automáticamente en vez de dejar que las busques tú en una pared de números:

- **más costoso** marca el único nodo con más tiempo *propio* - su propio tiempo con el de cada hijo restado. Esto es deliberadamente distinto de "el nodo con más tiempo total", que casi siempre es el nodo más externo simplemente porque incluye a todos los de debajo; el tiempo propio encuentra dónde se fue el tiempo realmente.
- **peor estimación** marca el nodo con la mayor diferencia entre lo que el planificador esperaba (`Plan Rows`) y lo que realmente salió (`Actual Rows`), en cualquier dirección. Una diferencia grande aquí, especialmente junto a un Seq Scan del que el planificador pensaba que devolvería poco, es la razón más común de que un plan salga mal - normalmente significa que las estadísticas de la tabla están desactualizadas (`ANALYZE` lo arregla) o que la cláusula WHERE tiene una correlación que `pg_stats` no puede ver.

Insignias por columna: `cost` (estimación del planificador, unidades arbitrarias, solo comparable dentro del mismo plan), `rows` (estimado → real, con un multiplicador "Nx" en cuanto el error es de 3x o más), tiempo `self` y `total`, y `×N loops` cuando un nodo bajo un Nested Loop se ejecutó más de una vez. Una breve línea de aviso aparece justo bajo el nodo al que se refiere - un Seq Scan grande, un Sort que se desbordó a disco, un Hash Join con más de un lote, un Nested Loop sobre un lado externo grande - las mismas categorías que ya usaban las insignias de resumen de arriba, ahora leídas de los propios campos del nodo en vez de adivinadas del texto impreso, y mostradas donde realmente está el problema en vez de en una lista separada debajo.

Un Seq Scan se juzga por las filas que **lee**, no por las que devuelve: con EXPLAIN ANALYZE son las filas que conserva más las que su filtro descarta, en todos los workers, y sin ANALYZE cuenta también un coste de escaneo alto. Cuando el filtro descarta casi todo, la línea dice cuántas filas se leyeron y cuántas se conservaron - el caso que resuelve un índice sobre la expresión filtrada. Solo se llama tabla pequeña a un escaneo que lee poco.

**Vista de texto** cambia a un bloque de texto plano y copiable - la misma información, con la forma de EXPLAIN de PostgreSQL, para pegar en un chat o un ticket. Se renderiza a partir del mismo plan analizado que lee el árbol (no una segunda llamada al servidor): EXPLAIN ANALYZE realmente ejecuta la consulta, así que pedirle a PostgreSQL el formato de árbol y el de texto por separado significaría ejecutar la consulta dos veces - coste real para una consulta lenta, y para una sentencia que modifica datos (envuelta en una transacción que se revierte de todos modos) el doble de escrituras antes del rollback. La contrapartida es que este texto es una reconstrucción cercana y legible, no garantizada byte a byte igual a lo que `EXPLAIN (FORMAT TEXT)` habría impreso de forma nativa - los números y la estructura son exactos, la salida real de PostgreSQL es lo único que los garantiza, solo la puntuación circundante se reproduce a mano.

#### La rejilla de resultados

Un `SELECT` trae las primeras 500 filas y las muestra al momento. Baja hasta
el final y llegan las siguientes 500, y así; la línea de estado dice
`500 de 48.213 filas`, para que siempre sepas dónde estás en el resultado. El
número se calcula una vez, por adelantado - en una tabla lo bastante grande
como para que contar tarde demasiado lo dice en lugar de un número, y la
paginación sigue funcionando.

Las filas se guardan en el servidor entre páginas, en un cursor que se cierra
en cuanto ejecutas otra cosa, cambias de pestaña o lo dejas sin tocar unos
minutos. El editor no mantiene una transacción abierta a tus espaldas.

Pasadas 5.000 filas en la rejilla, la carga automática se detiene y toma el
relevo un botón **Cargar 500 más**, para que un desplazamiento descuidado
sobre un resultado enorme no llene la ventana. Para el resultado completo,
`Exportar CSV` transmite cada fila en el servidor, sin pasar por la rejilla.

En la rejilla misma no hay orden ni filtro. Solo contiene una parte del
resultado, así que ordenarla ordenaría 500 de N y parecería que lo ha
ordenado todo. Ordena en la consulta, con `ORDER BY`.

#### El campo search_path

Junto al selector de modos, `search_path` fija dónde se buscan los nombres sin
cualificar, sólo para esa ejecución. Si lo deja vacío, rige el contexto de
sesión de la cabecera.

La flecha al final del campo despliega los esquemas de la base conectada - 
sólo aquellos en los que esta sesión puede entrar, porque un esquema sin
`USAGE` no es una respuesta. Los suyos primero, `pg_catalog` e
`information_schema` al final bajo su propio encabezado. Un clic en un nombre
lo añade, otro clic lo quita, y todo lo que ya está en la ruta aparece
marcado, así que el estado completo del campo se ve de una vez. El menú sigue
abierto mientras elige, y el campo se puede escribir a mano.

Importa más de lo que parece: un
esquema mal escrito en el search_path no es un error. PostgreSQL acepta una
ruta que nombra esquemas inexistentes y luego no encuentra su tabla, lo que se
lee como «la tabla ya no está» en vez de «la ruta está mal».

#### Qué ejecuta realmente «Ejecutar»

El editor nunca adivina un límite de sentencia que tú no ves. Dos botones, dos alcances:

| Botón | Atajo | Ejecuta |
|---|---|---|
| `▶ Ejecutar` | `Ctrl/Cmd + Enter` o `F5` | todas las sentencias de la pestaña, en orden |
| `▶| Sentencia` | `Ctrl/Cmd + Mayús + Enter` | el texto seleccionado, o - sin selección - la sentencia en la que está el cursor |

«La sentencia en la que está el cursor» va del `;` anterior al siguiente. Los
cuerpos entre signos de dólar (`$$ ... $$`), las cadenas y los comentarios quedan
fuera de esa búsqueda, así que una función cuyo cuerpo contiene
`BEGIN ... ; ... END` cuenta como una sentencia y no como cuatro.

`F5` recarga la página en un navegador; DBApguin lo intercepta. En la vista SQL
ejecuta, en el resto refresca la vista actual. `Ctrl/Cmd + R` sigue refrescando
en todas partes, también en la vista SQL.

#### Transacciones y autocommit {#transactions-and-autocommit}

El desplegable junto a Ejecutar decide qué ocurre alrededor de las sentencias de una ejecución:

- **autocommit** (por defecto) - cada sentencia confirma por su cuenta, como hace
  `psql`. Lo que ejecutaste es duradero en cuanto vuelve, y un fallo a medio
  camino deja aplicadas las sentencias anteriores.
- **transacción** - toda la ejecución va envuelta en un `BEGIN ... COMMIT`. Si
  falla cualquier sentencia, todo se revierte y la base queda exactamente como
  estaba. Este es el modo para una migración.
- **simulación** - la ejecución va envuelta en `BEGIN` y siempre termina en
  `ROLLBACK`. Obtienes recuentos de filas, errores y tiempos sin dejar rastro.
  Nada de lo que hagas se conserva, tampoco lo que salió bien.

**read-only** decide si las sentencias de esta pestaña se ejecutan en una
sesión de solo lectura. Está desactivado por defecto, por pestaña, y la
aplicación no adivina: activado, PostgreSQL rechaza todo lo que escribe, lo que
sirve para poner un script de migración ante un servidor de producción y ver
dónde se detendría.

Antes se deducía del propio SQL, y esa deducción no se puede hacer bien. La
primera palabra clave no dice si una sentencia escribe: `IMPORT FOREIGN SCHEMA`
no parece ninguna de las dos cosas, y `SELECT cron.schedule(...)` - tal como la
documentación de pg_cron indica programar un trabajo - escribe una fila con el
aspecto exacto de una consulta. Ambas volvían rechazadas con *«cannot execute
... in a read-only transaction»*, en un primario, por una restricción que la
propia aplicación había impuesto, mientras la misma sentencia pasaba sin
problema por `psql`. Se ejecuta tal como está escrito.

Algunas sentencias no pueden ejecutarse dentro de una transacción: `CREATE
INDEX CONCURRENTLY`, `REINDEX CONCURRENTLY`, `VACUUM`, `CREATE DATABASE`.
Ejecútelas en modo **autocommit**. En los otros dos el servidor responde
*«cannot run inside a transaction block»*, y el editor indica qué selector
cambiar.

- **ejecutar como script** - como autocommit, salvo que una sentencia que
  falla no interrumpe la ejecución. Se intentan todas, cada fallo se imprime
  con su número y su mensaje, y lo que pasó queda confirmado. Es el modo para
  un archivo de DDL del que ya se aplicó una parte: quiere que entre el resto
  y una lista de lo que no. El resultado es solo la salida en bruto, como la
  imprime `psql -f` - la etiqueta o las filas de cada sentencia, sus avisos,
  su error - con **Copy** al lado; no hay tabla.
- **mantener abierta (usted la cierra)** - la transacción sigue abierta
  **entre** ejecuciones y termina cuando pulsa Commit o Rollback, que aparecen
  junto al selector mientras está abierta. Ejecute algo, mire qué hizo y sólo
  entonces decida.

  Una transacción abierta retiene sus bloqueos hasta que alguien la cierra,
  así que su estado está siempre a la vista - cuántas sentencias lleva y
  cuánto tiempo lleva abierta - y la que nadie ha tocado en diez minutos se
  deshace por usted. Una sentencia que falla **no** la termina: PostgreSQL
  marca la transacción como abortada y nada más pasará hasta que haga
  rollback, que es justo lo que la aplicación dice en lugar de tirar su
  trabajo.

#### Cuánto puede durar una sentencia

Junto al selector de modos hay un tiempo límite: una casilla y un número de
segundos. Cada pestaña SQL nueva empieza con **180 segundos activados**, y el
ajuste pertenece a la pestaña - una sostiene una migración que está dispuesto
a esperar una hora, otra consultas que todavía está afinando.

Cuando salta, la sentencia se cancela **en el servidor**, no sólo en la
aplicación. Si quita la casilla, la sentencia corre hasta terminar o hasta que
usted la detenga; la aplicación lo pregunta una vez, porque una sentencia sin
límite retiene sus bloqueos todo el tiempo que dure.

`ADD CONSTRAINT`, `CREATE INDEX` y un `UPDATE` grande sobre una tabla real
duran todos más que los antiguos treinta segundos fijos, a los que esto
sustituye.

#### DDL, bloqueos exclusivos y la cola de bloqueo {#ddl-exclusive-locks-and-the-lock-queue}

`DROP`, `ALTER TABLE`, `TRUNCATE`, `REINDEX` (sin `CONCURRENTLY`), `CLUSTER` y
`VACUUM FULL` toman un **AccessExclusiveLock** sobre la tabla. Lo importante de
ese bloqueo es lo que pasa *antes* de que la sentencia lo consiga:

> Una petición de bloqueo exclusivo no se salta la cola. Mientras espera,
> PostgreSQL pone cada nueva petición de bloqueo sobre esa tabla detrás de ella
> - incluido un simple `SELECT`. Así que un `DROP` que solo está *esperando* ya
> bloquea nuevas lecturas y escrituras sobre esas tablas, antes de haber
> cambiado nada.

Así es como un `DROP SCHEMA` detrás de un informe lento se convierte en una
caída de todo el servidor. El editor trata estas sentencias de forma distinta
a una consulta normal:

- **Una comprobación previa antes de ejecutar.** Cuando la sentencia toma un
  bloqueo exclusivo y ya hay algo sobre el objetivo, un diálogo enumera esas
  sesiones - quién, desde dónde, cuánto tiempo y si alguna ya está en cola - y
  explica la espera antes de que te comprometas. Una sesión en el primer
  segundo de una sentencia, en una transacción no más antigua, no aparece: en
  una tabla con mucho uso siempre hay una (las sentencias cortas de la
  aplicación, el panel de esta aplicación leyendo tamaños de tablas) y termina
  antes de que el DDL lo note. Una transacción que se dejó abierta - en otro
  programa o en otra pestaña del editor de esta aplicación - aparece por joven
  que sea, igual que una sentencia que lleva más tiempo.
- **El bloqueo de cada sentencia, antes de ejecutarla.** La confirmación de ejecución lista cada sentencia DDL con el bloqueo que toma y sobre qué, y dice cuándo vuelve a escribir la tabla o la lee entera - preguntando al servidor donde el texto no lo dice: `ADD COLUMN ... DEFAULT clock_timestamp()` (cualquier valor por defecto volátil - `random()`, `gen_random_uuid()`, un `serial`) reescribe cada fila, mientras que un valor constante o `now()` no (PostgreSQL 11+); ampliar `varchar(n)` a un `varchar` más largo o a `text`, o `numeric` a más dígitos con la misma escala, es un cambio de catálogo y filas e índices se quedan; `ADD FOREIGN KEY` sin `NOT VALID` lee toda la tabla hija mientras las escrituras en ambas tablas esperan, así que añádela `NOT VALID` y luego `VALIDATE CONSTRAINT` - que en una clave foránea toma también ROW SHARE sobre la tabla referenciada, así que solo espera el DDL que bloquea esa tabla en exclusiva; eliminar una clave foránea toma también la tabla referenciada bajo ACCESS EXCLUSIVE, y toda lectura de ella espera - igual que eliminar una columna de la que está hecha una clave foránea, eliminar una tabla que referencia a otra (con `CASCADE` también las tablas que la referencian) y cambiar el tipo de una columna de clave en cualquiera de los dos lados, y la confirmación nombra esa otra tabla; `VACUUM FULL`, `CLUSTER` y `REPACK` retienen ACCESS EXCLUSIVE y reescriben la tabla; `REINDEX` sin `CONCURRENTLY` detiene casi toda consulta sobre la tabla. Comparar -> Aplicar lee las sentencias del mismo modo.
- **Un límite de espera de bloqueo por defecto.** El DDL con bloqueo exclusivo
  ejecutado desde el editor recibe un `lock_timeout` de **3 segundos**,
  independientemente de la casilla del tiempo de sentencia. Si no consigue el
  bloqueo en tres segundos, se rinde, y nada ha esperado detrás de él más que
  eso. Es una protección, no el tiempo de sentencia: solo limita la espera
  *por el bloqueo*, no el trabajo posterior.
- **Desactivar la protección.** La comprobación previa tiene una casilla
  *"esperar lo que haga falta"*. Márcala y el DDL espera el bloqueo
  indefinidamente: hazlo solo si puedes detener la carga, porque hasta que el
  DDL entre, el tráfico sobre esas tablas también está detenido.

Si la protección se dispara, el mensaje lo dice - qué límite, y que es la
protección DDL del editor y no un `statement_timeout` o un `lock_timeout` del
lado del servidor - y mantiene en pantalla la última imagen de la cola.

#### Mientras se ejecuta

La línea de estado lleva un reloj, de modo que una sentencia de cuatro minutos
nunca parece una aplicación que ha dejado de responder. En cuanto la sentencia
llega al servidor, la aplicación pregunta también qué está haciendo, desde una
segunda conexión:

- **esperando un bloqueo** - se muestra por delante de todo lo demás. Una
  sentencia que espera no es lenta, está atascada, y cada cosa pide una
  reacción distinta. Para una espera de bloqueo el editor muestra toda la
  cola: cuántas sesiones retienen el bloqueo por delante de ti, cuánto has
  esperado y - la parte que no muestra nada más - cuántas consultas nuevas se
  han acumulado ya detrás de ti. Los botones de ahí abren la vista de bloqueo
  o terminan a los que lo retienen.
- **progreso**, para las operaciones de las que PostgreSQL lo informa - 
  `CREATE INDEX`, `VACUUM`, `ANALYZE`, `CLUSTER`, `COPY` - con la fase en que
  está y una barra.

La barra aparece sólo cuando el servidor informa de un porcentaje real. Uno
inventado sería peor que ninguno.

**Detener** está junto a Ejecutar mientras algo se ejecuta y lo detiene en el
servidor, que es el único sitio desde el que se puede: la petición que lo
lanzó está bloqueada en esa misma conexión, esperando la misma respuesta.
Escape hace lo mismo. La sentencia vuelve como cancelada a petición (57014), y
la sesión sobrevive - incluida una transacción que usted mantenga abierta.

Una cancelación es una petición, no una orden, y un backend metido en una
llamada al sistema no la nota. Si no cuaja en unos segundos, el botón ofrece
**Forzar**, que en su lugar cierra la conexión. Eso siempre funciona y cuesta
la sesión: una transacción abierta se deshace con ella, así que lo pregunta
antes.

La ejecución pertenece a la pestaña desde la que se lanzó. Si sale de esa
pestaña, su × de cerrar se convierte en un anillo que gira - una marca cuando
la sentencia llega, una cruz cuando termina con error. Al volver a la
pestaña, la × regresa, porque para entonces el resultado ya está delante de
usted. Las pestañas en las que no se ejecuta nada propio quedan intactas.

#### El panel de salida

Debajo del resultado, la aplicación imprime lo que realmente ocurrió: lo mismo
que imprime `psql`, y una cosa que en otros sitios es fácil pasar por alto.

Por cada sentencia de la ejecución: la etiqueta del comando (`UPDATE 3`,
`CREATE INDEX`, `SELECT 1`), cuántas filas tocó, cuánto tardó y la sentencia
misma. Un script de cinco sentencias dice así cuál fue la lenta, en lugar de
dar un único total para las cinco.

Debajo, los **mensajes del servidor**: todo lo que `RAISE NOTICE` produjo en
una función o en un bloque `DO`, y el comentario continuo de `VACUUM VERBOSE`.
Si depura PL/pgSQL levantando avisos, aquí es donde aparecen.

La cabecera lleva la hora de inicio y la duración total, y **Copy** deja todo
el panel en el portapapeles, para un ticket o un correo.

Cuando una sentencia falla, este panel es la mitad más útil de la respuesta:
las sentencias que aparecen son las que ya se ejecutaron. En autocommit están
confirmadas y el fallo no las deshace; en modo transacción se revirtió toda la
ejecución y ninguna queda.

#### Guardar el resultado en otro formato

`Exportar CSV` pide al servidor que ejecute la consulta de nuevo y envíe todas
las filas, así que es la opción para una tabla entera. El selector **guardar
como…** que hay al lado escribe el resultado que ya está en pantalla, es decir
el que usted está leyendo:

| Formato | Para qué sirve |
|---|---|
| JSON | quien programa y pega datos en una prueba |
| JSON Lines | un objeto por línea - `jq`, una tubería de logs, una carga masiva |
| `INSERT` de SQL | cargar las filas en otra base de datos; se usa el nombre real de la tabla cuando el resultado viene de una |
| Tabla Markdown | un ticket, un merge request, una página de wiki |
| Texto alineado | un correo o un terminal, legible sin herramienta alguna |

Todos los formatos pasan por el mismo diálogo de archivos que cualquier otra
exportación, así que usted elige dónde va y la aplicación le indica después la
ruta completa.

### 5.3 Esquema

Vista detallada de un objeto. Pestañas:
- Columnas (con valor por defecto, tipo, nulabilidad, comentario)
- Índices
- Restricciones
- Disparadores
- Privilegios (editables)
- Relaciones por clave foránea (entrantes y salientes)
- Estadísticas
- DDL - en una tabla o una tabla particionada es el editor de columnas (abajo);
  en todo lo demás, la sentencia CREATE formateada
- Datos - las filas, editables en el sitio en una tabla con clave
  primaria. Dos modos: **builder** arma la consulta con el límite, las
  columnas, los filtros y ORDER BY, y muestra el SQL que ejecutará; **raw
  SQL** te deja escribir la consulta entera. En modo raw SQL el cuadro SQL
  es editable y el selector de columnas, los filtros y ORDER BY se apartan;
  el cuadro empieza con lo que tenía el builder. La consulta se ejecuta en
  solo lectura y debe ser una sola sentencia SELECT, WITH, TABLE o VALUES;
  Limit sigue acotando las filas obtenidas y Ctrl+Enter la ejecuta. La
  cuadrícula sigue siendo editable solo cuando la consulta devuelve todas
  las columnas de esta tabla con su propio nombre; si no, el resultado es de
  solo lectura. Al volver al builder lo encuentras como lo dejaste.

El explorador de esquema de la izquierda gobierna esta vista. Un clic en un objeto allí lo carga aquí.

#### 5.3.1 Editar columnas {#editing-columns}

Seleccione una tabla y abra la pestaña DDL. En lugar de un CREATE TABLE de
sólo lectura obtiene una fila por columna, cada una editable en el sitio:

| Campo | Qué hace el botón contiguo |
|---|---|
| Nombre | `RENAME COLUMN` |
| Tipo | `SET DATA TYPE`, con una expresión `USING` opcional para conversiones que PostgreSQL no hace por su cuenta |
| NOT NULL | `SET NOT NULL` o `DROP NOT NULL`, según la casilla |
| Valor por defecto | `SET DEFAULT` desde el campo, o `DROP DEFAULT` |
| Acciones | `DROP COLUMN`, con una casilla CASCADE al lado |

La columna Adicional muestra lo que el catálogo sabe y aquí no se edita:
PRIMARY KEY, `IDENTITY (ALWAYS)` o `IDENTITY (BY DEFAULT)`, y
`GENERATED STORED` o `GENERATED VIRTUAL`.

**Añadir columna**, abajo, pide un nombre, un tipo, un NOT NULL opcional y uno
de estos:
- *sin valor por defecto*
- *DEFAULT* - una expresión
- *GENERATED* - una expresión más el tipo de almacenamiento. STORED materializa
  el valor al escribir y requiere PostgreSQL 12 o posterior; VIRTUAL lo calcula
  al leer y requiere 18 o posterior. La aplicación siempre escribe la palabra
  clave, porque PostgreSQL 18 cambió lo que significa un
  `GENERATED ALWAYS AS (...)` a secas - antes era un error, ahora es VIRTUAL -
  y si no, la misma sentencia haría cosas distintas en servidores distintos.

Cada acción muestra primero la sentencia exacta, con el bloqueo que toma y lo
que ese bloqueo impide, y no ejecuta nada hasta que usted confirma.

Cuatro de ellas - `SET DATA TYPE`, `SET NOT NULL` y `ADD COLUMN` con un valor
por defecto o una expresión GENERATED - hacen un trabajo proporcional al tamaño
de la tabla: la primera reescribe cada fila, la segunda lee cada fila para
probar la restricción. Esas se ejecutan como tareas en segundo plano, así que
no se cortan a medias en una tabla grande y continúan si cierra el panel;
sígalas en **Base de datos → Tareas en segundo plano**. El resto son cambios de
catálogo que terminan de inmediato y van como una petición normal.

Las vistas, las vistas materializadas y las tablas foráneas no se editan aquí:
sus columnas provienen de su definición, así que la pestaña muestra sólo la
estructura.

#### 5.3.2 Menús de objeto: qué puede pedirse a cada tipo {#object-menus}

Haga clic derecho sobre un objeto en el explorador de esquemas. Bajo las
entradas de navegación y exportación, cada tipo lleva los comandos que
PostgreSQL realmente tiene para él, no un subconjunto común. Dos reglas
mantienen el menú honesto en todas las versiones que la aplicación admite:

- **No se oculta nada.** Una entrada para la que este servidor es demasiado
  antiguo aparece atenuada, y su tooltip nombra la versión que necesita
  (`requiere PostgreSQL 12 o posterior`). Un menú más corto en un servidor
  antiguo dejaría la duda de si la opción no existe o la aplicación la
  olvidó.
- **El límite no vive sólo en el menú.** Cada regla de versión y de tipo de
  objeto que dibuja el menú se aplica de nuevo en el endpoint que hay detrás,
  de modo que una petición repetida no puede saltársela.

**Índice**

| Entrada | Notas |
|---|---|
| REINDEX | Bloqueo exclusivo, rápido |
| REINDEX CONCURRENTLY | PostgreSQL 12+. Sin bloqueo de escritura; una ejecución interrumpida deja un índice NO VÁLIDO |
| REINDEX con opciones… | CONCURRENTLY, VERBOSE (9.5+) y TABLESPACE (14+), ofrecidos sólo donde existen |
| Renombrar / Mover a tablespace | |
| Parámetros de almacenamiento… | Sólo los que acepta este método de acceso: `fillfactor`, `deduplicate_items` (btree, 13+), `fastupdate` y `gin_pending_list_limit` (gin), `pages_per_range` y `autosummarize` (brin), `buffering` (gist) |
| Fijar estadísticas de una expresión… | PostgreSQL 11+, y sólo para una columna de expresión: una columna normal se rechaza en todas las versiones |
| Eliminar | CONCURRENTLY y CASCADE como casillas, mostrando la sentencia antes de ejecutarla |

Un índice NO VÁLIDO recibe una entrada extra arriba que lo dice, con
reconstrucción en un clic. Un índice que existe sólo para aplicar una
restricción indica cuál, y se niega a eliminarse por su cuenta.

**Tabla, tabla particionada, vista materializada**

Un submenú **Mantenimiento** con ANALYZE, VACUUM, VACUUM ANALYZE, VACUUM
FULL, REINDEX TABLE, REINDEX TABLE CONCURRENTLY (12+) y CLUSTER. Cada uno se
ejecuta como tarea en segundo plano: cierre el panel, vuelva y encuéntrelo en
**Base de datos → Tareas en segundo plano**.

*VACUUM con opciones…* es la lista completa, cada fila atenuada con la
versión que necesita si el servidor es anterior: `DISABLE_PAGE_SKIPPING`
(9.6+), `SKIP_LOCKED`, `INDEX_CLEANUP` y `TRUNCATE` (12+), `PARALLEL` (13+),
`PROCESS_TOAST` (14+), `PROCESS_MAIN`, `SKIP_DATABASE_STATS` y
`BUFFER_USAGE_LIMIT` (16+). `PROCESS_TOAST`, `PROCESS_MAIN` y `TRUNCATE`
están activados en PostgreSQL, por eso aparecen marcados: desmárquelos para
desactivarlos.

*CLUSTER…* lista los índices de la tabla y señala aquel por el que está
agrupada actualmente, porque `CLUSTER` sin `USING` sólo funciona si existe.

**TRUNCATE…** es una entrada propia con ONLY, RESTART IDENTITY y CASCADE, y
muestra la sentencia antes de ejecutarla. También pasa por la vía de tareas:
CASCADE puede alcanzar un árbol de tablas que referencian a esta, y el
bloqueo que necesita puede esperar primero detrás de otras sesiones.

En **Gestionar**: *Parámetros de almacenamiento…* (`fillfactor`,
`parallel_workers` (10+), `toast_tuple_target` (11+), `vacuum_index_cleanup`
y `vacuum_truncate` (12+), el bloque `autovacuum_*` y el par de sólo
inserción desde 13), *Mover a tablespace…* y, para una tabla ordinaria,
*Registrada / no registrada…* (9.5+).

**Vista materializada** tiene además REFRESH, REFRESH CONCURRENTLY y
REFRESH … WITH NO DATA. Esta última vacía la vista y hace fallar toda
consulta sobre ella hasta la próxima actualización completa, así que pregunta
por separado.

**Vista** - *Opciones de la vista…*: `check_option` (9.4+),
`security_barrier` (9.2+) y `security_invoker` (15+). La última decide si una
consulta a través de la vista se comprueba con sus privilegios o con los del
propietario de la vista.

**Secuencia** - *Propiedades de la secuencia…* es todo el bloque de
parámetros de `ALTER SEQUENCE` en un formulario: tipo `AS` (10+), INCREMENT
BY, MINVALUE, MAXVALUE, START WITH, CACHE, CYCLE y OWNED BY. Sólo los campos
que cambie entran en la sentencia, y la vista previa muestra exactamente lo
que se ejecutará. *Reiniciar…* fija el siguiente valor que entregará.

**Disparador** - un submenú **Disparo** con los cuatro estados: ENABLE,
DISABLE, ENABLE REPLICA (se dispara sólo mientras `session_replication_role`
sea `replica`) y ENABLE ALWAYS (se dispara pase lo que pase, de modo que una
aplicación de replicación o una restauración no puede silenciarlo). Además
*Renombrar…*.

**Función / procedimiento** - el bloque Propiedades del editor lleva ahora
también COST, ROWS, PARALLEL (9.6+, atenuado por debajo) y LEAKPROOF, junto a
los controles de renombrar / esquema / propietario / search_path que ya
estaban.

**Esquema** - un submenú **Mantenimiento** que ejecuta ANALYZE, VACUUM,
VACUUM ANALYZE o REINDEX sobre cada tabla ordinaria y vista materializada del
esquema, una tras otra, continuando más allá de una tabla que no pueda
bloquear. El panel completo, con alcance de clúster y todas las opciones,
sigue una entrada más abajo.

#### Una copia de las definiciones antes del cambio {#copy-before-the-change}

Mover un objeto a otro esquema, eliminar objetos, cambiar el propietario o
el tablespace escribe primero las definiciones de los objetos en la carpeta
de datos, como SQL que los devuelve: para una tabla las columnas,
restricciones, índices, triggers, políticas, propietario, privilegios,
comentarios y el tablespace en el que están ella y sus índices; para una
rutina su definición. Trabajos en segundo plano lista la copia ("Before:
..."), el aviso tras el cambio la abre, y desde ahí va al editor SQL o a un
archivo `.sql`. La copia sale de `pg_dump --schema-only` cuando hay un
pg_dump que corresponde al servidor, y del catálogo si no. Cuando no se puede
hacer la copia, el cambio se rechaza con el motivo; puedes seguir sin ella, y
el registro de auditoría deja constancia. Un cambio que falla lo deshace
PostgreSQL; esto es para el cambio que salió bien y no debía.

**Una eliminación y todo lo que se lleva.** El clic derecho **Drop** sobre una
tabla, vista, secuencia, índice, rutina, tipo o esquema hace la copia igual
que la eliminación de varios objetos - antes no hacía ninguna. Con
**CASCADE**, la copia contiene todo lo que la eliminación quita, no solo lo
nombrado. Qué objetos son lo dice el propio PostgreSQL: la eliminación se
ensaya en una transacción que se deshace, y el catálogo dice qué
desapareció. Medido, un `DROP TABLE ... CASCADE` se lleva vistas sobre la
tabla en otros esquemas, una vista sobre una de ellas, vistas materializadas,
una función cuyo argumento es el tipo de fila de la tabla y las claves
foráneas de otras tablas; un `DROP TYPE ... CASCADE` saca las columnas de ese
tipo de otras tablas, con sus valores; una función de trigger eliminada se
lleva sus triggers, una secuencia eliminada los valores por defecto que la
llaman. El diálogo las lista cuando lo pide (**Calcular qué se lleva el
drop**), leídas del grafo de dependencias: para la lista no se ensaya nada
ni se bloquea nada, también para un esquema con cientos de tablas. La
copia las nombra arriba, exactamente como las encontró el ensayo. Cuando el
ensayo no puede ejecutarse - una tabla demasiado ocupada para bloquearla en
tres segundos, o event triggers que se dispararían - también la lista de la
copia se lee del grafo de dependencias y la copia dice que es una
estimación. Lo que la copia no puede escribir aparece bajo *NOT in this
copy*.

**Qué copiar antes.** El diálogo de eliminación, y el que elimina varios
objetos seleccionados, se abren sin preguntar nada al servidor. Bajo la
sentencia y sus opciones (**CASCADE**, **IF EXISTS**, el SQL) preguntan, en
este orden:

1. **¿Guardar una copia antes del drop?** La primera opción es **Sin copia -
   ejecutar solo el DROP**: no se guarda nada ni se calcula nada antes, solo
   se ejecuta la sentencia DROP (para un esquema con **CASCADE**, `DROP
   SCHEMA ... CASCADE` sobre ese esquema y nada más). No se puede deshacer,
   salvo desde una copia de seguridad. **Guardar antes una copia** abre la
   pregunta siguiente.
2. **Qué guardar:** **Copiar solo las definiciones** (las definiciones de
   todo lo que se lleva la eliminación; una tabla vuelve vacía) o **Copiar
   definiciones y filas** (también las filas de las tablas que se van se
   escriben en la carpeta de datos).
3. **Calcular qué se lleva el drop** - solo cuando lo pulsa: los objetos que
   quita **CASCADE** y, para las tablas que se van, cuántas son, unas
   cuántas filas y cuántos datos. Una copia lo necesita antes de habilitar
   **Eliminar**; sin copia también puede pedirlo, pero no hace falta.

Los tamaños son la estimación que lleva el servidor (filas y páginas de sus
estadísticas), leída sin abrir ninguna tabla; una tabla nunca pasada por
vacuum ni analizada aún no tiene tamaño, y el diálogo lo dice. Una copia
ensaya primero la eliminación para saber exactamente qué se va, y eso
mantiene los bloqueos de la eliminación un momento (espera como mucho tres
segundos por ellos). El diálogo recuerda la última elección hasta que se
cierra la aplicación (no se guarda en ningún sitio): la siguiente
eliminación empieza por ella, salvo que las filas por encima de 50 MB pasan
a solo las definiciones en cuanto el análisis lo muestra, y el diálogo dice
por qué. Sin copia, cuando la eliminación puede llevarse datos u otros
objetos (una tabla, un esquema, cualquier cosa con **CASCADE**, salvo que
el análisis haya mostrado que no se va nada), escribe el nombre del objeto
(para varios objetos, cuántos son) antes de que se habilite **Eliminar**.
Un tipo del que no se puede escribir copia (una extensión, un disparador, un
rol ...) lo dice y ofrece solo el análisis.

**Mientras se ejecuta.** La eliminación es un trabajo en segundo plano, y el
diálogo la sigue paso a paso: averiguar exactamente qué quita la
eliminación, copiar las definiciones, copiar las filas (tantas de tantas
tablas, cuánto se ha escrito), eliminar - con el tiempo transcurrido hasta
ahora. Hasta que empieza el propio DROP, **Cancelar** dice **Detener**: se
detiene en el próximo punto seguro, no se elimina nada, y una copia hecha
hasta entonces se borra con sus filas. Una vez en marcha el DROP ya no se
puede detener, y el botón lo dice. Cerrar el diálogo no detiene el trabajo;
sigue en Trabajos en segundo plano. Mover y clonar desde el mismo diálogo
muestran el mismo estado de trabajo.

**Las filas.** Las definiciones no son los datos. **Copiar definiciones y
filas** escribe las filas de cada tabla que
se lleva la eliminación en la carpeta de datos junto a la copia, y los
valores de una columna que un cascade quita de una tabla que se queda
(emparejados por su clave primaria; una tabla sin ella no puede recuperarlos,
y la copia lo dice). Para deshacer una eliminación: abre la copia, ejecútala
en el editor SQL y luego **Devolver las filas** - una transacción, padres
antes que hijos, en tablas que deben existir de nuevo y estar vacías, así
que nada se carga dos veces. Las filas vuelven a donde se tomó la copia y a
ningún otro sitio: la copia guarda el servidor (su dirección, el servicio
libpq o el túnel SSH, y el identificador de sistema del propio servidor) y la
base de datos, **Devolver las filas** usa una pestaña abierta o una conexión
guardada que llegue a ese servidor - nunca la pestaña de delante si está en
otro sitio - y al servidor se le pregunta quién es antes de escribir una sola
fila. **Abrir en el editor SQL** abre la copia solo en una pestaña conectada a
esa base de datos, y lo dice cuando no hay ninguna. Una clave foránea de una tabla que se queda
vuelve como `NOT VALID` cuando esa tabla tiene filas que apuntan a una tabla
que vuelve vacía, y se comprueba cuando vuelven las filas; entonces también
se refrescan las vistas materializadas. Las filas de una tabla foránea viven
en el otro servidor y no se guardan. Con solo las definiciones, la copia
dice arriba que solo contiene las definiciones.

### 5.4 Sesiones

Tres modos (conmutador arriba):

- Lista: tabla ordenable de pg_stat_activity con pid, base, usuario, aplicación, cliente, estado, espera, duraciones y consulta.
- Carga en vivo: progreso y tiempo estimado por consulta activa. Cruza el historial de pg_stat_statements (media / desviación / bloques por llamada) con las vistas pg_stat_progress_* (VACUUM / ANALYZE / CREATE INDEX / CLUSTER / COPY).

#### «idle in transaction» y lo que cuesta

Una sesión en ese estado recibe una de tres lecturas, y todas están medidas, no
supuestas:

| lo que dice | lo que significa |
|---|---|
| *N sesiones esperándola* | alguien está atascado detrás. Es un problema de bloqueo, y esas mismas sesiones aparecen en Blocking y en la comprobación `db-locks` |
| *mantiene N bloqueos, nadie espera* | tomó los bloqueos que necesitaban sus sentencias y los mantendrá hasta el commit, pero de momento nadie los quiere |
| *sin bloqueos que alguien pueda esperar* | una transacción de solo lectura dejada abierta. No bloquea a nadie |

El tercero es el habitual, y antes se leía «holding locks» como todos los
demás. Eso mandaba a la gente a Blocking y a los bloqueos de la salud del
clúster, donde con razón no encontraban nada: la etiqueta describía el estado,
no los bloqueos. El identificador virtual de transacción propio no se cuenta:
lo tiene toda sesión y nadie puede esperarlo.

Una transacción abierta mucho tiempo conviene cerrarla también en el tercer
caso, y la tarjeta lo dice pasados cinco minutos: la transacción abierta más
antigua retiene el horizonte xmin, así que autovacuum no puede limpiar ninguna
versión de fila más reciente que ella, en ninguna parte de la base de datos, no
solo en las tablas que esa transacción tocó.
- Panel: instantánea en una pantalla (versiones, estados de conexión, eventos de espera, acierto de caché, tablas más calientes, estadística de checkpoints, bloqueos, retraso de replicación, transacción abierta más larga, workers de autovacuum).

Clic derecho en una sesión: Cancelar (la consulta en curso) o Matar (el backend).

### 5.4.1 Lo que cuesta al servidor una ventana abierta {#what-an-open-window-costs-the-server}

Una ventana conectada pregunta al servidor una cosa cada cinco segundos: las sesiones activas, los datos detrás de las luces de alarma de la cabecera y si el servidor es primario o standby. Lo que no cambia de un ciclo al siguiente (qué columnas y funciones tiene, dónde está pg_stat_statements y si está cargado, los ajustes de workers y de archivado) se pregunta una vez y se guarda, diez minutos, o un minuto donde una extensión puede cambiarlo; una extensión instalada, actualizada, movida o eliminada desde la aplicación se vuelve a preguntar al momento. La vista Live load usa la misma respuesta de cinco segundos que la cabecera en lugar de preguntar dos veces. Medido en PostgreSQL 19, para una ventana:

| Ventana | Sentencias por minuto | Tiempo de servidor por minuto | Bloques por minuto |
|---|---:|---:|---:|
| En reposo (una pestaña de comprobación delante) | unas 500 | 14 ms | unos 2.000 |
| Live load delante | unas 470 | 12 ms | unos 1.500 |

La mayoría de las sentencias son los pocos ajustes que cada petición hace en su conexión (rol, search path, límites de tiempo), menos de 0,1 ms cada uno. Con la ventana minimizada no se pregunta nada. Una comprobación cuesta lo que cuestan sus propias consultas, una vez, al ejecutarla; el muestreador ASH añade una lectura de pg_stat_activity por muestra (ver *db-ash* arriba).

### 5.4.2 Las alarmas de la cabecera y sus líneas {#header-alarms}

Las lámparas de alarma de la cabecera se evalúan a partir de la misma respuesta cada cinco segundos. Cada línea es un juicio, así que aquí están con su motivo (revisadas el 27.10.2026, después de que la alarma de volcado y la de wraparound dieran ambas falsas alarmas):

- **Conexiones:** aviso al 80% de `max_connections`, crítico al 95%.
- **Bloqueo:** aviso cuando alguien lleva 5 s esperando un bloqueo, crítico a los 60 s - un minuto detrás de un bloqueo no se va a soltar solo; quince segundos, la línea antigua, eran una migración normal.
- **Volcado a disco:** la tasa es una media de 60 s de `pg_stat_database.temp_bytes` y tiene que *mantenerse*: aviso tras tres minutos por encima de 2 MB/s, crítico tras cinco minutos por encima de 20 MB/s. Un informe que volcaba una vez empujaba la media por encima de la línea y la alarma quedaba un minuto; ya no. Mientras se ejecutan `CREATE INDEX`, `REINDEX`, `CLUSTER` o `VACUUM FULL` (PostgreSQL 12+ lo dice), su ordenación en disco es asunto de `maintenance_work_mem` y no se evalúa.
- **Wraparound de IDs de transacción:** `autovacuum_freeze_max_age` (200 millones por defecto) es donde autovacuum empieza por sí mismo los vacuums anti-wraparound, y las tablas ocupadas están cerca en funcionamiento normal - las líneas antiguas, el 80% y el 90% de eso, estaban al 8% del camino hasta la parada real. La alarma se enciende ahora a cinco veces el ajuste, con tope en 1.000 millones, y pasa a crítica a 7,5 veces, con tope en 1.500 millones, donde el Health check dice CRIT; el vacuum failsafe de PostgreSQL empieza a 1.600 millones y la parada está cerca de 2.100 millones. Los topes valen para un DBA que subió el ajuste; los múltiplos permiten que un clúster de pruebas alcance las líneas bajándolo. El texto dice la antigüedad en millones y el camino hasta la parada en porcentaje, y nombra la relación más antigua, su base de datos, cada relación más allá de la línea y el poseedor del snapshot que vacuum no puede superar. Los IDs multixact se evalúan con la misma regla frente a `autovacuum_multixact_freeze_max_age` y encabezan el texto cuando son los más antiguos de los dos.
- **Una transacción abierta:** aviso a los 5 minutos. Crítico a los 30 minutos solo cuando está *idle in transaction* - algo que nada va a terminar; una sentencia activa tan antigua es un informe o un lote y sigue siendo un aviso.
- **Transacciones preparadas:** aviso tras una hora, crítico tras un día.
- **Un slot de replicación muerto:** aviso en cuanto está inactivo, crítico cuando retiene 10 GB de WAL.
- **Archivado de WAL:** aviso cuando el último intento falló; crítico cuando lleva cinco minutos fallando - un intento fallido se reintenta un minuto después. La alarma nombra el segmento que falló y cuándo, el último archivado, el comando o la biblioteca, y cuántos segmentos esperan en `pg_wal`. `archive_mode` activado sin `archive_command` ni `archive_library` es un aviso, crítico cuando esperan 10 GB de WAL; ambos definidos es crítico (desde PostgreSQL 16 el archivador se niega a funcionar).
- **Pools de workers:** aviso al 80% de un pool, crítico al 95%.
- **Retraso de replicación:** aviso a los 60 s, crítico a los 300 s.
- **Cola LISTEN/NOTIFY:** aviso al 10% del máximo de la cola o 512 MB, crítica al 50% o 4 GB, lo que llegue antes. La lámpara aparece donde alguien escucha (o la cola ya guarda algo); lleva a *Tablas de cola y LISTEN/NOTIFY*, que nombra las sesiones que escuchan.

Una alarma crítica abre la ventana que pide confirmación (registrada en el log de auditoría) y puede ponerse en la lista de tareas; un aviso es una lámpara y una notificación. La pregunta antes de una ventana crítica se puede desactivar en Ajustes.

### 5.5 Mantenimiento

Operaciones de mantenimiento y ajuste de autovacuum.

- Elige esquema y tabla y ejecuta VACUUM / VACUUM (FULL) / VACUUM ANALYZE / REINDEX / CLUSTER / ANALYZE / pg_repack. El progreso en vivo viene de pg_stat_progress_*.
- Asesor de autovacuum por tabla: edita las reloptions (`autovacuum_vacuum_scale_factor`, `autovacuum_vacuum_threshold`, …) con un valor recomendado derivado del calor de la tabla (muy caliente / caliente / media / por defecto). En PostgreSQL 13 y posteriores, una tabla solo de inserciones recibe además `autovacuum_vacuum_insert_threshold` y `autovacuum_vacuum_insert_scale_factor` recomendados y aplicados: las filas muertas nunca llegan a su disparador de vacuum, así que el disparador de inserciones es el único vacuum que recibe.

La lista de opciones se construye segun la version del propio servidor.
PROCESS_TOAST llego en 14 y BUFFER_USAGE_LIMIT en 16; enviar una de ellas a un
servidor anterior es un error duro, no un aviso, asi que no se ofrece nada que el
servidor no conozca. Los botones de VACUUM / ANALYZE / REINDEX del table advisor
y del vacuum status advisor usan la misma lista.

Aqui no hay panel de log. Una ejecucion de mantenimiento es un trabajo en segundo
plano como cualquier otro: el panel muestra que algo se esta ejecutando y lleva a
**Background jobs**, donde se lee su salida junto a las demas operaciones largas.

**Una ejecucion que no hizo nada lo dice.** PostgreSQL omite una relacion que
quien la invoca no puede mantener y aun asi informa exito. El trabajo termina
como *skipped* en lugar de *done*, y nombra lo que se omitio.

### 5.6 Servidor

Ajustes del clúster + editor de pg_hba + extensiones + tablespaces + roles.

- Ajustes clave: tarjetas editables para unos 70 parámetros con su razón, regla práctica y distintivos de estado. Los cambios se guardan con ALTER SYSTEM (donde el proveedor lo permite).
- pg_hba.conf: editor de texto que hace copia antes de guardar. Restauración desde cualquier copia. El proveedor debe permitir escritura directa de archivos.

#### Cada ALTER SYSTEM hecho aquí se puede deshacer {#settings-undo}

Cada cambio aplicado desde DBApguin - ajustes clave, los consejos de ajuste,
el editor de ajustes - se registra con el valor que reemplazó y de dónde
venía ese valor. El aviso que confirma el cambio lleva **Deshacer**, y la
vista de ajustes lista los cambios hechos aquí, el más reciente primero,
cada uno con Deshacer hasta que se deshace. Deshacer es exactamente la
sentencia que devuelve el parámetro a como estaba: `ALTER SYSTEM SET` al
valor antiguo cuando ese valor era la propia línea de DBApguin en
`postgresql.auto.conf`, `ALTER SYSTEM RESET` en otro caso, para que vuelva a
regir la fuente anterior (postgresql.conf, el valor por defecto). El
registro es por servidor, en la carpeta de datos, y el registro de auditoría
guarda el valor de antes y de después.

#### pg_hba.conf pertenece a un nodo

`pg_hba.conf` es un archivo en el directorio de datos de un servidor, y la
replicación no lo transporta: el WAL lleva datos, no configuración. Una regla
añadida en el primario no está en el standby, que sigue teniendo lo que
contenía su base backup. El día que importa es un failover: conexiones que ayer
funcionaban las rechaza un nodo que nunca recibió el cambio, y para entonces
nadie está mirando esta pantalla.

Por eso el editor lo dice encima del archivo, y nombra los nodos allí donde el
servidor dice quiénes son:

- **standbys que transmiten desde este servidor**, con su nombre de aplicación
  y su dirección, desde `pg_stat_replication`;
- **ranuras de replicación física**, cuando hay un standby configurado pero
  ausente en este momento: una ranura sin nada conectado sigue siendo un nodo
  que se espera de vuelta;
- **«este nodo es un standby»**, cuando edita la copia de una réplica: el
  primario tiene la suya y un cambio aquí no le llega.

Cuando nada de eso es visible, el aviso es más corto pero permanece. Un rol sin
`pg_monitor` no puede leer `pg_stat_replication`, y un standby desconectado se
ve desde aquí igual que uno que no existe. Sea cual sea la topología, el mismo
cambio hay que hacerlo en cada nodo, incluido un pooler de conexiones delante
de ellos, que tiene reglas propias de otra clase.

**Editar el archivo en una réplica funciona.** Antes fallaba con *cannot set
transaction read-write mode during recovery*: un mensaje sobre modos de
transacción, procedente de una operación que escribía un archivo. Dos causas:
la aplicación pedía una sesión de lectura-escritura, que un servidor en
recuperación rechaza de plano, y el archivo se escribía mediante un objeto
grande, que es una escritura en la base de datos. Ninguna de las dos hace
falta. Ahora la sesión se pide tal como el servidor puede darla, y en un
standby el archivo se escribe con `COPY … TO`, que no toca nada de la base de
datos. La copia previa al guardado, la recarga posterior y el informe de
errores de análisis funcionan igual.
- Extensiones: inventario e instalar / actualizar / eliminar / cambiar de esquema por extensión, con selector de versión.
- Tablespaces: inventario y crear / alterar / eliminar.
- Roles: listado con editor de atributos. Un clic en un rol abre la auditoría completa.

### 5.7 Roles, usuarios y privilegios {#roles-users-and-privileges}

#### El editor de roles

`Resumen de roles y usuarios` lista usuarios con login y roles de grupo en dos
tablas editables. Cada atributo booleano es un desplegable true/false;
`conn_limit` y `valid_until` son campos de texto.

**Primero elige el rol, luego la acción.** Haz clic en la fila para
seleccionarla, o marca su casilla; ctrl-clic y las casillas seleccionan varios.
Una fila de botones está junto a *+ Create role / user* y actúa sobre lo
seleccionado: permanece oculta mientras no haya nada seleccionado. Con varios
roles seleccionados siguen activas las acciones que tienen sentido sobre un
conjunto (**Privileges**, **Lock**, **Drop**, **Save**) y las que editan los
ajustes de un solo rol se atenúan y explican por qué.

**Guardar** envía solo la diferencia, así que un rol que solo miraste no genera
nada. Con varios seleccionados emite una sentencia `ALTER ROLE` por cada rol
que realmente cambiaste, y los enumera antes en la confirmación.

**Edit** abre ese rol en su propia pestaña: atributos, privilegios y
pertenencias en un solo lugar, una pestaña por rol, de modo que dos roles se
pueden trabajar en paralelo. El formulario de privilegios de esa pestaña es el
mismo que el botón **Privileges** abre como diálogo.

**Privileges** con varios roles seleccionados muestra los privilegios actuales
del primero y escribe el mismo conjunto GRANT / REVOKE en cada rol
seleccionado; el aviso de arriba los nombra. No existe el estado actual
compartido de cinco roles - lo compartido es el cambio - así que la base que se
muestra es la de un rol y el diálogo dice cuál.

Los roles de grupo tienen su propia columna **`can_login`**. Un rol de grupo es
`NOLOGIN` por definición, así que la columna empieza siempre en false - está ahí
para que puedas convertirlo más tarde en rol de login sin salir de la aplicación.
Los usuarios con login tienen un botón **Bloquear** para el sentido contrario,
que pone `NOLOGIN` tras confirmar.

#### Límites y tiempos límite de un rol

Abra los atributos de un rol y la sección **Límites y caducidad** contiene,
junto al límite de conexiones y la fecha de caducidad, los cinco tiempos
límite por sesión:

| ajuste | detiene |
|---|---|
| `statement_timeout` | una sola sentencia que dura demasiado |
| `lock_timeout` | la espera de un bloqueo |
| `idle_in_transaction_session_timeout` | una sesión inactiva dentro de una transacción abierta |
| `idle_session_timeout` | una sesión inactiva fuera de una transacción (PostgreSQL 14+) |
| `transaction_timeout` | una transacción entera, por ocupada que esté (PostgreSQL 17+) |

Sólo se ofrecen los que este servidor tiene - los dos últimos llegaron en
versiones posteriores, y ofrecer un campo que el servidor va a rechazar sería
peor que omitirlo.

**Un número sin unidad son milisegundos.** PostgreSQL lee un valor sin unidad
en la unidad propia del ajuste, y para los cinco esa unidad son milisegundos:
`30` significa treinta milisegundos, que no es lo que quiere decir nadie que
escriba 30. Escriba `30s`, `2min` o `500ms` y no queda nada que adivinar. Un
campo vacío borra el ajuste para el alcance en el que esté.

**En caso contrario** es el valor que el rol encuentra cuando este alcance no
fija nada, y nombra la capa de la que viene: el ajuste del propio rol para
todas las bases, el de la base de datos o el del clúster, en el orden en que
PostgreSQL los aplica. En el clúster se indica si viene de un fichero de
configuración o es el valor por defecto compilado, porque significan cosas
distintas al decidir si conviene fijar uno. Se lee sin los ajustes de la base
conectada de por medio: una base con
`ALTER DATABASE … SET idle_in_transaction_session_timeout = '90s'` hacía que
esta columna diera 90s como valor del clúster, cuando el clúster no impone
nada.

**Alcance** elige dónde aterriza un cambio. Si lo deja en *todas las bases de
datos* es `ALTER ROLE … SET`; si elige una base es
`ALTER ROLE … IN DATABASE … SET`, que es el ajuste más estrecho y allí gana. La
lista marca las bases que ya llevan algo para este rol, de modo que veinte
bases con dos anulaciones ya no significan abrir la lista veinte veces.

El resto de atributos de un rol - LOGIN, INHERIT, SUPERUSER, CREATEDB, el
límite de conexiones, la caducidad - **no** son por base de datos y no tienen
selector de alcance. Viven en un catálogo compartido por todo el clúster, así
que `ALTER ROLE` surte el mismo efecto se ejecute desde la base que se
ejecute. Los privilegios sobre tablas, esquemas y funciones son lo contrario:
viven en los catálogos de cada base, y el editor de privilegios los aplica en
la base indicada arriba en la ventana.

Son ajustes `USERSET`: el rol puede subirlos o desactivarlos en su propia
sesión con un simple `SET`. Detienen un accidente, no a alguien deliberado. El
único límite que un rol no puede levantarse a sí mismo es el de conexiones que
está encima - conviene saberlo al elegir entre los dos.

#### Qué concede una pertenencia

Marcar un rol en **Miembro de** emite `GRANT parent TO este_rol`. Junto a cada
uno están las opciones que deciden qué da realmente la pertenencia:

| | |
|---|---|
| **ADMIN** | el miembro puede conceder este rol a otros y revocarlo. Desactivado por defecto; sin ello la pertenencia es un callejón sin salida |
| **INHERIT** | el miembro usa los privilegios del rol automáticamente, sin `SET ROLE`. PostgreSQL 16+ |
| **SET** | el miembro puede hacer `SET ROLE` a él. PostgreSQL 16+, activado por defecto |

Las dos últimas llegaron con PostgreSQL 16, que dividió una pertenencia en tres
respuestas distintas; en servidores anteriores solo se ofrece ADMIN, porque solo
ADMIN existe. Cada casilla muestra lo que la pertenencia lleva ahora mismo,
leído del servidor, de modo que una concedida en otro sitio se ve tal como es y
no como un valor por defecto.

INHERIT desactivado con SET activado es la forma que conviene conocer: el
miembro tiene los privilegios solo mientras se haya convertido en el rol
deliberadamente, que es como se evita que los objetos se creen bajo el
propietario equivocado. INHERIT activado con SET desactivado es lo contrario:
los privilegios valen siempre y convertirse en el rol está prohibido.

Cambiar una opción de una pertenencia que ya existe vuelve a emitir `GRANT`. No
hay `ALTER` para ella, y la vista previa del SQL muestra exactamente eso.

Cada casilla de este diálogo y de Crear rol se explica al pasar por encima.

#### El editor de privilegios

Se abre desde el menú contextual de un rol o desde `Privilegios` en el editor de
roles. Se aplica a un rol y un esquema a la vez; el selector de esquema está
arriba.

Las casillas muestran **lo que el rol ya tiene**, y tienen tres estados en lugar
de dos, porque cada una cubre todos los objetos del esquema a la vez:

| Estado | Significado |
|---|---|
| Marcada | todos los objetos del esquema tienen ese privilegio |
| Un guion | algunos sí y otros no - el recuento aparece al lado, p. ej. `[3/12]` |
| Vacía | ninguno lo tiene |

Un guion no es una decisión. Déjalo y no se emite nada por él. Un clic lo marca:
un `GRANT … ON ALL …` que sube al resto. Otro clic lo vacía: un `REVOKE` que se
lo quita a los pocos que lo tenían.

El panel SQL de abajo muestra exactamente qué ejecutará Aplicar, y dice «sin
cambios» hasta que cambias algo de verdad. Abrir el editor y pulsar Aplicar no
hace nada.

Los privilegios en gris son los que el **usuario conectado** no puede conceder;
el tooltip lo dice.

Un botón `Acciones` junto a Aplicar lleva todo lo del menú contextual del rol - 
auditoría, `search_path`, crear uno similar, copiar privilegios, exportar DDL,
comparar, eliminar - para que no tengas que cerrar el editor. Todo lo que cambia
el rol cierra antes el editor, para que no se aplique sobre una pantalla vieja.

#### Una advertencia sobre qué significa «tiene»

Estas pantallas muestran privilegios **efectivos**: lo que el rol puede hacer de
verdad, incluido lo que hereda por pertenencia a grupos y lo que obtiene por ser
superusuario. Un superusuario aparecerá con todo marcado, exista o no una sola
concesión explícita. Para ver solo las concesiones directas, usa la pantalla de
privilegios por objeto en la vista Esquema.

#### Exportar GRANT

**Exportar GRANT** escribe los privilegios como un script para guardar en git
o ejecutar en otro sitio. Está en tres lugares:

- la ventana de privilegios de un objeto: ese objeto y los privilegios sobre
  sus columnas (también un privilegio de columna en una tabla particionada);
- la ventana de privilegios de un esquema: el esquema y cada tabla, vista,
  secuencia, columna, rutina y tipo que contiene, con los privilegios por
  defecto fijados `IN SCHEMA` para él;
- la auditoría de privilegios de un rol: todo lo que se le concedió al rol,
  en las bases de datos que marque (la base de la conexión está marcada,
  *Todas las bases de datos* lee cada una), y sus privilegios sobre bases de
  datos, tablespaces y parámetros del servidor. Con más de una base de datos
  cada parte empieza con `\connect`, así que ejecute ese script con psql.

El script convierte objetos creados de nuevo por los mismos propietarios en
los que exportó. Por eso los privilegios propios de un propietario no se
escriben, y un privilegio retirado - `EXECUTE` a `PUBLIC`, el `TRUNCATE`
propio de un propietario, un privilegio por defecto quitado con `ALTER
DEFAULT PRIVILEGES ... REVOKE` - sale como `REVOKE`. Lo que un rol alcanza por
ser miembro de otro rol no está en su exportación; el script nombra esos
roles.

### 5.8 Renombrar un esquema {#renaming-a-schema}

Clic derecho en un esquema del explorador → **Renombrar esquema**.

El renombrado en sí es un `ALTER SCHEMA ... RENAME TO ...`, y PostgreSQL se lleva
casi todo con él: tablas, vistas, secuencias, índices, restricciones, claves
foráneas y permisos siguen al esquema sin más trabajo. El nombre se comprueba
antes del intento, así que un nombre ya ocupado se informa como respuesta y no
como fallo.

Lo que **no** sigue automáticamente es cualquier referencia escrita como texto
dentro de una rutina: un cuerpo de función que dice `SELECT * FROM nombre_viejo.t`,
o un `search_path` fijado en una función. Esas se listan tras el renombrado bajo
*«rutinas que siguen referenciando ...»*, con el nombre de la rutina y dónde se
encontró la referencia (cuerpo, `search_path` o ambos). No se reescribe nada por
ti: el cuerpo de una función es código, y editar el código de alguien en silencio
es peor que decirle qué cuatro funciones mirar.

Si el nombre de destino ya existe, renombrar no es lo que quieres - mira abajo.

### 5.9 Fusionar dos esquemas {#merging-two-schemas}

Clic derecho en un esquema del explorador → **Fusionar esquema**.

`ALTER SCHEMA ... RENAME` no puede plegar un esquema dentro de otro, porque el
nombre de destino ya está ocupado. Por eso la fusión es objeto por objeto, y cada
objeto necesita su decisión cuando el mismo nombre existe en ambos lados.
DBApguin lo resuelve antes de que ocurra nada.

#### El plan

Elegir un destino y pulsar **Analizar** lanza una comprobación previa que no
cambia nada y clasifica cada tabla, vista, vista materializada y secuencia del
origen:

| En el destino | Qué significa | Acciones que puedes elegir |
|---|---|---|
| `no está` | solo lo tiene el origen | se mueve - no hay nada que decidir |
| `mismas columnas` | ambos lo tienen, nombres y tipos coinciden | **Fusionar filas**, **Reemplazar**, **Omitir** |
| `las columnas difieren` | ambos lo tienen, las formas no coinciden | **Reemplazar**, **Omitir** |
| `se mueve con su tabla` | una secuencia `serial` o de identidad | ninguna - sigue a su tabla |

Los objetos que solo existen en el *destino* se listan debajo y no se tocan nunca.

La columna de acción es por objeto. **Aplicar a todos** es una comodidad que
aplica una elección donde es legal y deja el resto en paz: no armará *Fusionar
filas* en una tabla cuyas columnas difieren. Una acción que la comprobación
previa no ofreció la rechaza el servidor en vez de rebajarla en silencio, así que
lo que mostró el diálogo es lo que se ejecuta.

#### Qué hace cada acción

- **Mover** - `ALTER ... SET SCHEMA`. El objeto cambia de esquema. Sus permisos,
  comentarios, índices y restricciones van con él, y **desaparece del origen**.
- **Copiar** - el origen conserva todo. La tabla se recrea en el destino con
  `LIKE ... INCLUDING ALL` (valores por defecto, restricciones, índices,
  comentarios, opciones de almacenamiento) y se insertan sus filas; una secuencia
  se recrea con el mismo valor. Solo se ofrece para tablas y secuencias: la
  definición de una vista nombra el esquema del que lee, así que una vista copiada
  seguiría leyendo en silencio los datos del origen, lo que es peor que no ofrecer
  la opción.
- **Reemplazar** - el objeto del destino se elimina con `CASCADE` y el del origen
  ocupa su lugar. Eso destruye los datos del destino *y* sus permisos sobre ese
  objeto. Es la única elección realmente destructiva del diálogo, por eso el botón
  Fusionar se pone rojo y cuenta lo que se va a eliminar.
- **Fusionar filas** - la tabla del destino se queda donde está y las filas del
  origen se insertan en ella. La tabla de origen ni se vacía ni se elimina: sus
  filas siguen ahí después, así que la fusión puede repetirse o comprobarse antes
  de limpiar.

Es decir, el origen sobrevive a **Copiar** y a **Fusionar filas**, y no sobrevive
a **Mover** ni a **Reemplazar**. El plan dice qué hará cada objeto antes de
ejecutarlo.
- **Omitir** - el objeto del destino se queda como está.

#### Cuando las filas chocan

Solo afecta a *Fusionar filas*, y lo decide la clave primaria de la tabla de
destino:

- **Quedarse con la fila del destino** - `ON CONFLICT DO NOTHING`. Ganan las filas
  que ya estaban; las nuevas se añaden.
- **Que gane la fila del origen** - un upsert. Las filas en conflicto se
  sobrescriben con los valores del origen.
- **Detener la fusión** - un `INSERT` simple. El primer conflicto aborta, y como
  toda la fusión es una transacción, no se aplica absolutamente nada.

Si la tabla de destino **no tiene clave primaria** no hay nada con lo que
reconocer un conflicto, y cada fila del origen se añade, duplicados incluidos. El
plan lo dice en la propia fila, antes de ejecutar, en lugar de dejarte encontrar
las filas duplicadas después.

#### Cosas que conviene saber

- **Toda la fusión es una transacción.** Si algo falla, no se aplica nada. Una
  fusión detenida a medias dejaría dos esquemas medio llenos y ningún registro de
  dónde paró, lo que es peor que no haber empezado.
- **Una secuencia `serial` o de identidad se mueve con su tabla**, porque
  PostgreSQL la mueve como parte de `ALTER TABLE ... SET SCHEMA`. Aparece en el
  plan para que veas que se moverá, pero no tiene acción propia. Las secuencias
  independientes son objetos corrientes y sí reciben una elección.
- **Funciones, procedimientos y tipos no se mueven.** Se quedan en el esquema de
  origen. La fusión trata de datos y de los objetos que los contienen; el cuerpo
  de una rutina puede nombrar cualquiera de los dos esquemas, y moverla es una
  decisión que solo puedes tomar tú.
- **Las referencias fijas se marcan antes de ejecutar.** Las vistas, las
  claves foráneas y las restricciones CHECK se reconstruyen desde el catálogo
  en cada lectura, así que siguen gratis a una tabla movida. El cuerpo de una
  función PL/pgSQL o de un trigger es texto plano resuelto por nombre en el
  momento de la llamada: un cuerpo que dice `public.pedidos` sigue apuntando
  al esquema viejo después de que la tabla pase a ser `archivo.pedidos`, y
  falla en la siguiente llamada, no al fusionar. La comprobación previa
  escanea exactamente eso en `pg_proc` y `pg_policies` y lista lo que se
  rompería, para que arregles esos cuerpos primero o decidas que la
  referencia está bien. Las políticas RLS están ligadas al OID como una
  vista y solo se marcan cuando llaman a una función que el escaneo ya
  detectó.
- **Las claves foráneas siguen apuntando a donde apuntaban.** Una tabla movida
  cuya clave foránea apuntaba a otra que se quedó acaba con una clave foránea
  entre esquemas que sigue vigente. Es válido, y suele ser señal de que la tabla
  referenciada también quería moverse.
- **Eliminar el esquema de origen si queda vacío** hace exactamente eso y nada
  más: si se omitió algo, el esquema se queda y el diálogo dice cuántos objetos
  siguen dentro.

### 5.10 Copia de seguridad y restauración {#backup-and-restore}

Clic derecho en una base → **Copia de seguridad** o **Restaurar**, o en un esquema
→ **Exportar esquema (pg_dump)**.

Ambos ejecutan los binarios reales `pg_dump` / `pg_restore`. Su ubicación sale de
Ajustes → Herramientas si la fijaste, y si no del descubrimiento automático; una
diferencia de versión entre la herramienta y el servidor se informa en lugar de
suponerse.

#### El destino se comprueba primero

Un volcado que falla a los veinte minutos porque la carpeta era de solo lectura
son veinte minutos perdidos, así que la ruta se comprueba antes de que `pg_dump`
arranque:

- un directorio donde se esperaba un archivo se rechaza
- una carpeta en la que no puedes escribir se rechaza, nombrándola
- un archivo existente se informa en lugar de sobrescribirse en silencio

Usa **Examinar** para elegir una carpeta en lugar de teclear una ruta. Muestra
solo directorios, marca los que admiten escritura y oculta las carpetas ocultas.
Explora el sistema de archivos **del servidor**, no el de esta máquina: pg_dump
corre en el servidor, y ahí es donde la ruta debe existir. La copia de la base y
la exportación del esquema usan el mismo explorador.

#### Copiar todo el clúster {#backing-up-the-whole-cluster}

Clic derecho en una base → **Copia del clúster (pg_dumpall)…**

`pg_dump` trabaja dentro de una base y estructuralmente no puede alcanzar lo que
vive fuera. **Los roles, los tablespaces y los ajustes por base son de todo el
clúster**, así que una copia hecha solo con `pg_dump` restaura tablas que nadie
puede poseer, algo que la gente descubre durante una restauración y no antes.

| Contenido | Qué obtienes |
|---|---|
| Solo objetos globales *(recomendado)* | roles y tablespaces - el complemento de tus volcados por base |
| Solo roles | únicamente `CREATE ROLE` |
| Solo tablespaces | únicamente `CREATE TABLESPACE` |
| Todo | los objetos globales más cada base del clúster, en un archivo |

*Solo objetos globales* es lo que la mayoría quiere: sigue haciendo volcados por
base en formato custom y toma este junto a ellos. *Todo* produce un único archivo
grande en SQL plano y se restaura de forma selectiva más despacio.

`pg_dumpall` **no tiene opción de formato**: la salida siempre es SQL plano, así
que vuelve con `psql`, no con `pg_restore`. El diálogo lo dice al terminar en
lugar de dejar que lo descubras.

#### Opciones de pg_dump {#pg-dump-options}

Abre **Opciones** en el diálogo de copia. Cada control corresponde a una opción de
`pg_dump`, y la opción está impresa en el control para que puedas contrastar tu
elección con la documentación de PostgreSQL:

- **Formato** - custom (`-Fc`, el que quiere `pg_restore`), SQL plano, directorio o tar
- **Compresión** - 0-9; ver más abajo
- **Sección** - qué tercio del volcado escribir; ver más abajo
- **Solo estos esquemas / Solo tablas / Excluir tablas / Excluir datos de tablas / Excluir esquemas** - nombres separados por comas, ver *Nombres y particiones* más abajo. **Elegir…** junto a *Solo estos esquemas* lista los esquemas de la base para marcarlos
- **Trabajos en paralelo** - solo con formato de directorio. Para cualquier otro formato el campo está desactivado, con la razón al lado, en vez de dejarte rellenarlo y fallar después
- `--no-owner`, `--no-privileges`, `--clean`, `--create`, `--inserts`, `--no-comments`, `--no-tablespaces`
- **Avanzado** - `--load-via-partition-root`, `--no-publications`, `--no-subscriptions`, `--no-security-labels`, `--enable-row-security`, objetos grandes incluidos o fuera, un tiempo de espera de bloqueo, *Solo extensiones* (`--extension`) y *Ejecutar como rol* (`--role`)

`--no-owner`, `--clean` (con `--if-exists`), `--create` y `--no-tablespaces`
solo cambian un archivo **SQL plano**. En un archivo custom, tar o directorio
`pg_dump` los ignora sin decir nada - se eligen al restaurar el archivo -, así
que para esos formatos están desactivados, con la razón bajo las opciones, y el
resumen no los promete.

Qué `pg_dump` se ejecuta depende del servidor (el cliente se ajusta a él), y una
opción que ese cliente no tiene queda desactivada con la versión que necesita -
`--extension` necesita la 14, por ejemplo. La línea bajo *Avanzado* nombra el
cliente.

**Nombres y particiones.** Escribe los nombres como los muestra el catálogo, con
mayúsculas y minúsculas: `Sales.Orders` encuentra la tabla que el árbol muestra
como `Sales.Orders`. Pon entre comillas dobles un nombre que contenga un punto o
una coma (`"order.lines"`). Cada nombre se busca antes de empezar el volcado: uno
que no coincide con nada - una errata, una tabla en otro esquema - lo detiene con
ese nombre y, cuando solo difieren mayúsculas y minúsculas, con el que sí existe.
Antes un nombre así se quedaba fuera en silencio de un volcado que parecía
completo.

Una tabla particionada son sus particiones: el padre solo no tiene filas. Por eso
una tabla en *Solo tablas* viene **con sus particiones y tablas hijas, en todos los
niveles** (la casilla bajo el campo, marcada por defecto), y lo mismo una tabla en
*Excluir tablas* o *Excluir datos de tablas*. Con `pg_dump` 16 o posterior eso es
`--table-and-children`; un cliente anterior recibe cada partición nombrada una a
una. Una vez escrito el volcado, su comprobación confirma que cada tabla pedida,
con sus particiones, está en el archivo. Lo que `pg_dump --table` deja fuera
se dice al empezar el volcado: lo que las tablas elegidas usan sin ser suyo -
la función que llama un trigger, una secuencia independiente de la que tira
un valor por defecto, el tipo de una columna, una tabla a la que apunta una
clave foránea, una extensión -, que la base de datos en la que se restaure ya
tiene que tener, y lo que las usa (una vista, la clave foránea de otra tabla).

Nada se pasa como texto libre. Una opción que DBApguin no conoce no se envía, y
eso es lo que evita que un `--algo` perdido acabe en la línea de órdenes.

**Exportar un esquema usa las mismas opciones.** Clic derecho en un esquema →
*Exportar esquema (pg_dump)* abre el mismo conjunto, sin *Excluir esquemas*, que
de todos modos no cambia nada en un volcado ya limitado a un esquema. Ambos
diálogos se construyen a partir de una definición, así que una opción añadida a
uno aparece en los dos.

Una cosa que la exportación de esquema hace por ti: `pg_dump` compara los
patrones de `--table` con nombres *calificados por esquema*, así que pedir
`orders` en un volcado de un solo esquema no encuentra nada y la herramienta se
detiene con *«no se encontraron tablas coincidentes»*. Los nombres escritos sin
esquema se califican con el esquema exportado, de modo que `orders` significa
`sales.orders`, y el esquema va entre comillas, así que uno con mayúsculas no se
pasa a minúsculas por el camino.

La ruta de destino completa se muestra bajo el campo en ambos diálogos. Un cuadro
de texto muestra el final de una ruta larga y oculta el principio, que es justo la
mitad que nombra la máquina y el directorio.

#### Sección: cuáles son las tres partes {#section-what-the-three-parts-are}

Un volcado se escribe en tres pasadas, y `pg_dump` te deja tomar una sola.
**todas** es el valor por defecto y lo que quieres salvo que tengas una razón
concreta.

| Sección | Qué contiene |
|---|---|
| `pre-data` | `CREATE SCHEMA`, `CREATE TABLE`, `CREATE VIEW`, `CREATE SEQUENCE`, propietarios - todo lo que una tabla necesita para existir, pero nada que frene la carga |
| `data` | las filas en sí, como bloques `COPY`, más las llamadas `setval` que dejan las secuencias en su valor actual |
| `post-data` | claves primarias, restricciones unique y foráneas, índices, disparadores - todo lo que comprueba o indexa las filas |

La división no es arbitraria: restricciones e índices van al final **porque cargar
filas en una tabla que ya los tiene es mucho más lento**, y cada fila se
comprobaría e indexaría de una en una. Restaurar pre-data, luego data y luego
post-data construye los índices una vez, al final, sobre la tabla ya completa.

Tomar una sección sola es para casos especiales:

- `pre-data` - la forma vacía de una base, para comparar estructuras o construir
  un destino que llenarás de otra manera
- `data` - refrescar filas en una estructura que ya existe
- `post-data` - recrear restricciones e índices tras una carga solo de datos

Restaurar un volcado `data` en una base sin tablas falla, igual que `post-data`
sin las tablas a las que engancharse. Las secciones son partes de un volcado, no
tres copias independientes.

#### Compresión: qué cuestan los números {#compression-what-the-numbers-cost}

La compresión se aplica a los formatos custom, directorio y plano. **Tar no se
puede comprimir en absoluto**: el campo se desactiva al elegirlo.

Medido sobre una tabla de 120.000 filas de hashes (una mezcla realista, no un
archivo de texto repetido):

| Nivel | Tamaño | Tiempo |
|---|---|---|
| 0 - ninguna | 13,4 MB | 0,3 s |
| 1 | 7,5 MB | 0,8 s |
| 5 | 7,35 MB | 1,5 s |
| 9 - el menor | 7,35 MB | 1,7 s |
| por defecto | 7,35 MB | 1,6 s |

La forma de esa tabla es justo lo importante: **casi todo el ahorro está entre 0 y
1.** El nivel 1 quita un 44 % en medio segundo; ir de 1 a 9 compra un 2 % más y
tarda el doble. El nivel 9 no es «mucho más pequeño», es «igual de grande, más
lento».

Así que:

- déjalo en **por defecto** si no tienes una razón: ya está cerca del mejor tamaño
- elige **0** cuando el volcado va a un sitio que ya comprime, o a un disco rápido
  donde el tiempo importa más que el espacio
- elige **9** solo cuando el archivo tenga que viajar por algo lento y te dé igual
  cuánto tarde el volcado

Los formatos custom y directorio comprimen por defecto; un volcado en SQL plano va
sin comprimir salvo que lo pidas, y pedirlo significa obtener un archivo gzip en
lugar de SQL legible.

#### Importar un esquema {#importing-one-schema}

Clic derecho en una base → **Importar esquema (pg_restore)…**. Es el diálogo de
restauración con el filtro de esquema delante: nombra el esquema tal como aparece
en el volcado y solo ese se restaura en esta base. Déjalo vacío y se restaura todo
el volcado, que es la restauración normal.

Esto funciona también con el volcado de una base entera: no necesitas un volcado
de un solo esquema para sacar uno.

Una cosa que DBApguin hace por ti aquí: `pg_restore --schema sales` restaura los
objetos *dentro* de `sales`, pero `CREATE SCHEMA sales` no es uno de ellos. Contra
una base que no tiene ya el esquema, cada objeto falla con *«schema sales does not
exist»*, y `pg_restore` aun así termina informando solo «errors ignored on
restore». Por eso, cuando la base no tiene el esquema, primero se restaura desde
el volcado, con su propietario, sus permisos y su comentario tal como los tiene
el volcado. Un esquema que ya existe se deja exactamente como está.

#### Importar con otro nombre de esquema

PostgreSQL no tiene equivalente del `REMAP_SCHEMA` de Oracle. Ni `pg_dump` ni
`pg_restore` pueden renombrar un esquema al entrar: esa opción no existe en
ninguno de los dos.

Lo que sí existe es `ALTER SCHEMA … RENAME`, que renombra como es debido:
restricciones, secuencias, índices y permisos siguen. Por eso **Restaurar como
esquema** en el diálogo de importación restaura con el nombre del volcado y
renombra después. El mismo resultado, por una vía que el servidor sí admite.

Si el nombre que pediste ya está ocupado, el renombrado *no* se fuerza: dos
esquemas no se pliegan renombrando uno sobre otro. La importación lo dice y
apunta a **Fusionar esquema** (5.9), que existe justo para eso.

#### Qué puede dejar roto una restauración

`pg_restore` informa de los problemas y sigue. El caso habitual es una importación
limitada a un esquema cuya clave foránea apunta a una tabla de otro esquema: la
restricción falla, la tabla se restaura **sin ella**, y `pg_restore` termina
diciendo solo *«errors ignored on restore»*. Después nada en la base registra que
esa restricción debería estar.

Tampoco hay una lista de objetos inválidos que consultar. **PostgreSQL no tiene un
estado INVALID como Oracle**: `pg_proc` no tiene columna de validez, y una función
que referencia una tabla ausente se crea sin protestar y falla solo cuando alguien
la llama. Las vistas no pueden invalidarse en absoluto, porque el servidor se
niega a eliminar aquello de lo que depende una vista.

Lo que el catálogo *sí* registra, y lo que la importación comprueba por ti al
terminar:

| Informado | De dónde sale |
|---|---|
| restricciones añadidas como `NOT VALID` | `pg_constraint.convalidated` - las filas existentes nunca se comprobaron |
| índices inválidos | `pg_index.indisvalid` - un `CREATE INDEX CONCURRENTLY` que no terminó |
| disparadores desactivados | `pg_trigger.tgenabled` - quedaron apagados tras una restauración interrumpida con `--disable-triggers` |
| claves foráneas que apuntan fuera del esquema | las que una restauración de un solo esquema no puede llevarse |

Cada hallazgo viene con la sentencia que lo arregla. Cuando no hay nada que
informar, lo dice: el silencio aquí es un resultado, no una respuesta ausente. Las
sentencias exactas que fallaron quedan en el registro del trabajo, accesible desde
**Trabajos en segundo plano**.

#### Una base de datos nueva es el destino por defecto {#restore-into-a-new-database}

El diálogo de restauración se abre en **(una base de datos nueva)**, con el
nombre de aquella desde la que lo abriste y la fecha añadida; la base de
datos se crea vacía y el volcado se restaura en ella, así que nada de lo
existente se sobrescribe - con el nombre que le des y desde cualquier
formato. La base de datos de la que se hizo el volcado puede quedarse donde
está, en el mismo clúster.

Eso vale también para un archivo SQL plano hecho con `--create`, que no
siempre fue así. Un archivo así crea su base de datos y se cambia a ella
(`CREATE DATABASE shop`, `\connect shop`), y psql le obedece a él, no a la
base en la que se inició: ejecutado tal cual no restauraba nada en la base
nueva, y uno hecho con `--create --clean` eliminaba primero la base de la que
venía y la reconstruía desde el archivo - medido: las filas añadidas después
del volcado desaparecieron. El diálogo lee ahora el archivo antes de
restaurar y avisa cuando hace esto; la restauración se ejecuta entonces desde
una copia sin las líneas `DROP DATABASE`, `CREATE DATABASE` y `\connect`, y
las sentencias sobre la base misma (sus ajustes, su comentario y sus
permisos) apuntan a la que elegiste. Tu archivo no se modifica. Un archivo
custom, tar o de directorio nunca tuvo el problema: `pg_restore` no recibe
`--create`, así que escribe en la base a la que se le dirige.

Un archivo de un clúster entero (`pg_dumpall`) contiene varias bases de
datos, roles y tablespaces y no cabe en una sola base; se rechaza con ese
motivo. Restáuralo con psql en un clúster que no tenga ninguna de sus bases,
o haz un volcado de la base que necesitas.

#### Restaurar en una base de datos que existe {#restore-into-an-existing-database}

Aún puedes elegir una base de datos existente de la lista; la confirmación
dice entonces qué contiene - cuántas tablas y vistas, qué tamaño - antes de
escribir nada. Conviene saber qué pasa allí, porque no es un reemplazo:

- **Sin --clean** no se reemplaza nada en la base. Un objeto que existe se
  queda como está - crearlo falla y se informa - y las filas del volcado se
  **añaden** a las tablas que existen. Una tabla con una clave primaria o
  única con la que choquen las filas del volcado no carga ninguna; una tabla
  sin clave las recibe por segunda vez. Medido: una tabla sin clave pasó de
  3 a 5 filas.
- **Con --clean** los objetos del volcado se eliminan primero y se crean de
  nuevo, así que sus filas actuales se sustituyen por las del volcado. Los
  objetos que el volcado no contiene no se tocan.

#### Tablespaces al restaurar {#restore-tablespaces}

Un volcado recuerda el tablespace de cada objeto que no estaba en el
predeterminado de su base y lo vuelve a poner allí. El diálogo lista los
tablespaces que nombra el archivo y cuáles no tiene el servidor. Uno que
falta no se pasa por alto en silencio: `pg_restore` pondría esos objetos en
el tablespace predeterminado con un error por cada uno, y psql se detendría
en el primero. En su lugar la restauración se rechaza con los nombres, y un
clic marca **--no-tablespaces**.

**Tablespace de la nueva base de datos** crea la base nueva en el
tablespace que elijas. Con **--no-tablespaces** también va allí cada objeto
del volcado - cada base en su propio tablespace, use lo que use el origen.
Sin él, los objetos conservan los tablespaces del volcado y solo el resto va
al de la base nueva.

También puede ser **un tablespace nuevo**: dale un nombre y un directorio en el servidor de la base de datos - que ya debe existir, estar vacío y pertenecer al usuario con el que se ejecuta PostgreSQL - y primero se crea el tablespace y luego la base de datos en él. Crear un tablespace requiere un superusuario.

#### Restaurar con otro nombre {#restoring-into-a-different-name}

`pg_restore` no puede renombrar un esquema al entrar: los nombres del volcado son
los que obtienes. DBApguin no finge lo contrario. Funcionan dos caminos:

1. **Restaurar y luego renombrar.** Restaura el volcado tal cual, después clic
   derecho en el esquema → **Renombrar esquema** (5.8). Restricciones, secuencias
   y permisos siguen al renombrado por sí solos.
2. **Restaurar en un esquema intermedio y fusionar.** Cuando el nombre que quieres
   ya está en uso y contiene datos que piensas conservar, restaura en un esquema
   intermedio y usa **Fusionar esquema** (5.9) para plegarlo en el vivo, con una
   decisión por objeto y una elección sobre las filas en conflicto.

El camino 2 es el indicado en una base en producción: la fusión va en una sola
transacción, así que un conflicto inesperado no cambia nada.

#### El formato se lee del archivo, no de su nombre {#the-format-is-read-from-the-file-not-its-name}

Deja **Formato** en *Detectar del contenido del archivo* y DBApguin mira dentro
del archivo en lugar de su extensión:

| Firma | Formato | Se restaura con |
|---|---|---|
| empieza por `PGDMP` | custom | `pg_restore` |
| `ustar` en la posición 257 | tar | `pg_restore` |
| un directorio | directorio | `pg_restore` |
| cualquier otra cosa | SQL plano | `psql` |

Esto importa porque la extensión no es prueba de nada. Un volcado en SQL plano
guardado como `backup.dump` es habitual, y antes acababa en `pg_restore`, que
respondía *«input file appears to be a text format dump. Please use psql.»*:
correcto, y de ninguna ayuda para quien solo quería restaurar su archivo. Ahora
funcionan ambos sentidos: SQL llamado `.dump` y un archivo custom llamado `.sql`.

El desplegable sigue permitiendo nombrar el formato explícitamente cuando quieras
saltarte la detección.

#### Opciones de restauración {#restore-options}

La restauración tiene su propio conjunto: `--clean`, `--if-exists` (solo con
`--clean`, en lo que `pg_restore` insiste), `--no-owner`, `--no-privileges`,
`--single-transaction`, `--exit-on-error`, `--disable-triggers` (en
restauraciones solo de datos), `--data-only`, `--schema-only`, `--no-comments`,
`--no-publications`, `--no-subscriptions`, `--no-security-labels`,
`--enable-row-security`, *Ejecutar como rol*, trabajos en paralelo, y - para
acotar lo que sale del volcado - **Solo estos esquemas**, **Solo tablas**,
**Excluir esquemas** y **Sección**.

`--single-transaction` y los trabajos en paralelo son mutuamente excluyentes;
elegir ambos se rechaza en lugar de pasarse adelante.

**`psql` reproduce un archivo SQL plano exactamente como se escribió**:
propietarios, permisos y todo lo demás vienen con él, y se detiene en el primer
error. `psql` puede ejecutarlo como una sola transacción y como otro rol, y esas
dos cosas se aplican; cualquier otra opción de arriba necesita un volcado custom,
tar o directorio, así que con un archivo SQL plano están desactivadas, y pedir una
se rechaza nombrándola en vez de ignorarla.

Cosas sobre restaurar *parte* de un volcado que merece saber, porque
`pg_restore` no es simétrico con `pg_dump`:

- **Los nombres se comparan exactamente como están en el volcado**, con
  mayúsculas y minúsculas; funcionan `sales.orders` y `orders`, y un nombre entre
  comillas dobles puede contener un punto o una coma. Uno que no coincide con nada
  en el archivo se rechaza antes de que empiece nada.
- **Solo tablas restaura la tabla completa.** `pg_restore --table` por sí solo
  trae la definición de la tabla y sus filas y nada más: sin claves primarias ni
  foráneas, índices, valores por defecto, disparadores, secuencias propias ni
  particiones - una tabla que en el árbol parece correcta y se comporta como si
  nada de eso existiera. Por defecto la restauración toma lo que `pg_dump --table`
  habría volcado: las particiones en todos los niveles, claves, índices, valores
  por defecto, disparadores, políticas, secuencias propias e identity con sus
  valores, comentarios y permisos, y la función que llama un disparador o la
  secuencia de la que toma un valor por defecto cuando esta base no las tiene.
  *solo la tabla y sus filas* sigue ahí, y dice lo que deja fuera.
- **Al restaurar no hay «excluir tablas».** `pg_restore` no tiene
  `--exclude-table`. Exclúyelas al hacer el volcado o nombra las que quieres.
  Pedirlo se rechaza con esa explicación, en vez de con un «unrecognized option»
  de la herramienta.
- **Excluir un esquema lo crea igualmente, vacío.** `--exclude-schema` omite el
  contenido, no el `CREATE SCHEMA`. Elimina después el esquema vacío si no lo
  quieres.

#### Antes y después de una restauración {#before-and-after-a-restore}

Al elegir el archivo, el diálogo lee lo que la restauración pedirá al destino y
dice lo que falta allí:

- **roles** a los que el volcado hace propietarios de objetos o concede permisos;
  con `--no-owner` / `--no-privileges` dejan de importar (no en un archivo SQL
  plano, que los lleva igualmente);
- **extensiones** que el volcado crea y este servidor no puede instalar;
- **esquemas** donde viven objetos del volcado pero que este no crea - un volcado
  hecho con *Solo tablas* no tiene `CREATE SCHEMA` -, con la oferta de crearlos
  antes, vacíos;
- una **codificación** distinta (a UTF8 no se pierde nada; a cualquier otra, un
  carácter sin código detiene la carga de su tabla) o un **locale** distinto (el
  texto se ordena e indexa en el orden de esta base).

Con `--exit-on-error` o `--single-transaction`, y siempre con un archivo SQL
plano, un rol, extensión o esquema que falte rechaza la restauración antes de
empezar: se pararía en el primero, a medias.

Cuando la restauración termina, el destino se compara con el propio volcado: las
filas de cada tabla (contadas en los datos del volcado, no estimadas), el valor de
cada secuencia, cada propietario y cada permiso que el volcado habría puesto. Lo
que difiere se lista. El código de salida de `pg_restore` solo dice si registró
errores; esto dice si la base contiene lo que contenía el volcado.

#### Clonar un esquema sin sus restricciones {#cloning-a-schema-without-its-constraints}

**Restricciones → omitir restricciones e índices**, en Copia de seguridad y en
Exportar esquema, vuelca las tablas y las filas y ahí se detiene: sin claves
primarias, sin claves únicas ni foráneas, sin índices.

Es una sola ejecución de `pg_dump` - `--section=pre-data --section=data`, que la
herramienta acepta juntas. pre-data lleva las definiciones de las tablas, data
lleva las filas, y todo lo que esto deja fuera vive en post-data. Verificado
contra un servidor real: el archivo resultante no tiene ni un `ADD CONSTRAINT`
ni un `CREATE INDEX`.

Dos razones para quererlo. Una carga masiva va más rápida sin las comprobaciones
ni el mantenimiento de índices, y una herramienta de replicación que hace su
sincronización inicial contra el destino no suele arrancar mientras allí se
impongan claves foráneas. Ambas acaban igual: el gestor de restricciones escribe
el script que las devuelve, y lo ejecutas cuando la carga o la sincronización
termina.

La casilla tiene prioridad sobre el selector **Section** en vez de combinarse
con él - dos controles para la misma opción producirían un volcado que ninguno
de los dos describe.

#### Tiempos límite en un volcado o una restauración larga {#timeouts-on-a-long-dump-or-restore}

Un volcado o una restauración que muere a medias - media hora dentro, sin que el
archivo tenga nada malo - suele ser el servidor cortándolo, no la herramienta
fallando.

Cuatro ajustes pueden hacerlo:

- `statement_timeout` - una sentencia duró demasiado.
- `lock_timeout` - la espera por un bloqueo duró demasiado.
- `idle_in_transaction_session_timeout` - una transacción quedó demasiado tiempo
  inactiva.
- `transaction_timeout` (PostgreSQL 17 y posteriores) - una transacción lleva
  demasiado tiempo abierta, esté inactiva o no.

`pg_dump`, `pg_restore` y un volcado cargado con `psql` quitan los tres primeros
por su cuenta: en un servidor real su registro muestra `SET statement_timeout =
0`, `SET lock_timeout = 0` e `SET idle_in_transaction_session_timeout = 0` antes
de empezar el trabajo. No quitan `transaction_timeout` - y ese es el que corta
el trabajo largo, porque `pg_dump` mantiene una única transacción REPEATABLE
READ para todo el volcado y `pg_restore --single-transaction` mantiene una para
toda la carga. Un servidor con `transaction_timeout = 30min` detiene ambos a los
treinta minutos por bien que vayan.

Un archivo `.sql` sencillo que no escribió `pg_dump` es el otro hueco: no lleva
cabecera `SET`, así que los cuatro se le aplican.

**Tiempos límite del servidor -> quitarlos para esta ejecución**, en Copia de
seguridad, Exportar esquema y Restaurar, quita los que tenga el destino. Va por
`PGOPTIONS`, que libpq aplica al conectar y que se impone sobre un `ALTER
DATABASE ... SET` o un `ALTER ROLE ... SET`. `transaction_timeout` solo se envía
a PostgreSQL 17 y posteriores: un parámetro desconocido en `PGOPTIONS` no se
ignora, la conexión se rechaza con `FATAL: unrecognized configuration
parameter`, y eso convertiría "tu restauración puede quedar cortada" en "tu
restauración no puede empezar".

La línea bajo la casilla informa de lo que el destino tiene configurado
realmente. Al desmarcarla, los ajustes del servidor se dejan como están.

Si una ejecución ya murió así, el mensaje nombra el ajuste: `canceling statement
due to statement timeout`, `terminating connection due to idle-in-transaction
timeout` o `canceling statement due to transaction timeout`.

La línea bajo la casilla informa no solo del valor sino de **dónde se fijó** - 
`statement_timeout = 30min (fijado en database sales)`, o `(fijado en role
app_user)`. Un valor sin origen nombrado viene de la configuración del propio
servidor: `postgresql.conf`, o el grupo de parámetros en Aurora y RDS. Esa es la
diferencia entre saber que algo corta la ejecución y saber qué cambiar.

**En Aurora y RDS en concreto.** `transaction_timeout` no existe antes de
PostgreSQL 17, así que en Aurora 15 o 16 una ejecución que muere en un número
redondo de minutos es una de estas:

- un `statement_timeout` en el grupo de parámetros, la base o el rol - la línea
  bajo la casilla dice en cuál;
- **RDS Proxy**, si la conexión pasa por uno. Su `IdleClientTimeout` es por
  defecto 1800 segundos, exactamente treinta minutos, y cuenta como inactivo a
  un cliente que está esperando una sentencia larga;
- una **NAT gateway o un cortafuegos** por medio. Una sola sentencia larga - la
  construcción del índice al final de una restauración, un COPY de una tabla
  enorme - no envía nada en ninguna dirección mientras corre, así que el
  dispositivo descarta el flujo y la sentencia termina contra un socket que ya
  no sostiene nadie. Las NAT gateway de AWS descartan un flujo inactivo a los
  350 segundos. Justo por eso la aplicación activa keepalives TCP en la conexión
  siempre que el host no es un socket Unix, lo que mantiene vivo el flujo sin
  tráfico propio.

Nada en la aplicación limita cuánto puede durar un volcado o una restauración.
Ambos corren como procesos separados con su estado en disco, así que sobreviven
a cerrar o reiniciar la aplicación, y ni la petición HTTP ni la ventana los
esperan.

#### Seguir un volcado o una restauración mientras se ejecuta

`pg_dump` y `pg_restore` van contando lo que hacen, y el diálogo lo muestra
según llega - desde la primera línea hasta la última, no solo la última. Si
sube para leer algo, deja de seguir el final; desmarque *follow* para que se
quede quieto. **Copy** deja todo el registro en el portapapeles.

El trabajo corre separado: cerrar el diálogo no lo detiene, y el registro sigue
disponible después en *Background jobs*.

#### «errors ignored on restore» no es una restauración fallida

`pg_restore` termina con estado de error en cuanto ha ignorado **algún** error,
e imprime solo un número. El más frecuente con diferencia es un `CREATE` sobre
un objeto que el destino ya tiene: restaurar un esquema en una base que ya lo
tiene falla en `CREATE SCHEMA` y restaura todo lo que hay debajo sin problema.

Por eso el diálogo enumera los errores en sí y dice cuáles de ellos son un
objeto que ya estaba. Cuando lo son todos, lo dice claramente: no se omitió
nada por su causa. Un error de verdad sigue siendo un error de verdad.

#### Un volcado de un PostgreSQL más nuevo

Un volcado `.sql` sencillo empieza con una serie corta de sentencias `SET`, y
un `pg_dump` más nuevo escribe ajustes que un servidor más antiguo nunca ha
oído: `transaction_timeout` llegó con PostgreSQL 17. Reproducido en 16, esa
línea detiene la restauración en el acto y no se restaura nada.

La aplicación lee primero la cabecera y se niega antes de empezar, nombrando el
ajuste, de dónde viene el archivo y qué es este servidor. Los volcados viajan
hacia adelante, no hacia atrás: vuelva a hacer el volcado con un `pg_dump` que
corresponda al destino, o restaure en un servidor al menos tan nuevo como aquel
del que salió.

#### Qué cliente se ejecuta

Un `pg_dump`, `pg_restore` o `psql` de una versión mayor distinta a la del
servidor no es intercambiable, y el desajuste duele en direcciones opuestas:

- `pg_restore` y `psql` **hablan con** el servidor. Abren la sesión con un
  bloque de sentencias `SET` que decide su propia versión, así que un cliente
  más nuevo que el servidor le envía ajustes que nunca ha oído -
  `pg_restore` 17 envía `transaction_timeout`, que no existe por debajo de 17,
  y entonces toda restauración informa de un error ignorado y termina con 1 en
  una carga que funcionó.
- `pg_dump` y `pg_dumpall` **leen** el servidor. Consultan catálogos que solo
  conocen en su propia versión, así que uno más antiguo que el servidor no
  arranca en absoluto: *"aborting because of server version mismatch"*.

Por eso la aplicación pregunta al servidor su versión y elige una copia que le
convenga, de los directorios versionados que las distribuciones mantienen uno
al lado del otro (`/usr/lib/postgresql/*/bin`, `/usr/pgsql-*/bin`, Homebrew,
los instaladores de EDB), además de la copia incluida y de lo que haya en
`$PATH`. Para una restauración hay una segunda condición - un cliente más
antiguo que el `pg_dump` que escribió el archivo no puede leerlo - y no se
resuelve con una tabla: se pregunta al candidato con `pg_restore -l`, que lee
el índice y no restaura nada, y si se niega se prueba la siguiente versión
mayor.

Releer un archivo no tiene servidor alguno dentro, solo el fichero, así que la
verificación de la copia toma la versión más nueva instalada. Eso es tanto una
corrección como un refinamiento: un archivo perfectamente bueno leído por un
`pg_restore` más antiguo que el `pg_dump` que lo escribió responde
*"unsupported version in file header"*, que se parece exactamente a una
corrupción.

**Settings -> Tools** muestra la ruta a la que resuelve cada herramienta y
enumera las demás versiones mayores instaladas al lado, porque la ruta que
aparece ahí es el recurso de reserva y no la respuesta completa. Una ruta que
pongas tú es absoluta y desactiva la elección por completo, que es la salida
cuando un servidor necesita un cliente que la aplicación no habría elegido.

#### Elegir archivos

En todos los sitios donde la aplicación lee o escribe un archivo - una
exportación, un volcado, el archivo a restaurar, importar conexiones, un
certificado - se abre el diálogo de archivos de su propio escritorio, con sus
lugares, sus carpetas recientes y su aviso de sobrescritura. En un sistema sin
ese diálogo, la aplicación recurre a su propio explorador de carpetas y no se
pierde nada.

El diálogo se abre donde están los archivos. `pg_dump` se ejecuta en la máquina
donde corre la aplicación y escribe donde esa máquina puede escribir, así que
el selector recorre ese sistema de archivos.

Guardar siempre pregunta por un nombre de archivo. El único sitio que en su
lugar pregunta por una carpeta es el destino de una copia, porque una carpeta
es lo que `pg_dump` escribe.

### 5.10.5 Tablas particionadas {#partitioned-tables}

Las tablas padre particionadas aparecen bajo la categoría **partitioned tables** del árbol de esquemas. Sus particiones hoja son tablas normales y de otro modo llenarían la categoría **tables** - cien entradas `events_2026_NN` sin nada que las una -, así que el árbol las oculta por defecto. Una línea `tables (12) · show 8 leaf partitions` las trae de vuelta por esquema (se recuerda la elección), y cada una mostrada lleva una marca `part of <padre>`.

**Crear una**: el diálogo *+ Table* tiene una sección **Partitioning** - elige RANGE, LIST o HASH y una clave (una columna, una lista `(col1, col2)`, o una expresión como `(date_trunc('month', at AT TIME ZONE 'UTC'))`). El padre se crea vacío; le añades particiones después. Todo `PRIMARY KEY` o `UNIQUE` en una tabla particionada debe contener cada columna de la clave de partición - PostgreSQL lo exige y el error es claro.

**Gestionarlas**: clic derecho en una tabla particionada y **Partitions…**. El diálogo abre con la estrategia y la clave y lista cada partición: sus límites, filas estimadas, tamaño, tablespace propio si lo tiene, y una etiqueta `detach pending` si se interrumpió un detach concurrente (ver abajo). La partición default está marcada y ordenada al final; una partición que a su vez está particionada se abre desde la misma fila.

- **Create partition** - una partición nueva y vacía adjuntada en una sentencia. Rellena los límites que pide la estrategia.
- **Attach existing table** - trae una tabla que ya cargaste. La nota de la sentencia dice si PostgreSQL hará un **escaneo completo del hijo** (muestra el número de filas y el tamaño) para validar el límite, o si puede ser solo metadatos. Es solo metadatos cuando el hijo ya tiene una restricción `CHECK` que demuestra que cada fila encaja; si no, añade un `CHECK` coincidente, ejecuta `ALTER TABLE ... VALIDATE CONSTRAINT` (eso toma solo `SHARE UPDATE EXCLUSIVE`, no el bloqueo completo), luego adjunta.
- **Detach** - la tabla deja de ser partición y conserva sus filas. **DETACH CONCURRENTLY** está marcado por defecto en PostgreSQL 14+: no toma un bloqueo `ACCESS EXCLUSIVE` en el padre, así que las consultas sobre las demás particiones siguen corriendo. Se ejecuta fuera de una transacción y no se puede revertir, y si se interrumpe la partición queda **detach pending** - la fila muestra entonces **Finalize detach**, que lo completa. PostgreSQL rechaza `DETACH CONCURRENTLY` mientras la tabla tenga una partición DEFAULT, y la casilla se deshabilita con ese motivo; en PostgreSQL 13 y anteriores se deshabilita porque la función no existe, y un detach simple mantiene el bloqueo exclusivo durante su duración (normalmente breve).
- **Drop** - elimina la partición y cada fila en ella.

Nada se ejecuta al hacer clic. El diálogo muestra primero la sentencia, con una línea que dice exactamente qué bloquea, durante cuánto y qué pasa si se interrumpe, y la ejecuta solo al pulsar *Run it*.

**Show DDL** para un padre particionado emite `CREATE TABLE ... PARTITION BY ...`; para una partición hoja emite `CREATE TABLE ... PARTITION OF <padre> FOR VALUES ...` sin lista de columnas y solo con las restricciones e índices definidos en esa partición.

**Convertir entre tabla normal y particionada**: clic derecho en una tabla y elija **Convert to partitioned (recipe)…**, o clic derecho en una tabla particionada y elija **Flatten to a plain table (recipe)…**. Ninguna de las dos opciones ejecuta nada. La conversión es una reconstrucción (una tabla nueva, una copia de cada fila y después un intercambio de nombres), así que el diálogo muestra un script SQL para copiar, leer y ejecutar usted mismo. El script es una sola transacción. Las escrituras esperan desde su primera sentencia hasta el COMMIT, y antes del COMMIT el script compara el número de filas, así que si algo falla, la tabla queda como estaba. Traspasa:

- columnas y valores por defecto, columnas generadas y restricciones CHECK y NOT NULL;
- contadores de identity y serial, así que el siguiente id sigue donde lo dejó la tabla antigua;
- claves e índices con sus nombres originales, y triggers;
- propietario, privilegios, comentarios y políticas de row-level security;
- claves foráneas en ambos sentidos, vistas que leen la tabla y publicaciones.

En una conversión hay dos TODO que completar:

- la clave de partición, que toda PRIMARY KEY y toda restricción UNIQUE debe incluir;
- las particiones. El script ya trae una partición DEFAULT, así que ninguna fila se rechaza durante la copia.

Lo que no puede traspasarse aparece arriba bajo *Read first*. Por ejemplo: checks NO INHERIT en una tabla particionada, restricciones EXCLUDE antes de PostgreSQL 17, vistas materializadas y funciones que usan el tipo de fila de la tabla. La tabla antigua queda como `<nombre>_old` hasta que la elimine.

### 5.10.6 Clonar un esquema

Clic derecho sobre un esquema, luego **Transfer → Clone schema…**. Copia todos
los objetos a un esquema nuevo de la misma base de datos: tablas, vistas,
secuencias, funciones, índices y restricciones, y opcionalmente las filas.

Las claves ajenas se reconstruyen apuntando a la copia y no de vuelta al
original: un clon cuyas claves siguen referenciando el origen no es una copia,
es una segunda puerta a los datos del primer esquema.

Con *copy the rows too* el orden importa y se resuelve por usted: los objetos
se crean sin claves ajenas, entran las filas y después se añaden las claves.
Copiar filas en tablas que ya imponen sus referencias falla en aquella que
resulte ir primero. Las secuencias se llevan a donde habían llegado las
originales, para que el primer insert en la copia no choque con una fila que
vino con los datos. Una columna `IDENTITY` se lleva también sus ajustes de
secuencia - `START WITH`, `INCREMENT BY`, `MINVALUE`, `MAXVALUE`, `CACHE`,
`CYCLE` - así que la copia no vuelve en silencio a los valores por defecto.

Las filas se toman **exactamente como son**, tengan lo que tengan. Todo lo que
juzga o reescribe una fila mientras se escribe espera a que las filas estén
dentro:

- una restricción que el origen tiene `NOT VALID` (un `CHECK`, una clave
  foránea, un `CHECK` de un dominio, el `NOT NULL ... NOT VALID` de
  PostgreSQL 18) se añade `NOT VALID` después de las filas, así que las filas
  que el origen nunca comprobó tampoco se comprueban en la copia, y las nuevas
  sí;
- los triggers y las reglas se crean después de las filas, así que ninguno
  rechaza, reescribe o añade filas a lo copiado;
- la seguridad a nivel de fila y sus políticas se activan después de las filas;
- un índice que un `CREATE INDEX CONCURRENTLY` fallido dejó inválido no se
  copia (en el origen no impone nada y, construido de nuevo sobre las mismas
  filas, vuelve a fallar) - igual que `pg_dump` lo omite.

La seguridad a nivel de fila no rechaza una lectura, la acorta. Si sus
políticas se aplican al rol con el que está conectado (no es el propietario de
la tabla o la tabla tiene `FORCE ROW LEVEL SECURITY`, y no es superusuario ni un
rol con `BYPASSRLS`), o no puede leer alguna de sus columnas, la clonación
nombra la tabla y el motivo **antes** de ejecutar nada y no copia nada, en vez
de copiar solo las filas que usted ve.

**Tabla por tabla** muestra, para cada tabla, las filas copiadas, lo que se
añadió `NOT VALID` como en el origen, lo que se creó después de las filas y lo
que no se copió. Si una fila no se puede copiar, el mensaje nombra la tabla, la
columna, la restricción, la fila que falla y las filas del origen por su clave
primaria. **Clonar tabla**, **Copiar tabla** y la opción *copiar* de
**Fusionar esquema** toman las filas de la misma manera.

**Show the script** imprime todo antes de que se ejecute nada. El clon en sí se
ejecuta como una única transacción: llega entero o no llega. Se ejecuta como
un trabajo del servidor, así que un clon de un esquema grande con sus filas -
minutos de trabajo - no se pierde si cierras el diálogo: **Background jobs**
lo muestra mientras corre y conserva el resultado completo para reabrirlo
después.

Al terminar, el diálogo enumera las comprobaciones contra las que se verificó
la copia, en vez de un simple «Hecho»: el esquema nuevo se lee de vuelta
antes de que la transacción haga commit, y cada tabla, secuencia, restricción
y recuento de filas se compara con el original.

Clonar sobre un esquema que ya existe se rechaza. Eso es una fusión, y para eso
está [Fusionar dos esquemas](#59-merging-two-schemas).

### 5.11 Cambiar el propietario de todo en un esquema {#changing-the-owner-of-everything-in-a-schema}

Clic derecho en un esquema → **Cambiar el propietario de todos los objetos…**

Un esquema restaurado bajo el rol equivocado, o alguien que se va, deja cada
objeto del esquema en manos de quien no debería tenerlo. Hacerlo objeto por
objeto está bien con tres objetos y es inviable con trescientos.

Elige el nuevo propietario. El diálogo lista cada objeto del esquema que tiene
propietario propio, agrupado como en el árbol del esquema: tablas, tablas
particionadas con sus particiones anidadas debajo en cualquier nivel, vistas,
vistas materializadas, tablas foráneas, secuencias, funciones, procedimientos,
agregados, tipos, dominios, tipos de rango, operadores, clases y familias de
operadores, intercalaciones, conversiones, configuraciones y diccionarios de
búsqueda de texto, objetos de estadísticas, y el propio esquema, marcado.
Debajo de la lista están las sentencias `ALTER ... OWNER TO` exactas de lo
marcado; nada se ejecuta hasta que pulsas el botón. Los objetos que ya
pertenecen al nuevo propietario se muestran y no se tocan.

Las particiones aparecen una a una porque PostgreSQL da a cada partición un
propietario propio: cambiar el propietario de una tabla particionada no cambia
sus particiones. Marcar o desmarcar una tabla particionada hace lo mismo con
todo lo que tiene debajo.

Los servidores foráneos y las publicaciones que usan las tablas del esquema
pertenecen a toda la base de datos, no al esquema. Aparecen al final, sin
marcar; márcalos si también deben cambiar.

Algunos objetos no se cambian por sí solos, y el diálogo dice cuáles:

- una secuencia que pertenece a una columna (serial o identity) se mueve con su
  tabla, y PostgreSQL se niega a cambiarla por separado. Cuando su tabla ya
  tiene el nuevo propietario y la secuencia no, nada la movería: una secuencia
  serial aparece entonces con las sentencias que la desvinculan, la entregan y
  la vinculan de nuevo a su columna; una secuencia identity no se puede
  desvincular, así que se muestra encima de la lista con las dos sentencias que
  la mueven a través de su tabla y lo que cuestan;
- los objetos que pertenecen a una extensión conservan el propietario de la
  extensión;
- los índices siempre tienen el propietario de su tabla.

La aplicación va en **una sola transacción**. Un esquema reasignado a medias es
más difícil de entender que uno que nadie tocó, y desde fuera no hay forma de
saber dónde se detuvo. El propio esquema se cambia primero, para que el nuevo
propietario pueda crear en él cuando recibe sus objetos. El resultado muestra
las sentencias que devuelven a cada objeto su propietario anterior, por si hay
que deshacer el cambio.

Después, el propietario anterior conserva en este esquema solo lo que también
dejaría el propio REASSIGN OWNED de PostgreSQL: los privilegios por defecto que
ese rol definió para el esquema.

**Un objeto.** Clic derecho en un objeto → **Cambiar propietario…** (para una
base de datos, en el menú de la base de datos). El diálogo muestra el
propietario actual y la sentencia exacta antes de ejecutar nada. Para una tabla
particionada dice también cuántas particiones tiene, contando todos los
niveles, y de quién son, y ofrece **Cambiar también el propietario de sus N
particiones**. La casilla está desactivada por defecto, como en PostgreSQL: la
tabla cambia de propietario y cada partición conserva el suyo. Si se deja así,
el resultado dice cuántas particiones conservaron su propietario.

### 5.12 Comparar esquemas entre servidores {#comparing-schemas-across-servers}

Clic derecho en un esquema → **Comparar con otro esquema**. El diálogo de
comparación se abre con ese esquema elegido en el lado A; elige uno también en
el lado B y compara los dos esquemas. Si dejas ambos en *toda la base de datos*,
compara las bases.

El lado B tiene su propio selector de **servidor** con tus conexiones guardadas,
así que la comparación puede ir entre dos clústeres - desarrollo contra producción
es el motivo habitual. Elegir un servidor recarga su lista de esquemas, porque los
esquemas de un clúster no dicen nada sobre los de otro.

Comparar `sales` de un clúster con `sales` de otro es algo perfectamente normal,
de modo que la protección de «mismo esquema» solo se aplica cuando ambos lados
están en la misma conexión.

El resultado es la diferencia a nivel de objeto más un **script de
sincronización** que dejaría B igual que A. Nada se ejecuta solo: ver
[Ejecutarlo en B](#running-it-on-b).

#### A y B lado a lado {#a-and-b-side-by-side}

Cada fila que difiere - un objeto de un esquema, el propietario y los permisos
de un esquema, una extensión, una propiedad, un rol, una conversión de tipo
(cast), las particiones de una tabla particionada - se abre en su sitio al
hacer clic en ella (o al pulsar su flecha: Tab llega a ella, Intro o Espacio la
abre). Dos paneles muestran la definición de A a la izquierda y la de B a la
derecha, con números de línea:

- una línea que solo tiene A está en rojo, una que solo tiene B en verde;
- una línea que tienen ambos pero distinta queda frente a su pareja, con las
  palabras que difieren marcadas;
- un tramo largo sin cambios se pliega en *N líneas sin cambios*; haz clic para
  ver esas líneas;
- **Copiar A** y **Copiar B** copian el texto de ese lado;
- bajo los dos paneles, las sentencias que el script de sincronización escribe
  para ese objeto - las mismas que escribe **Generar script de sincronización**,
  con las opciones actuales.

En una ventana estrecha los paneles se apilan, A encima de B. Una definición de
miles de líneas se abre al momento: solo se dibujan los cambios y tres líneas a
cada lado hasta que abras un pliegue.

**Comparar bases de datos** - menú Base de datos, o clic derecho en la base en la
barra lateral → Gestionar → Comparar con otra base de datos - hace lo mismo con
bases enteras y acepta una lista de esquemas para acotar la comparación. Eso
importa en una base grande: el recorrido de objetos por esquema es la mitad
cara, y comparar un esquema de cuarenta no debería costar lo mismo que comparar
los cuarenta.

La comparación de bases cubre también lo que no pertenece a ningún esquema:
publicaciones, suscripciones, servidores externos, disparadores de eventos y las
conversiones de tipo que alguien creó con CREATE CAST (las incorporadas y las
que trae una extensión quedan fuera). Una conversión que difiere tiene su
casilla como un esquema o un rol, y el script de sincronización la borra y la
crea de nuevo: una conversión no guarda datos y no se puede modificar.

Las listas de bases de datos y de esquemas de ambos lados se pueden ordenar en su desplegable: A-Z, Z-A o primero los más grandes. El tamaño de las bases de datos es exacto; el de los esquemas es una estimación de las estadísticas del planificador (el último VACUUM o ANALYZE), que no cuesta nada ni espera ningún bloqueo. La entrada de la base de datos completa queda arriba en cualquier orden.

#### Ejecutarlo en B {#running-it-on-b}

Cada fila para la que el script escribe sentencias tiene una casilla, marcada;
una fila para la que no escribe nada (un número de filas, una nota) tiene una
que no se puede marcar. En la comparación de bases,
la casilla de un esquema marca sus objetos. Bajo el resultado, tres acciones
trabajan con las sentencias de las filas marcadas, en el orden del script:

- **Prueba...** las ejecuta en B en una transacción que siempre se revierte.
  Cada sentencia se ejecuta bajo su propio savepoint y se informa por separado:
  ejecutada, fallida con el error de PostgreSQL, u omitida. No se conserva nada,
  pero mientras dura toma los mismos bloqueos que aplicar.
- **Aplicar a B...** las ejecuta en B en una sola transacción: todas o, ante el
  primer error, ninguna, con la sentencia fallida nombrada y las posteriores
  marcadas como no ejecutadas. Cada sentencia espera como mucho 5 segundos por
  su bloqueo.
- **Exportar changeset...** las guarda como migración de Flyway
  (`V<marca de tiempo>__<nombre>.sql`) o como changelog de Liquibase (SQL con
  formato o YAML, un changeset por sentencia, con su id y autor), donde elijas.

Antes de una prueba o de aplicar, una revisión muestra, leído del propio B:

- cada sentencia con el bloqueo que toma y sobre qué objeto - ACCESS EXCLUSIVE
  detiene toda lectura y escritura de esa tabla hasta que acaba la transacción,
  SHARE deja seguir las lecturas - y si elimina, separa o reescribe algo;
- las sentencias que no pueden ejecutarse dentro de un bloque de transacción
  (cualquier CONCURRENTLY, ALTER TYPE ... ADD VALUE antes de PostgreSQL 12,
  ALTER SYSTEM): se omiten y se listan, para ejecutarlas por separado. La
  excepción son `CREATE INDEX CONCURRENTLY` y `DROP INDEX CONCURRENTLY`: aplicar
  las ejecuta tras el COMMIT, una a una en el orden del script, y la primera
  que falla detiene el resto; la prueba las comprueba como sentencia normal;
- la copia: antes de aplicar, las definiciones de cada objeto de B que las
  sentencias cambian o eliminan se guardan como SQL que las restaura; el
  informe enlaza con ella y Trabajos en segundo plano la conserva. Lo que no
  sabe escribir - un rol, un cast, un objeto de estadísticas, una intercalación,
  objetos de búsqueda de texto, operadores - se nombra con el motivo. Las filas
  no están en la copia: una tabla o columna que la aplicación elimina vuelve
  vacía desde ella;
- lo que tu inicio de sesión no puede hacer en B, preguntado a B: un objeto
  cuyo dueño es un rol del que no eres miembro, un esquema en el que no puedes
  crear, un rol al que no puedes entregar un objeto.

Cuando algo se elimina, se separa o se reescribe, **Aplicar a B** espera a que
escribas el nombre del esquema (dos esquemas) o de la base (dos bases) de B.

Tras aplicar, los dos se comparan de nuevo enseguida; el informe queda encima
del nuevo resultado, que es lo que falta. Solo se aplica al B con el que se
comparó el resultado: si el diálogo apunta a otro sitio, pide comparar de
nuevo. Cada prueba y cada aplicación entra en el registro de auditoría con sus
sentencias; una aplicación revertida se registra como fallida, con la
sentencia que falló.

#### Entregar la comparación

**Informe PDF** convierte lo que hay en pantalla en un documento: de qué
servidor viene y cuándo, los recuentos, la tabla de lo que difiere y el script
de sincronización. Se construye a partir de la misma respuesta que muestra la
pantalla, así que ambas no pueden discrepar.

El archivo se escribe donde usted elija, desde la aplicación y no desde el
navegador, y un informe existente nunca se reemplaza sin preguntar.

Un límite que conviene conocer: el informe usa las fuentes que todo lector de
PDF ya tiene, que cubren por completo el alemán y el español pero no la č, la
ć ni la đ croatas. Esas se transliteran - «sifra», no un hueco - de modo que
un nombre sigue siendo legible aunque pierda su acento.

#### Qué significan los cuatro estados {#what-the-four-statuses-mean}

Todo el resultado de una comparación es una palabra por fila, así que conviene
ser preciso. Ambos diálogos comparan **estructura, no datos**: definiciones,
propiedades y permisos. No se lee ninguna fila de ningún lado, salvo que pida
también el estado actual (más abajo), ni se escribe nada en ninguno hasta que
apliques las filas marcadas a B.

| | |
|---|---|
| **same** | Existe en ambos lados y es idéntico. Un esquema cuyos objetos coinciden todos queda oculto mientras *differences only* esté marcado. |
| **changed** | Existe en ambos lados con el mismo nombre, pero las dos definiciones no son el mismo texto: una columna añadida, un tipo ampliado, un cuerpo de función distinto, una propiedad puesta a otro valor. Haz clic en la fila para ver A y B lado a lado (ver [A y B lado a lado](#a-and-b-side-by-side)). |
| **only in A** | Existe en el origen y no en el destino. Un script de sincronización lo crea en B. |
| **only in B** | Existe en el destino y no en el origen. Un script de sincronización lo **elimina**, porque igualar B a A significa quitar lo que A no tiene. |

La dirección no es simétrica: A es el origen, B el destino, y todo script que
salga de estos diálogos cambia B para que se parezca a A.

#### Acotar lo que se compara {#restricting-what-is-compared}

**Qué comparar** contiene los filtros, y se aplican igual a la comparación y al
script que genera: una comparación que ignora los índices no debe producir un
script que los toque.

- **Tipos de objeto**: tablas, vistas, vistas materializadas, secuencias,
  funciones. Desactivar uno deja la comparación ciega a él.
- **Dentro de una tabla**: columnas, índices, claves foráneas, claves primarias,
  restricciones únicas, restricciones check. Así se obtiene una comparación que
  solo mira claves foráneas, o una que ignora las diferencias de índices - útil
  cuando dos entornos están indexados a propósito de forma distinta.
- **Por nombre**: una lista de inclusión y otra de exclusión, ambas admiten `*`.
  `audit_*` en el campo de exclusión mantiene la comparación en lo que importa.
- Seis ajustes rápidos hacen las combinaciones habituales de un clic: todo, solo
  tablas, solo claves foráneas, solo secuencias, estructura sin índices, vistas y
  funciones.
- **Permitir sentencias DROP** decide si el script de sincronización puede quitar
  lo que solo existe en B. Desmárcalo y el script solo crea y modifica, que es lo
  seguro cuando B no es tuyo para podarlo.
- **Mover las tablas al tablespace y al método de acceso de A** está apagado
  hasta que lo marques: mover una tabla la reescribe entera bajo un bloqueo
  ACCESS EXCLUSIVE. Apagado, la diferencia se sigue mostrando y el script dice
  por qué la deja.
- **Crear y eliminar índices CONCURRENTLY** está apagado hasta que lo marques.
  Marcado, los cambios de índices de las tablas que B ya tiene se escriben como
  `CREATE INDEX CONCURRENTLY` y `DROP INDEX CONCURRENTLY`, en una sección
  propia tras la transacción: las lecturas y escrituras en la tabla siguen
  mientras se crea un índice. No se deshacen con la transacción, y una creación
  que falla (un índice único sobre filas duplicadas, por ejemplo) deja un
  índice INVALID, que `IF NOT EXISTS` luego pasa por alto: el informe lo dice;
  elimínalo con `DROP INDEX CONCURRENTLY`, quita la causa y aplica de nuevo
  (*validez y uso de los índices* en el estado actual lo muestra). Una tabla
  particionada, un índice
  detrás de una restricción y el índice que usa REPLICA IDENTITY se quedan
  dentro de la transacción.
- **Contar un orden de columnas distinto como diferencia** está apagado.
  PostgreSQL no puede mover una columna, así que ninguna sentencia puede poner
  las columnas de B en el orden de A. Apagado, una tabla con las mismas columnas
  en otro orden compara como *same* y lleva una nota - *columnas en otro orden* -
  que muestra los dos órdenes, el de B tal como quedará después del script (una
  columna que el script añade queda al final). Marcado, el orden cuenta y el
  script dice que hay que reconstruir la tabla a mano para igualarlo.

#### Particiones, herencia y almacenamiento {#partitions-inheritance-and-storage}

Una partición se compara con su lugar en el árbol: su tabla padre y su límite
(`FOR VALUES ...`), y una tabla particionada con su clave de partición. El
script de sincronización mueve particiones con **ATTACH PARTITION** y **DETACH
PARTITION**, nunca borrándolas y creándolas de nuevo, porque una partición
guarda filas:

- un límite que cambió: se separa y se vuelve a unir con el límite de A;
- una partición que A tiene unida y B mantiene como tabla propia: se une; al
  revés: se separa;
- una partición bajo otra tabla padre: se separa de la de B y se une a la de A;
- una partición que solo tiene B, bajo una tabla padre que tienen ambos lados:
  se separa y se conserva como tabla propia. Se borra después solo si está
  marcado **Permitir sentencias DROP**.

Cada DETACH va primero, en una sección propia, para que un límite escrito más
abajo no se solape con una partición todavía unida. La cabecera del script
indica los bloqueos: DETACH toma ACCESS EXCLUSIVE sobre la tabla padre y la
partición; ATTACH toma SHARE UPDATE EXCLUSIVE sobre la tabla padre (ACCESS
EXCLUSIVE antes de PostgreSQL 12) y lee la partición para probar que sus filas
caben en el límite. En PostgreSQL 14 y posteriores, tras cada DETACH va su forma
`... CONCURRENTLY` como comentario: toma un bloqueo más débil, pero no puede
ejecutarse dentro de un bloque de transacción ni cuando la tabla padre tiene una
partición DEFAULT, así que ejecútala por separado si la usas.

Una clave de partición no se puede cambiar en su sitio. Cuando A y B
particionan una tabla de forma distinta, el script lo dice y no escribe nada
ejecutable para ella, y lo que A pone bajo esa tabla espera, como comentario,
hasta que la clave se resuelva a mano.

**INHERITS.** Una tabla que hereda de otra (herencia clásica, no una partición)
se compara con sus tablas padre. El script escribe `ALTER TABLE ... INHERIT` o
`NO INHERIT`, y un hijo nuevo se crea solo con sus propias columnas, después de
su padre.

**Tablespace y método de acceso** forman parte de la definición de una tabla:
una tabla en otro tablespace, o guardada con otro método de acceso, compara
como changed. Una tabla nueva se crea donde la tiene A; una existente solo se
mueve con el interruptor de arriba.

**Una columna que cambia de tipo bajo vistas.** PostgreSQL rechaza `ALTER
COLUMN ... TYPE` mientras una vista, una vista materializada o una regla lee la
columna. El script elimina cada una de ellas en B justo antes del cambio - y
también las vistas construidas sobre ellas, en cualquier esquema, la más
externa primero - y las vuelve a crear justo después, tal como están en B:
definición y opciones, los índices y las filas de una vista materializada
(REFRESH), los valores por defecto de columnas y los disparadores INSTEAD OF de
una vista, reglas, propietario, comentarios, permisos y permisos de columnas.
Cada DROP y CREATE mantiene un bloqueo ACCESS EXCLUSIVE sobre su objeto hasta
que termina la transacción, así que las consultas sobre ellos esperan; la
revisión marca cada uno de esos DROP como uno que vuelve. Lo que no se puede
reponer desde aquí - un disparador cuyo WHEN lee la columna, una política, una
columna generada, el cuerpo de una función - el script lo nombra como trabajo
manual antes de ejecutarse.

**Una columna que se elimina bajo vistas.** PostgreSQL rechaza `DROP COLUMN`
mientras una vista, una vista materializada o una regla lee la columna. El
script elimina cada una de ellas en B justo antes de la columna - también las
vistas construidas sobre ellas, la más externa primero - y después cada una
pasa a ser lo que el resto del script hace con ella: una que solo está en B
sigue eliminada; una que A define de otra forma, sin la columna, se crea tal
como está en A, con el propietario, los permisos y los comentarios de B (y los
comentarios de columnas, permisos de columnas, reglas, disparadores e índices
que no nombran una columna perdida), y su propio cambio no se escribe una
segunda vez más abajo; cualquier otra vuelve tal como está en B. La revisión
marca cada DROP que vuelve. Una vista que volvería leyendo todavía la columna -
una en otro esquema, una que el script no toca, una construida sobre una vista
que cambia o desaparece - no puede volver, y tampoco un disparador cuyo WHEN
lee la columna, una política o el cuerpo de una función: entonces el script
dice cuál tiene que cambiar primero y no elimina esa columna (escribe el DROP
como comentario), porque un DROP que falla deshace toda la ejecución. Cambie
ese objeto y compare de nuevo.

#### Propietario y permisos {#owner-and-grants}

Los interruptores **propietario** y **permisos** deciden dos cosas a la vez: si
cuenta una diferencia en quién es dueño de un objeto o quién puede usarlo, y si
el script de sincronización escribe los ALTER ... OWNER TO, GRANT y REVOKE que
la cierran. Ambos están activados por defecto; desactive uno y esa clase de
diferencia ni se informa ni se escribe. Los permisos incluyen los de columnas,
los de la secuencia de una columna identity, lo que un propietario se revocó a
sí mismo, y los del propio esquema junto con los privilegios por defecto
definidos en él, que aparecen como una fila llamada *(this schema)*. Un permiso
se lee como PostgreSQL lo aplica: un objeto sobre el que nadie concedió nada y
uno cuyos permisos se concedieron y se volvieron a quitar comparan igual. Cuando el script nombra un rol que no existe en el destino, su
cabecera dice cuáles.

#### Comparar también el estado actual {#comparing-the-current-state-too}

La comparación es por definición: dos esquemas son iguales cuando cada objeto
de ellos se crearía igual. **Comparar también el estado actual** añade lo que
tiene una base en ejecución y no tiene una definición. Cada interruptor está
desactivado por defecto, porque dos copias del mismo esquema siempre difieren
en estado.

| | |
|---|---|
| **valores actuales de secuencias** | Dónde está cada secuencia y si ya se ha llamado. El script fija las de B con setval(). |
| **si una vista materializada está poblada** | El script ejecuta REFRESH MATERIALIZED VIEW, o REFRESH ... WITH NO DATA cuando la de A está vacía. |
| **número de filas** | Tablas, particiones y vistas materializadas. La estimación sale de las estadísticas del planificador y cuenta como diferencia cuando las dos se separan un 5% o más; el **recuento exacto** ejecuta count(*) en cada tabla, lee todas las filas y compara exactamente. |
| **tamaños** | Tamaño total, con índices y TOAST. Distintos cuando se separan al menos 1 MB y además un 10% o 64 MB. |
| **validez y uso de los índices** | Un índice que un CREATE INDEX CONCURRENTLY fallido dejó no válido, y cuántas veces se ha usado cada índice: distinto cuando un lado lo ha usado y el otro nunca, o cuando los dos recuentos se separan en más de 1000. |

El número de filas, los tamaños y el uso de índices solo se informan y nunca se
escriben en el script - ninguna sentencia de sincronización los igualaría - y el
script termina con la lista de objetos que difieren en ellos. En la comparación
de bases de datos, las particiones de una tabla particionada que difieren se
nombran, no solo se cuentan.

#### Comparar lo que un rol puede hacer, en dos lados {#comparing-what-a-role-may-do-on-two-sides}

**Comparar usuarios** responde a "¿puede este rol hacer aquí lo mismo que
allí?", y lo pregunta en ese orden: qué roles, desde dónde, y hasta qué nivel
mirar.

Los roles en PostgreSQL son de todo el clúster - un `pg_authid` por clúster - 
así que la lista de roles es lo que tienen los dos *servidores*, y un rol que
falta en un lado se informa en vez de omitirse. Los privilegios son lo
contrario: viven en el catálogo de cada base, por eso importan los selectores de
base y por eso la comparación es por base. Nombrar esquemas acota más; dejarlo
vacío compara todos los esquemas que la conexión ve.

**PUBLIC** - todos los roles, actuales y futuros - tiene su propia fila en la
lista de roles. No tiene atributos que comparar, solo permisos: **compare
privileges** en esa fila muestra lo concedido a PUBLIC en cada lado, y el
script de sincronización escribe los GRANT ... TO PUBLIC y REVOKE ... FROM
PUBLIC que lo cierran.

Se leen siete niveles, y el primero es del que dependen todos los demás:

| | |
|---|---|
| **database** | `CONNECT`, `CREATE`, `TEMPORARY` - si el rol puede siquiera entrar |
| **schema** | `USAGE`, `CREATE` |
| **table** | y vistas, vistas materializadas, tablas particionadas y foráneas |
| **column** | un rol puede tener `SELECT` sobre tres columnas y nada sobre la tabla, lo que toda comprobación a nivel de tabla lee como "sin acceso" |
| **sequence** | `USAGE`, `SELECT`, `UPDATE` |
| **function** | `EXECUTE` |
| **default** | lo que llevarán los objetos *nuevos* - un rol puede estar bien concedido sobre todo lo que existe hoy y perder el acceso a todo lo que se cree mañana |

Los permisos se leen directamente de las ACL del catálogo, no con
`has_table_privilege`. Esa función responde "¿puede este rol hacer X?"
incluyendo todo lo heredado por pertenencia, y un permiso heredado no se puede
conceder ni revocar sobre este rol. Solo se puede actuar sobre un permiso
directo, así que solo esos se comparan; la pertenencia se compara como atributo
del rol.

**El script de sincronización** iguala un lado al otro, en la dirección que
elijas. Ninguno de los dos lados es automáticamente el correcto - producción
puede ser el que se desvió, o el que está bien - así que se ofrecen ambas
direcciones y ninguna viene puesta de forma que oculte la elección.

Los permisos que tiene el destino y no el origen se escriben como comentarios
hasta que marques *también revocar*. Dar un permiso se puede deshacer; quitarlo
a alguien en mitad de su jornada, no.

Aplicar desde la comparación de bases muestra primero las sentencias.
**Aplicar** antes concedía los permisos con el clic y luego informaba de un
recuento; ahora abre el mismo diálogo que recibe todo script generado - copiar,
guardar, abrir en el editor SQL - con Aplicar al lado, y ejecuta sólo lo que se
te mostró. Si los permisos cambian entre mirarlos y aplicarlos, la aplicación se
detiene y te entrega la lista nueva en vez de ejecutar otra cosa.

No se ejecuta nada. Lee el script, guárdalo con **Guardar script…** y ejecútalo
cuando lo decidas.

### 5.13 Clonar una base de datos {#cloning-a-database}

Clic derecho en una base → **Clonar base de datos (mismo servidor)**.

Es una sola sentencia - `CREATE DATABASE "copia" TEMPLATE "original"` - y
PostgreSQL copia la base a nivel de archivos en lugar de volcarla y restaurarla.
Por eso es rápido: una base de 25 MB se clona en bastante menos de un segundo, y
el tiempo depende del tamaño, no del número de objetos.

La copia es completamente independiente desde que existe. Escribir en una no
afecta a la otra; no comparten almacenamiento.

#### El coste, y por qué el diálogo insiste

Una base no puede servir de `TEMPLATE` mientras algo esté conectado a ella, así
que antes de copiar DBApguin ejecuta:

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'origen'
```

**Se corta toda conexión abierta al origen**, incluidas las de otras personas, y
se aborta cualquier transacción en curso. Es un requisito de PostgreSQL, no una
decisión de DBApguin, y por eso el diálogo te pide marcar una casilla en lugar de
dejarte pasar sin más.

**Tu propia conexión se va con ellas.** La sesión de DBApguin sobre la base de
origen se corta como cualquier otra, y el pool abre en silencio una nueva para la
siguiente petición, así que la aplicación sigue y nada parece ir mal. Lo que no
sobrevive es todo lo atado a la sesión antigua: una transacción abierta en el
editor SQL se revierte, las tablas temporales desaparecen, los `SET` de sesión
vuelven a sus valores por defecto, los advisory locks se liberan y una consulta en
curso se aborta. Confirma o termina lo que estés haciendo antes de clonar.

#### Qué pasa y qué no

Todo lo que está *dentro* de la base se copia. Las propiedades atadas *a* la base
no: pertenecen al catálogo del clúster, no a los archivos que se copian:

| Se copia | No se copia |
|---|---|
| tablas, filas, índices, restricciones | el **propietario** de la base - el clon es de quien clonó |
| secuencias, con su valor actual | `GRANT CONNECT` / `TEMPORARY` sobre la base |
| vistas, vistas materializadas, funciones, tipos | ajustes por base (`ALTER DATABASE … SET work_mem`, `search_path`, …) |
| extensiones | el comentario de la base |
| privilegios de tablas y columnas | roles y usuarios (son de todo el clúster y ya existen) |
| comentarios en objetos, políticas RLS, objetos grandes | |

La columna derecha es la parte que sorprende. Una base de producción clonada para
pruebas llega siendo tuya, sin permisos de `CONNECT` para el rol de la aplicación,
y con los valores por defecto del servidor para `work_mem` y `search_path` en vez
de los que tenía el original. Los datos son idénticos; el entorno no.

Para restaurar esas cuatro cosas, sobre el clon:

```sql
ALTER DATABASE copia OWNER TO propietario_original;
GRANT CONNECT, TEMPORARY ON DATABASE copia TO rol_aplicacion;
ALTER DATABASE copia SET work_mem = '64MB';
COMMENT ON DATABASE copia IS '…';
```

#### Cuándo usarlo

- antes de una migración o un cambio arriesgado, para tener un estado exacto al que volver
- para una copia de prueba o desarrollo en el mismo servidor
- **no** para mover una base a otro servidor: `TEMPLATE` solo funciona dentro de un
  clúster; para eso están la copia y la restauración
- **no** sobre una base que la gente está usando ahora mismo: los desconectará
- un clon ocupa tanto disco como el original; no se comparte nada

### 5.14 Renombrar una base de datos {#renaming-a-database}

Clic derecho en una base de datos -> **Rename database**.

Es una sola sentencia, `ALTER DATABASE "viejo" RENAME TO "nuevo"`, y cambia
**solo la entrada del catálogo**. No se copia ni se mueve nada, así que es
instantánea sea cual sea el tamaño de la base. El propietario, cada `GRANT`, el
comentario y todos los ajustes por base de datos (`ALTER DATABASE ... SET ...`)
siguen adjuntos y sin cambios - el nombre es lo único distinto después.

#### No puede ejecutarse mientras haya alguien conectado

PostgreSQL rechaza `ALTER DATABASE ... RENAME` mientras haya cualquier sesión
conectada a esa base - incluido el propio pool de conexiones de DBApguin si has
estado navegándola. Sin ayuda, la sentencia falla con:

```
database "shop" is being accessed by other users
DETAIL: There is 1 other session using the database.
```

Así que el diálogo comprueba primero. Si la base está inactiva, el renombrado
simplemente se ejecuta. Si no lo está, el diálogo muestra **cuántas conexiones
hay y qué usuarios las tienen**, y ofrece terminarlas - el mismo
`pg_terminate_backend(...)` que usa Clone. Es destructivo: una transacción
abierta sobre esa base se revierte, las tablas temporales y los `SET` de sesión
desaparecen, y una consulta que se esté ejecutando en ese momento se aborta.
Tienes que marcar la casilla.

Las propias conexiones en pool de DBApguin se terminan con las demás; el pool
simplemente abre otras nuevas contra el nombre nuevo en la siguiente petición.
La conexión que ejecuta el renombrado está en la base `postgres`, nunca en el
destino, así que el renombrado no puede cortarse a sí mismo.

#### Tu conexión guardada y las pestañas abiertas

Si renombras la base de datos a la que esta ventana está **conectada ahora
mismo**, DBApguin actualiza la entrada de conexión guardada al nombre nuevo,
cambia la conexión activa y recarga el árbol de objetos. Las pestañas de SQL y
de esquema ligadas al nombre viejo se marcan obsoletas con una insignia hasta
que las refrescas, igual que en cualquier cambio de base. Una conexión guardada
para *otra* ventana que también apuntaba al nombre viejo no se toca -
actualízala a mano.

#### Lo que no arregla

El renombrado es un cambio de catálogo y nada fuera del catálogo lo sabe: una
aplicación con el nombre viejo de la base en su cadena de conexión, una entrada
en `~/.pg_service.conf`, un script de copia, una definición de servidor
`dblink` o `postgres_fdw`, o SQL que fija el nombre. `current_database()` en una
función devuelve el nombre nuevo desde el momento en que el renombrado hace
commit. Renombra una base de producción solo cuando puedas actualizar todo lo
que la alcanza en la misma ventana de mantenimiento.

`postgres`, `template0` y `template1` no se pueden renombrar - el diálogo los
rechaza - porque initdb y muchísimas herramientas dan por hecho que esos
nombres existen.

### 5.15 Extensiones, y qué hace «mover» {#extensions-and-what-move-does}

**Base de datos -> Gestionar extensiones** lista lo que el servidor tiene
disponible y lo que esta base tiene instalado, con la versión instalada junto a la
predeterminada, de modo que un candidato a `ALTER EXTENSION ... UPDATE` se ve de un
vistazo.

Una extensión es un paquete: funciones, operadores, tipos, a veces tablas, todo
instalado en un esquema. **Mover** ejecuta

    ALTER EXTENSION "nombre" SET SCHEMA "destino"

y traslada el paquete entero con una sola sentencia.

Por qué lo harías:

- **Para que `public` sea tuyo.** `CREATE EXTENSION pg_trgm` sin esquema deja unas
  cuantas docenas de funciones en el primer esquema del `search_path`, normalmente
  `public`, mezcladas con tus propias tablas.
- **Seguridad.** Quien pueda crear objetos en un esquema puede definir una función
  con el mismo nombre y los mismos tipos de argumento que una de la extensión. Si
  ese esquema va antes en el `search_path`, se ejecuta la suya. Tener las
  extensiones en un esquema `ext` donde solo escribe el propietario elimina esa
  posibilidad. Por eso la receta habitual es
  `CREATE SCHEMA ext; CREATE EXTENSION pg_trgm SCHEMA ext;`

Lo que no hará:

- **Las extensiones no reubicables no se pueden mover.** La propia extensión lo
  declara (`pg_extension.extrelocatable`), normalmente porque su código nombra su
  esquema. PostGIS es el caso conocido. DBApguin las marca como
  `(non-relocatable)` y no ofrece el botón Mover, en vez de dejar que el servidor
  lo rechace.
- **No actualiza tu código.** Todo lo que nombre explícitamente el esquema antiguo
 - el cuerpo de una vista, una función, una consulta de la aplicación - se rompe.
  Las llamadas sin calificar siguen funcionando solo si el nuevo esquema está en el
  `search_path`: `ALTER DATABASE app SET search_path = public, ext;`
- **No es una actualización de versión.** Para eso está Actualizar.

Mover requiere ser propietario de la extensión, lo que en la práctica significa
superusuario. La sentencia es rápida - reescribe entradas del catálogo, no datos-
pero toma un bloqueo sobre los objetos de la extensión, así que hazlo cuando nada
esté trabajando contra ellos.

### 5.16 Lista de tareas

Una alarma, una recomendacion de un advisor y un hallazgo de una comprobacion
dicen todos "alguien deberia hacer esto" y luego desaparecen. Un cluster con IDs de transaccion de 1.200 millones de antiguedad
con nueve tablas por encima del umbral es el trabajo de
una manana, no un aviso pasajero.

**View -> Lista de tareas**, con el numero de tareas abiertas en el propio menu;
el numero se vuelve rojo cuando alguna es critica.

Las tareas llegan desde tres sitios: desde una **notificacion**, desde la
**ventana de alarma critica** (confirmar significa "lo he visto", no "esta
resuelto") y desde **cualquier fila de advisor con severidad**. Tambien puede
escribir una propia; esa nunca se fusiona con nada ni se marca como obsoleta.

Cada tarea guarda el servidor, la base de datos y el objeto del que vino. **Las
contrasenas nunca se guardan**: la conexion se registra como host, puerto, base
de datos y usuario, solo para identificarla.

**Una tarea cuya condicion ya no existe lo dice.** Permanece en la lista y
simplemente deja de afirmar que sigue vigente. **Exportar lista** escribe la lista
como texto plano con casillas `[ ]`, el objeto, el servidor y su propia nota.

### 5.17 Servidores foráneos y asignaciones de usuario {#foreign-servers-and-user-mappings}

**Database -> Manage foreign servers** enumera cada servidor foráneo de esta
base de datos con el wrapper que lo sostiene, su propietario, sus opciones y las
asignaciones de usuario que cuelgan de él.

Crear un servidor foráneo era un asistente; todo lo que venía después era el
editor SQL. Un host remoto se muda, un puerto cambia, una contraseña remota
guardada caduca, un servidor hay que entregarlo a otro propietario: todo
corriente, nada de ello alcanzable desde el lugar que creó el servidor, y las
opciones que el asistente había pedido no se mostraban de vuelta en ninguna
parte. Esta es esa otra mitad.

#### De qué está hecho un servidor foráneo

Tres objetos, y los tres tienen que coincidir antes de que pase una sola fila:

| objeto | qué lleva | de quién es |
|---|---|---|
| el wrapper | `postgres_fdw`, `mysql_fdw`, `file_fdw`, ... | de la extensión |
| el servidor | dónde está el otro extremo: host, puerto, dbname | de la base de datos |
| la asignación | quién eres allí: usuario remoto y contraseña | de un rol local |

La asignación es la parte que se olvida. PostgreSQL la busca **por el rol que
realmente se conectó**, así que un servidor con una asignación solo para
`postgres` es invisible para todos los demás: sus consultas fallan con *"user
mapping not found"* por correcto que esté el servidor. Por eso se ofrece una
asignación para cualquier rol de login del clúster y no solo para ti mismo y
`PUBLIC`: un rol local a un login remoto es la forma normal, y la razón de que
las credenciales remotas sean por rol.

Una asignación **para un rol de grupo NOLOGIN** la acepta el servidor y luego no
la usa nunca, porque ese rol no se conecta jamás. Esos roles, y los `pg_*`
integrados, quedan fuera del selector a propósito.

#### Editar un servidor

**Edit** muestra las opciones que el servidor lleva ahora, cada una como campo
editable, con sitio para claves que todavía no tiene. Solo se envía lo que
cambias.

**Vaciar el valor de una opción elimina la clave.** PostgreSQL no tiene opciones
con cadena vacía, y `OPTIONS (SET k '')` es una sentencia distinta de que la
clave no exista, así que el campo vacío es la manera de decir "quita esto".

Si una clave necesita `ADD`, `SET` o `DROP` depende de si el servidor ya la
lleva, y eso se lee del catálogo en la misma transacción que el `ALTER`, no se
adivina de lo que la pantalla mostró por última vez. Un cambio de cuatro
opciones sale como una sola sentencia:

    ALTER SERVER "loopback" OPTIONS (SET host '10.0.0.9', SET port '5433',
                                     DROP dbname, ADD connect_timeout '7')

**Version** y **Owner** son sentencias `ALTER SERVER` separadas a propósito.
PostgreSQL las acepta en una, pero entonces una sola clave de opción errónea se
lleva consigo el cambio de propietario, y el error no nombra ninguno de los dos.
Todas van en una transacción, así que un cambio a medio aplicar no es un estado
en el que puedas acabar.

#### Contraseñas

La contraseña remota de una asignación nunca se muestra. PostgreSQL no devuelve
a un cliente una contraseña guardada, y una opción de servidor llamada
`password` se enmascara aquí por la misma razón, incluido en el SQL que se
devuelve y en el registro de auditoría, que es un fichero que la gente exporta y
reenvía. El campo de contraseña del editor de asignaciones está por eso vacío y
solo viaja si escribes algo en él.

#### Eliminar

Un servidor que todavía tiene tablas foráneas rechaza un `DROP` simple, así que
el diálogo ofrece `CASCADE` de forma explícita y cuenta las asignaciones de
usuario que se van con él. CASCADE se lleva también cada tabla foránea de ese
servidor: son entradas de catálogo locales, así que nada del otro extremo se
toca, pero las consultas y vistas que leen a través de ellas se rompen.

**Drop user mapping** es el corte más estrecho: ese rol pierde sus credenciales
para el servidor y sus consultas fallan hasta que exista de nuevo una
asignación, mientras cualquier otro rol sigue funcionando.

## 6. Flujos de trabajo habituales

### 6.1 Triaje de rendimiento en una base lenta

1. Abre la conexión.
2. Ejecuta primero `db-health` - el resumen en una pantalla.
3. Si el acierto de caché es bajo: `db-cache-advisor` para el detalle por objeto.
4. Si dominan los seq scans: `db-index-advisor` para candidatos a índice.
5. Si las consultas son lentas: `db-stat-statements` para las 25 primeras con indicadores por consulta. Ordena por total_exec_time o toma las marcadas con AVISO.
6. Si sospechas de bloat: `db-bloat-breakdown` para HEAP+TOAST+índices por tabla. Para una medición exacta (y lenta), `db-pgstattuple`.

### 6.2 Problema de bloqueos

1. `db-health` -> categoría BLOCKING, y al detalle para ver los pares bloqueador/bloqueado.
2. O la vista Sesiones -> Lista, ordenando por blocking_pids no vacío.
3. Clic derecho en la sesión bloqueadora y Matar cuando estés listo.

### 6.3 Retraso de replicación

1. `db-replication` para el estado de la réplica y el retraso de escritura/flush/replay en tiempo y bytes.
2. `db-repl-slots` para la retención de WAL por slot. Los slots inactivos que retienen gigabytes de WAL son la causa clásica de un disco lleno.
3. Para la lógica: `db-logical-replication` (lado del suscriptor) muestra suscripciones DESACTIVADAS / NO INICIADAS y retraso de aplicación >=16 MB.
4. `db-replica-identity` si los suscriptores se quejan de identidad ausente.

### 6.4 Auditoría de seguridad

1. `db-roles-list` para los hallazgos de seguridad de todo el clúster.
2. `db-role-perms` (elige el rol auditado) para profundizar por objetivo.
3. `db-ownership` para los hallazgos sobre propiedad de objetos.
4. Exporta el registro de auditoría desde Ajustes -> menú Ver -> Registro de auditoría.

### 6.5 Migración de esquema con seguridad

1. Compara dos bases desde la vista Servidor -> Comparar esquemas.
2. Revisa el SQL de sincronización generado.
3. Aplica las sentencias elegidas.

### 6.5.5 ¿De verdad ayudaría este índice?

`db-index-advisor` informa de qué tablas se están leyendo secuencialmente. La
pregunta siguiente es siempre si un índice sobre *esta* columna lo arreglaría,
y la única forma honesta de responderla era construir uno: un bloqueo de
escritura largo para una respuesta que a menudo es «el planificador no lo
habría usado igualmente».

`EXPLAIN` tampoco la responde. El planificador calcula el coste de los índices
que existen, así que sin el índice no hay plan con él, y `EXPLAIN ANALYZE` lo
necesita real por el mismo motivo.

Debajo de la tabla del advisor hay un panel que la responde sin construir nada.
Elija la tabla, escriba el índice que está considerando, pegue la consulta que
le interesa y pulse **Compare plans**. Informa de:

- si el planificador usaría siquiera el índice - la respuesta frecuente es que
  no, y conviene saberlo antes de construir un índice de 4 GB;
- el coste estimado con y sin él, y la diferencia;
- aproximadamente cuánto espacio ocuparía el índice.

Deliberadamente no informa de un tiempo. No se ejecuta nada, así que no hay
medición que dar, y presentar una estimación como si lo fuera sería peor que no
dar ninguna.

Para esto hace falta la extensión `hypopg`, que crea el índice solo en la
cabeza del planificador: no se escribe nada ni se toma ningún bloqueo, así que
es seguro incluso en producción. Está disponible en RDS y Aurora; en
instalaciones propias viene como paquete aparte. Si la extensión no está
instalada, el panel lo dice, y dice también si este servidor podría tenerla.

### 6.6 Análisis de carga al estilo Oracle (AWR + ASH + trazado) {#oracle-style-workload-analysis-awr-ash-trace}

#### Cuándo se ejecuta: el plan de captura {#capture-plan}

Las pestañas AWR, ASH y Traza de sesión empiezan igual: una fila **Cuándo** bajo la cabecera.

- **Instantánea ahora** (solo AWR): una instantánea, con un comentario opcional que aparece junto a ella en cada lista.
- **Durante un tiempo**: empieza ahora y se detiene sola. Junto a duraciones rápidas hay una rueda de días, horas, minutos y segundos, así que se puede elegir cualquier duración dentro de los límites de la herramienta: AWR de 10 minutos a 92 días, ASH de 1 minuto a 31 días, la traza de 10 segundos a 30 minutos.
- **Programado**: de una fecha y hora a otra, elegidas con el selector de fecha y hora según el reloj de este ordenador, opcionalmente **cada día**. Una traza se programa con un inicio y una duración, hasta un día de antelación.
- **Hasta detenerlo**: lo que hacían antes el planificador y el muestreador (no para la traza).

**Cada** es la frecuencia con la que la herramienta recoge, con sus propias opciones rápidas y la misma rueda: AWR una instantánea cada 1 minuto a 1 día, ASH una muestra cada 1 segundo a 1 hora, la traza una lectura cada 0,1 segundos a 1 minuto. **Conservar** son los días que se guardan las instantáneas o los días almacenados. En AWR, **también muestrear las sesiones cada 5 s** inicia el muestreador ASH para la misma ventana; en un plan de hasta una hora toma en su lugar el informe de carga (una instantánea en cada extremo con las sesiones muestreadas entre ambas), antes la vista Capture.

Sobre el botón, una frase dice qué ocurrirá, cuántas instantáneas o muestras son, lo que ocupa en disco y cuesta en el servidor y qué se abre al final; avisa cuando una elección es pesada (una instantánea AWR más a menudo que cada 5 minutos). Mientras un plan se ejecuta, el panel se pliega en una barra: qué se ejecuta, desde y hasta cuándo, una línea de progreso, el tiempo restante, **Cambiar** y **Detener**; un plan programado dice cuándo empieza y ofrece **Iniciar ahora**. Cuando termina un plan con fin, el informe de su periodo se abre solo: en AWR su primera instantánea frente a la última, en ASH el informe de su ventana.

En AWR el **Comentario** junto a la fila Cuándo está en todos los modos: va en la única instantánea de *Instantánea ahora* y en cada instantánea de un plan. El selector de fecha y hora de *Programado* tiene horas rápidas hacia delante (en 1 h, esta noche 02:00, mañana 08:00, lunes 00:00) y ruedas para la hora y el minuto; el chevrón al final de la fila Cuándo pliega el panel en una línea en pantallas pequeñas.

**Detener** termina un plan antes de tiempo y funciona como su propio final: lo recogido se queda, AWR toma una última instantánea en ese momento (con el comentario del plan) y se abre el informe del periodo: la primera instantánea de la ejecución frente a esa última, o en ASH el tiempo que muestreó. Un plan programado que aún no ha empezado solo se cancela.

Los planes se ejecutan mientras DBApguin está abierto: no queda nada en el servidor y nada se ejecuta solo. Un plan a mitad de su ventana cuando se cierra la aplicación continúa con el resto la próxima vez que se abra el servidor, uno diario espera a su siguiente ventana y uno cuya ventana pasó con la aplicación cerrada ha terminado. Una traza programada solo vive mientras la aplicación está abierta.

En un Mac, el trackpad hace tic al pasar cada valor de las ruedas, como un selector en un teléfono.

1. **Comprueba la configuración una vez**: ejecuta `db-setup-advisor`. Si `pg_stat_statements`, `auto_explain` o `compute_query_id` aparecen como FALTA, sigue el remedio impreso. Esto se hace una vez por clúster.

2. **Registra un periodo lento**: cuando alguien se queje de lentitud, ejecuta `db-workload-report` durante 5 minutos. Las secciones de SQL principal muestran qué consumió tiempo de verdad en esa ventana.

3. **Observabilidad continua**: cuando quieras horas de historial de sesiones (y no una sola captura de cinco minutos), arranca el muestreador ASH:
   - `db-ash` action=start
   - déjalo en segundo plano (sigue hasta que pulses action=stop o cierres la aplicación)
   - más tarde ejecuta `db-ash` action=report para agregar cualquier rango de fechas UTC

4. **Comparación antes y después de un despliegue**: toma una instantánea antes, otra cuando la carga se estabilice, y compáralas:
   - `db-awr-snapshot` action=take, comment="antes del despliegue v24.07"
   - (despliegue)
   - `db-awr-snapshot` action=take, comment="después del despliegue v24.07"
   - `db-awr-snapshot` action=list (marca los dos y luego **Comparar los dos elegidos**)
   - `db-awr-snapshot` action=compare, snap_id_a=<antes>, snap_id_b=<después>

5. **Trazado profundo de una aplicación**: cuando una aplicación con pool va lenta y sospechas de una consulta concreta:
   - `db-trace-session` target_mode=application_name, target_value=`myapp-worker`, duration_seconds=120, category_filter=`select`, query_filter=`reports`
   - En una psql paralela conectada como ese usuario: `SET auto_explain.log_min_duration = 0; SET auto_explain.log_analyze = on;` y luego provoca la carga
   - El informe muestra las transiciones de consulta y los eventos de espera de cada backend coincidente; los planes los captura el log del servidor

#### Cómo leer un informe ASH {#reading-an-ash-report}

`db-ash` action=report convierte las muestras de la ventana elegida en seis secciones. Las dos primeras son la propia lectura que hace el informe del resto; las tablas que siguen son los recuentos.

**Hallazgos** es lo que destacó, lo peor primero, cada uno con la sentencia detrás y los mismos botones que toda sentencia lleva aquí: copiar, cargar en el editor SQL, EXPLAIN allí, o **Plan** - el plan que realmente se ejecutó, dibujado en el sitio (vea [El plan que realmente se ejecutó](#the-plan-that-actually-ran)). "Mostrar en la línea de tiempo" ilumina la fila de la sesión en la sección siguiente.

| Hallazgo | Salta cuando | Qué hacer |
| --- | --- | --- |
| Bloqueo: sesión N | una sesión retuvo a otras 3 muestras o más; crítico a partir de un minuto perdido o cinco sesiones retenidas | El texto dice si el bloqueador estaba inactivo en una transacción (la aplicación olvidó el commit) o ejecutando una sentencia (una escritura larga). "Una cadena" significa que el bloqueador esperaba a su vez a otro. |
| Una sentencia se llevó el X% del tiempo activo | una sentencia tiene el 20% o más de las muestras en ejecución o espera, y había otras sentencias con las que compararla | El hallazgo dice adónde fue su tiempo: sobre todo bloqueos, sobre todo disco o sobre todo CPU - cada uno apunta a un arreglo distinto. |
| El X% del tiempo activo fueron esperas de bloqueo | el 20% o más de las muestras activas fueron esperas `Lock` | La sección de parejas de bloqueo nombra las sesiones. |
| X% esperando al disco | el 30% o más fueron esperas `IO` | Un conjunto de trabajo mayor que `shared_buffers`, o un escaneo que vacía la caché; las consultas principales dicen qué sentencias. |
| Contención interna (LWLock) | el 20% o más fueron esperas `LWLock` | `WALWrite` es el disco del WAL, `buffer_content` / `buffer_mapping` son páginas calientes, `lock_manager` son muchísimos bloqueos por transacción. |
| Sesión inactiva en una transacción hasta T | una sesión estuvo inactiva dentro de una transacción 5 minutos o más | Conserva sus bloqueos y frena el vacuum de todo el servidor; crítico cuando además bloqueó a alguien. `idle_in_transaction_session_timeout` termina esas sesiones. |
| Sentencias cortadas | el texto de una sentencia muestreada alcanzó `track_activity_query_size` | De un texto cortado no se puede hacer un plan; la sección de ajustes al final dice qué subir. |

**Línea de tiempo** es la ventana como una cuadrícula: una fila por sesión (las cuarenta más ocupadas, el resto sumado en una fila "otras"), una columna por tramo de tiempo - un segundo para una ventana corta, un minuto o más para una larga, dicho en el título. Cada celda se pinta del color de lo que la sesión hizo más en ese tramo:

| Color | Significado |
| --- | --- |
| verde | corriendo en CPU (activa, sin evento de espera) |
| rojo | esperando un bloqueo que tiene otra sesión |
| amarillo | esperando al disco (eventos de espera `IO`) |
| azul | otra espera: `LWLock`, `IPC`, cliente, tiempo de espera |
| morado | inactiva dentro de una transacción - sin trabajar, pero conservando sus bloqueos |
| vacío | no está en las muestras: inactiva o no conectada |

Una celda pálida es aquella en la que la sesión estuvo solo parte del tramo. **Pulsa una celda** para ver las sentencias de esa sesión en ese tramo, con el evento de espera, los pids que la bloquearon (cada uno un botón que salta a su fila) y los botones de la sentencia. **Pulsa la etiqueta de una sesión** para todo lo que hizo en la ventana: su reparto por colores y sus sentencias sumadas. **Pulsa una hora** arriba para ver cada sesión de esa columna. Los pids son del pasado: un backend puede haberse ido o ser ya otra sesión, así que la línea de tiempo no ofrece cancelar ni terminar.

**Histograma de estados**, **Eventos de espera** y **Sesiones principales** son los recuentos; las sesiones principales llevan ahora también las partes de bloqueo, disco e inactividad en transacción de cada sesión.

**Consultas principales** son tarjetas, una por sentencia, ordenadas por muestras: su parte del tiempo activo, una barra con su reparto por colores (adónde fue su tiempo), cuántas sesiones la ejecutaron y desde qué aplicaciones, la ejecución individual más larga vista, su evento de espera más frecuente y - cuando `pg_stat_statements` está en el servidor - sus llamadas, tiempo medio y total desde el reinicio de los contadores. Los botones son los mismos que en los hallazgos; una sentencia cortada en `track_activity_query_size` no tiene botón Plan, porque el planificador solo informaría del corte.

**Parejas de bloqueo** nombran ambos lados: el que espera y el bloqueador con sus aplicaciones, qué hacía el bloqueador mientras bloqueaba (`idle-txn` es el que hay que mirar), cuándo se vio la pareja por primera y última vez, el bloqueo que buscaba el que espera y ambas sentencias completas.

**Ajustes para un mejor análisis** cierra el informe (y la acción status también lo muestra, antes de tomar una sola muestra): los ajustes del servidor que deciden lo que las muestras pueden decir, cada uno con su valor actual, el recomendado, si requiere recarga, reinicio o CREATE EXTENSION, y por qué. Las advertencias son ajustes sin los cuales este informe es ciego a algo - `compute_query_id` (sin identidad de sentencia en las muestras), `track_activity_query_size` (sentencias cortadas), `pg_stat_statements` (sin contadores por sentencia), `track_io_timing` (tiempo de disco invisible), `log_lock_waits` (sin registro de esperas más cortas que una muestra); el resto son deseables (`auto_explain`, la única forma de ver el plan que *realmente corrió*, `log_min_duration_statement`, `log_temp_files`, `track_functions`). **Aplicar** ejecuta `ALTER SYSTEM SET` y una recarga tras la confirmación habitual; un ajuste que requiere reinicio se marca así y espera al siguiente. Sin superusuario las sentencias están ahí para copiar. También lista todo lo que necesita el plan que realmente se ejecutó - auto_explain y cómo registra, el recolector y el destino del registro, el privilegio para leer el registro (un GRANT, aplicado aquí cuando este inicio de sesión puede concederlo) y pg_store_plans (instalar un paquete se muestra como un comando para copiar) - cada uno con su coste; vea [El plan que realmente se ejecutó](#the-plan-that-actually-ran).

**Antes de que ASH o AWR empiecen a recoger datos** (ASH start, AWR take y auto-start) la aplicación pide al servidor la misma lista. Si falta algo que leen, primero aparece una ventana: las extensiones (`pg_stat_statements` - ¿creada en esta base de datos? ¿en `shared_preload_libraries`? - y `auto_explain`), y después cada ajuste que no está en su sitio, con su valor actual, el recomendado, si requiere reload, reinicio o CREATE EXTENSION, y por qué. Lo que deja ciego al informe viene marcado; lo deseable lo marca usted. **Activar lo seleccionado** ejecuta exactamente las sentencias mostradas bajo la lista (dos filas que añaden a `shared_preload_libraries` se muestran como la única sentencia en la que terminan), vuelve a leer el servidor y dice qué espera ahora un reinicio - la aplicación nunca reinicia el servidor. Una fila que el clic dejó en su sitio sigue en la lista, con su resultado, hasta **Comprobar de nuevo**. **Extensiones…** cierra la ventana y abre el gestor de extensiones; **Empezar de todos modos** funciona con lo que hay. Cada pestaña pregunta una vez; cuando no falta nada ya no aparece sola, y **Comprobar ajustes...** junto a Start en el plan de ASH y de AWR la muestra en cualquier momento, con lo deseable (como los ajustes propios de `auto_explain`, que solo existen tras el reinicio que lo carga); su Start inicia el plan. Volver a ejecutar una pestaña (su flecha circular) vuelve a leer la vista de la pestaña; nunca inicia, detiene ni toma nada. Un usuario que no es superusuario recibe las sentencias para copiar. La ventana se desactiva en Ajustes (*Comprobar antes de iniciar ASH / AWR*) o con su propia casilla.

#### Búsqueda en el historial de consultas {#query-history-search}

`db-query-history` es una búsqueda sobre `pg_stat_statements`, cruzada con el muestreador ASH para obtener una hora. Sus parámetros se rellenan ANTES del check, en el mismo diálogo de parámetros que usa cualquier check parametrizado:

| Campo | Hace |
| --- | --- |
| Statement contains | Coincidencia de subcadena sin distinguir mayúsculas/minúsculas contra el texto normalizado de la sentencia (los literales se eliminan - un nombre de tabla o columna suele ser lo más útil de escribir). Vacío coincide con todo. |
| Run by | Solo sentencias ejecutadas por este rol. |
| Worst by | Por qué ranking ordenar - tiempo total (en qué gasta su vida la base de datos), tiempo medio (qué es lento por ejecución), peor ejecución individual, número de llamadas, o filas devueltas. El tiempo total y el tiempo medio responden preguntas distintas: una consulta que tarda 2 ms un millón de veces supera a una que tarda un minuto dos veces. |
| Ignore faster than (ms) | Descarta filas cuyo tiempo medio esté por debajo de esto, para que una página de ruido rápido no entierre esa única forma lenta que merece la pena leer. |
| How many | Límite de filas, 1-200. |
| Include plans | EXPLAIN del texto normalizado para las 5 filas superiores - el plan de la forma, no un registro del plan realmente usado (ver la advertencia abajo). |

**Esta búsqueda ya se ejecuta contra todo el historial del lado del servidor cada vez que abres el diálogo y pulsas Run check** - no solo lo que está en pantalla en ese momento. La tabla de resultados recibe después el mismo cuadro de filtro por columna que recibe cualquier resultado de check (escribir en el encabezado de una columna acota las filas ya mostradas); ese filtro solo acota lo que esta búsqueda ya devolvió, no vuelve a buscar en todo el conjunto. Para buscar más atrás o por otro criterio, cambia los campos de arriba y ejecuta el check de nuevo.

Columnas del resultado:

| Columna | Qué es | Cómo leerla |
| --- | --- | --- |
| `user` | Rol que ejecutó la forma de sentencia | - |
| `calls` | Ejecuciones desde el último reinicio de los contadores | - |
| `total` / `mean` / `max` | Tiempo de ejecución acumulado / medio / peor individual | Misma lectura que `total_ms`/`mean_ms` en el informe de carga, arriba. |
| `rows` | Filas totales devueltas o afectadas en todas las llamadas | - |
| `ash_samples` | Cuántas muestras ASH capturaron el `queryid` de esta sentencia en ejecución | 0 significa que el muestreador ASH no estaba en marcha mientras esto se ejecutaba, no que nunca se ejecutó - `calls` es la cifra que de verdad importa. Necesita `db-ash` iniciado antes; PG13 no tiene `queryid` en ASH en absoluto. |
| `last_seen` | Marca de tiempo de la muestra ASH más reciente que la capturó | Vacío por la misma razón que `ash_samples` = 0. |
| `query` | El texto normalizado de la sentencia, truncado | Los literales se eliminan a marcadores (`$1`, `$2`, ...) - una fila cubre cada ejecución que solo difería en sus valores. |

Los contadores se reinician con `pg_stat_statements_reset()` o un fallo del servidor; un reinicio limpio del servidor los conserva cuando `pg_stat_statements.save` está activo (el valor por defecto de PostgreSQL). Una sentencia de antes del último reinicio simplemente no está - "nada coincidió" puede significar eso con la misma facilidad que un patrón equivocado.

#### Cómo leer un informe AWR / de carga {#reading-an-awr-report}

Cada número de estos informes es un **delta sobre la ventana** - el valor entre los dos snapshots, no una tasa ni un total desde el arranque. Un informe guardado lleva este glosario consigo (la exportación a PDF lo incluye), así que un informe reabierto semanas después sigue explicándose solo.

**Resumen de la ventana**

| Métrica | Qué es | Cómo leerla |
| --- | --- | --- |
| `window_seconds` / `window_hours` | Tiempo real entre los dos snapshots | Todo lo demás es "por esta cantidad de tiempo". Una ventana de 30 segundos y una de 6 horas no son comparables. |
| `txn_commit` / `txn_rollback` | Transacciones confirmadas / revertidas en la ventana | Recuentos, no tasas. Divide entre `window_seconds` para TPS. |
| `rollback_pct` | `txn_rollback / (txn_commit + txn_rollback)` | Por encima de ~10% con tráfico real significa que la aplicación lanza errores o entra en deadlocks - mira el vigilante de errores y el recuento de deadlocks. Unos pocos rollbacks en una ventana tranquila son ruido. |
| `cache_hit_pct` | `blks_hit / (blks_hit + blks_read)` - fracción de lecturas de bloque servidas desde **shared_buffers**, no desde disco | Es la caché de búferes de PostgreSQL, no la caché de páginas del SO, así que un número "bajo" puede seguir siendo rápido si el SO la tenía en caché. Por debajo de ~95% con tráfico apreciable suele significar que `shared_buffers` es demasiado pequeño para el working set, o que un escaneo secuencial grande vació la caché durante la ventana. |
| `deadlocks` | Deadlocks detectados en la ventana | Cualquier valor por encima de 0 es un fallo de orden de bloqueo en la aplicación. PostgreSQL lo resolvió matando una transacción; el par está en el log del servidor. |
| `temp_files` / `temp_bytes` | Ficheros (y tamaño total) que PostgreSQL volcó a disco para ordenaciones y hashes | Distinto de cero significa que la ordenación o el hash de una consulta no cupo en `work_mem`. Cruza con Top SQL por escrituras temp. |
| `new_sessions` / `sessions_per_s` | Sesiones iniciadas en la ventana (`pg_stat_database.sessions`, PostgreSQL 14+), y por segundo | Muchas sesiones cortas son rotación de conexiones: una aplicación sin pool de conexiones, o un pool que abre y cierra conexiones. Antes de PostgreSQL 14 el resumen dice que no se puede medir. |
| `mean_session` | Tiempo de sesión acumulado en la ventana dividido entre las sesiones iniciadas | Las conexiones de larga vida (del pool) también suman su tiempo, así que las sesiones cortas parecen más largas de lo que son: un número pequeño es una señal fuerte de rotación, uno grande no prueba nada. |

**Top SQL** (cuatro clasificaciones de las mismas filas: por tiempo total, por número de llamadas, por lecturas de disco, por escrituras temp)

| Columna | Qué es | Cómo leerla |
| --- | --- | --- |
| `calls` | Ejecuciones de esta sentencia normalizada en la ventana | - |
| `total_ms` | Suma del tiempo de ejecución sobre todas las `calls` (el delta de `total_exec_time`) | Es la clasificación que más importa: la consulta con el `total_ms` más alto consumió más tiempo de servidor en la ventana, sea o no lenta individualmente. |
| `mean_ms` | `total_ms / calls` - tiempo medio de una ejecución | **Un `mean_ms` grande** (digamos > 100 ms) significa que cada ejecución es lenta: falta un índice, mal plan, esperas de bloqueo, o una consulta realmente grande - hazle EXPLAIN (ANALYZE, BUFFERS). **Un `mean_ms` pequeño con un `total_ms` grande** significa que la consulta es barata por llamada pero se ejecuta tan a menudo que domina - la solución es llamarla menos (cachear el resultado, agrupar el trabajo) o rebajar el coste por llamada, no ajustar el plan. |
| `rows` | Total de filas devueltas o afectadas sobre todas las llamadas | `rows / calls` es el tamaño medio del resultado; una consulta de informe que trae 100k filas por llamada es un problema distinto de una consulta OLTP que trae 3. |
| `shared_read` | Bloques de 8 KB leídos **de disco** (no de caché) sobre todas las llamadas | Alto significa que los datos de esta consulta no están en `shared_buffers` - caché fría, o un working set mayor que la caché. `shared_read x 8192` es el volumen en bytes. |
| `temp_written` | Bloques de 8 KB escritos en **ficheros temp** sobre todas las llamadas | Distinto de cero significa que la ordenación o el hash se volcó. Sube `work_mem` para la sesión que la ejecuta, o corrige el plan para que ordene menos filas. `temp_written x 8192` es el volumen en bytes. |
| `qid` | El `queryid` (huella de la sentencia normalizada) | Mismo valor entre snapshots y en `pg_stat_statements`; úsalo para seguir una sentencia en el tiempo. |

**Top tablas por DML**

| Columna | Qué es | Cómo leerla |
| --- | --- | --- |
| `ins` / `upd` / `del` | Filas insertadas / actualizadas / borradas en la ventana | Impulsa el autovacuum. Una tabla con mucho `upd` + `del` y sin autovacuum reciente es candidata a bloat - ver [Vacuum, bloat y estadísticas de tablas](#vacuum-bloat-and-table-statistics). |
| `hot_upd_%` | Actualizaciones HOT como fracción de todas las actualizaciones | Las actualizaciones HOT no tocan índices y no necesitan limpieza de índice. **Bajo** (< ~50%) en una tabla muy actualizada significa que las actualizaciones cambian columnas indexadas, o que las páginas no tienen sitio libre - baja el `fillfactor` de la tabla para que las actualizaciones se queden en la página. |
| `seq_scan` / `idx_scan` | Escaneos secuenciales vs de índice iniciados sobre la tabla en la ventana | Un delta de `seq_scan` grande en una tabla grande es señal de índice faltante - confírmalo con el asesor de índices. |

**Actividad de fondo / checkpoint**

| Columna | Qué es | Cómo leerla |
| --- | --- | --- |
| `ckpt_timed` / `ckpt_req` | Checkpoints disparados por horario vs por volumen de WAL | Mayormente `req` significa que los checkpoints se fuerzan por volumen de escritura antes de `checkpoint_timeout` - sube `max_wal_size` para que se repartan. |
| `ckpt_write_ms` / `ckpt_sync_ms` | Tiempo escribiendo / haciendo fsync de búferes sucios durante los checkpoints | Un `sync_ms` grande significa que el almacenamiento no puede absorber la ráfaga de escritura del checkpoint. |
| `buffers_backend` | Búferes sucios que los **backends** tuvieron que escribir ellos mismos | Valores no triviales significan que el background writer y el checkpointer se quedaron atrás y los backends de consulta pagaron el coste de escritura en línea. |

**Hallazgos** es la propia lectura que hace el informe de la ventana, lo peor primero, dibujada como los hallazgos de ASH: cada uno con la sentencia detrás y sus botones. Más allá de los umbrales anteriores (deadlocks, parte de rollbacks, aciertos de caché) nombra una sentencia que se llevó el 30% o más de todo el tiempo de ejecución, sentencias con una media de más de 500 ms por llamada, sentencias que volcaron 64 MB o más a archivos temporales (`work_mem`), checkpoints forzados por volumen de WAL más a menudo que por el reloj (`max_wal_size`), backends que escriben ellos mismos una quinta parte o más de los búferes sucios (`bgwriter_*`) y una tabla muy actualizada con menos de la mitad de actualizaciones HOT (`fillfactor`). Todo umbral de sentencias de aquí exige que la ventana tenga al menos diez segundos de tiempo de ejecución en total, para que una base de pruebas tranquila no parezca un servidor en apuros. Cuatro hallazgos se juzgan en cambio por su propio volumen - una carga N+1 pura, una búsqueda rápida ejecutada cientos de miles de veces sin nada lento al lado, nunca llega a diez segundos y antes se leía "Nada destacó en esta ventana":

- **Patrón 1+N**: una sentencia (B) se ejecuta k veces por cada ejecución de otra (A), en la misma base de datos con el mismo usuario, con k al menos 5 y B ejecutada al menos 1.000 veces. La proporción sola podría ser casualidad, así que además necesita una prueba: A devolvió (con un 5% de margen) tantas filas como veces se ejecutó B - una por fila; o la proporción se mantuvo dentro de un 10% en al menos tres intervalos entre instantáneas mientras el ritmo propio de A cambiaba la mitad o más (un intervalo en el que A empezó o paró no cuenta, y con una carga plana dos sentencias cualesquiera mantienen su proporción, lo que no prueba nada); o las llamadas de B son un múltiplo exacto de las de A con un margen de dos llamadas, habiéndose ejecutado A al menos 500 veces. Una comparación guardada lee las instantáneas tomadas entre las dos que elegiste (hasta 24); un informe de carga cuenta las sentencias una docena de veces mientras graba. Ámbar cuando B se ejecuta 200 veces por segundo o se lleva el 10% del tiempo de ejecución, azul por debajo. Las sentencias ejecutadas dentro de una función se dejan fuera - giran en el servidor, sin viaje de ida y vuelta que ahorrar. El hallazgo dice lo que pg_stat_statements no puede: cada llamada además le cuesta a la aplicación un viaje de ida y vuelta, normalmente más que la propia consulta.
- **Una sentencia de menos de un milisegundo ejecutada muy a menudo**: al menos 10.000 llamadas de menos de 1 ms cada una Y el 10% de todo el tiempo de ejecución (la proporción) o 200 llamadas por segundo (el ritmo), lo que llegue antes; una sentencia ya nombrada en un hallazgo 1+N no se repite.
- **Rotación de conexiones** (PostgreSQL 14+): ámbar a partir de 20 sesiones nuevas por segundo, azul a partir de una sesión nueva por cada dos transacciones, ambos con al menos 200 sesiones nuevas en la ventana. Nombra las bases de datos y añade las sesiones que el cliente abandonó sin cerrar, que acabaron en un error fatal o que fueron terminadas.
- **Bloqueo optimista**: un UPDATE o DELETE cuyo WHERE compara una columna de versión con un parámetro y que en algunas de sus llamadas no escribió ninguna fila - esas perdieron la carrera contra otro escritor y se reintentaron. Ámbar cuando al menos el 5% de al menos 100 llamadas, o 1.000 llamadas, no escribieron nada; consulta [Conexiones, sesiones y carga](#connections-sessions-and-load).

**Top SQL a revisar** son tarjetas de las sentencias que nombraron los hallazgos, ordenadas por su parte de todo el tiempo de ejecución, cada una con las razones como etiquetas y los mismos botones: copiar, cargar en el editor SQL, EXPLAIN allí, o **Plan** - el plan dibujado en el sitio. Muestra el plan que realmente se ejecutó, de `auto_explain` en el registro del servidor o de `pg_store_plans`, y el plan que el servidor elegiría *ahora* solo cuando ninguno lo tiene, marcado como tal - vea [El plan que realmente se ejecutó](#the-plan-that-actually-ran).

**Ajustes para un mejor análisis** cierra una comparación, una toma y una captura en vivo: la misma tabla con la que termina el informe ASH - ver [Cómo leer un informe ASH](#reading-an-ash-report). Se lee del servidor al que estás conectado, así que cada clúster recibe sus propias recomendaciones para su versión: `compute_query_id` y `track_wal_io_timing` desde PostgreSQL 14, `pg_stat_statements.track_planning` desde 13, y no se propone nada que la versión del servidor no tenga.

**Lo que solo tiene el informe de carga**, a partir de sus propias muestras de pg_stat_activity (todo el servidor, cada pocos segundos):

- **Sesiones activas a lo largo del tiempo** es un gráfico con un punto por muestra: todas las sesiones activas y, de ellas, cuántas se ejecutaban, esperaban un bloqueo, esperaban el disco o esperaban otra cosa, junto a las sesiones inactivas dentro de una transacción (retienen sus bloqueos). Los colores son los de la línea de tiempo de ASH, así que un bloqueo es rojo en ambos. Una ventana larga se dibuja como líneas sin puntos; la exportación escribe cada muestra como una fila, y el PDF las promedia en como máximo 60 filas, cada una con su intervalo de tiempo.
- **Eventos de espera** muestran dónde esperaban los backends muestreados: `Lock` (bloqueado por otra transacción), `IO` (esperando disco), `LWLock` (contención interna), `Client` (esperando al cliente - normalmente solo conexiones inactivas, no es un problema). Una ventana dominada por esperas `Client` es un sistema inactivo sano.
- **Pares de bloqueo observados** lista quién esperó a quién, en cuántas muestras, con ambas sentencias; **Transacción abierta más larga vista** es la transacción más antigua que captó alguna muestra.
- Sus hallazgos se unen a los demás: sesiones que esperaron el bloqueo de otra sesión (se muestra la sentencia que lo retenía, la que esperaba al lado); una quinta parte o más del tiempo activo esperando bloqueos Y al menos 5 muestras así; sesiones inactivas dentro de una transacción en una quinta parte o más de las muestras ocupadas Y al menos 10 de ellas; una transacción abierta un minuto o más.

#### Cómo leer una comparación de periodos {#reading-a-period-comparison}

Una comparación de periodos (`db-awr-snapshot` action=compare-periods, la vista **Comparar** de la pestaña AWR) responde a "qué cambió entre entonces y ahora" en lugar de "qué pasó en esta ventana". Cada número sigue siendo una diferencia sobre su propia ventana; el informe pone las dos ventanas una al lado de la otra.

- **Periodos comparados**: una fila por métrica - duración de la ventana, transacciones confirmadas en total y por segundo, porcentaje de rollbacks, acierto de caché, bloques leídos de disco, bytes temporales escritos, interbloqueos, checkpoints programados y solicitados, búferes escritos por los backends, bytes de WAL, el tiempo de ejecución de todas las sentencias y cuántas sentencias se ejecutaron. **Cambio** es el periodo 2 menos el periodo 1, con una flecha hacia arriba o hacia abajo, y **cambio %** es relativo al periodo 1 (para un porcentaje el cambio va en puntos). Rojo es un cambio a peor, verde a mejor; una métrica en la que subir no es ni lo uno ni lo otro (más transacciones es más trabajo, no un servidor peor) no se colorea. Cuando la comparación es de una base de datos, las últimas filas son las de todo el servidor.
- Los recuentos crecen con la ventana: si los periodos tienen distinta duración, compare las tasas por segundo y los porcentajes - un hallazgo lo dice.
- **Hallazgos** citan los números de ambos periodos: una sentencia cuyo tiempo por llamada subió la mitad o más y al menos 10 ms, cuando eso costó al periodo 2 al menos un segundo; una sentencia que no se ejecutó en el periodo 1 y ocupó una quinta parte o más del tiempo de ejecución del periodo 2 (y al menos un segundo); el acierto de caché dos puntos o más abajo; archivos temporales al menos duplicados y de 64 MB o más; checkpoints forzados por volumen de WAL más a menudo que por el reloj solo en el periodo 2; la proporción de rollbacks cinco puntos o más arriba con al menos 50 transacciones; interbloqueos que aparecieron. Una sentencia principal que se volvió al menos el doble de rápida por llamada, ahorrando al menos un segundo, recibe una línea verde. Un reinicio, un restablecimiento o una expulsión dentro de cualquiera de los periodos se dice primero, con el periodo en que ocurrió.
- **Sentencias: periodo 1 frente a periodo 2** lista las 30 sentencias cuyo tiempo total más cambió, cada una con las llamadas, el tiempo total y medio, las filas y las lecturas de disco de ambos periodos, su estado - *ambos*, *solo periodo 1* (desaparecida) o *solo periodo 2* (nueva) - y los botones que tiene Top SQL (copiar, el editor SQL, EXPLAIN, **Plan**).
- **Tablas por DML: periodo 1 frente a periodo 2** lista las 20 tablas de la base de datos conectada cuyas inserciones, actualizaciones y borrados más cambiaron, con los recorridos secuenciales y por índice de cada periodo.

#### El plan que realmente se ejecutó {#the-plan-that-actually-ran}

**Plan** en una sentencia del informe ASH, en los hallazgos y las tarjetas **Top SQL a revisar** de los informes AWR y de carga, y en una comparación de periodos muestra el plan con el que la sentencia *realmente se ejecutó* siempre que el servidor lo guardó, y dice claramente cuándo no.

**Por qué EXPLAIN ahora no basta.** Un EXPLAIN hecho después vuelve a planificar la sentencia, hoy: con las estadísticas de tablas de hoy, los ajustes e índices de hoy y - para una sentencia con parámetros `$1` - un plan genérico en lugar del que obtuvieron los valores reales. El plan que hizo lenta la sentencia en la ventana puede ser otro, y EXPLAIN no puede decirlo.

**Dos fuentes guardan el plan real:**

- **auto_explain** (un módulo contrib que viene con PostgreSQL) escribe en el registro del servidor el plan de cada sentencia más lenta que `auto_explain.log_min_duration`, con el número real de filas cuando `auto_explain.log_analyze` está activo. DBApguin lo vuelve a leer por la misma conexión a la base de datos, solo cuando abre un plan, y solo una parte acotada del registro: los archivos de registro que se solapan con la ventana del informe, leídos desde el final, como mucho 32 MB y 8 segundos por consulta. Un plan registrado se asocia a la sentencia por su identificador de consulta donde el registro lo tiene, y si no, por el texto de la sentencia sin sus constantes.
- **pg_store_plans** (de terceros, licencia BSD) guarda cada plan de cada sentencia en memoria compartida con las llamadas, el tiempo total y la primera y última llamada de cada plan, bajo el mismo query id que usa `pg_stat_statements`. No se lee nada de un registro.

**Qué muestra la vista del plan.** Tres botones arriba - **auto_explain (registro del servidor)**, **pg_store_plans** y **EXPLAIN ahora** - cada uno con el número de planes que encontró; se abre en el primero que tiene alguno, en ese orden. Una etiqueta dice qué tipo de plan está viendo: *El plan que se ejecutó* en verde, o *El plan que el servidor elegiría ahora, no el que se ejecutó* en amarillo, con una línea sobre qué activar. Cuando una sentencia se ejecutó en la ventana con más de un plan (el plan cambió), los planes se listan del más reciente al más antiguo, cada uno con cuándo se registró por última vez (según el reloj del servidor, como el Desde y el Hasta del informe), su duración, cuántas ejecuciones se registraron con él y su tiempo más corto, medio y más largo; un clic lo dibuja. Bajo el árbol: si el plan se asoció por query id o por texto, si lleva filas y tiempos reales, los valores de los parámetros cuando el servidor los registró y cuánto del registro se leyó. Una fuente sin resultado dice por qué (no cargado, sin recolector de registros, sin privilegio para leer el registro, nada más lento que el umbral en la ventana, una plataforma gestionada). EXPLAIN ahora sigue funcionando en todas partes y solo se ejecuta cuando lo elige o cuando no hay nada mejor.

**Configuración, paso a paso.** *Ajustes para un mejor análisis* al final de cada informe ASH, AWR, de carga y de traza, y la ventana antes de iniciar ASH, AWR o una traza, listan cada pieza con su valor actual, el recomendado y cómo se aplica. Lo que DBApguin puede hacer por sí mismo tiene **Aplicar** (o se marca en la ventana), con la confirmación habitual y una entrada en el registro de auditoría; lo que necesita el sistema operativo muestra el comando para copiar.

1. **Cargar auto_explain** - en DBApguin: Aplicar añade `auto_explain` a `shared_preload_libraries`. En un terminal: reinicie el servidor (los comandos para apt.postgresql.org, RPM de PGDG, `pg_ctl` y un contenedor se muestran bajo la tabla; un reinicio termina todas las sesiones).
2. **Registrar planes** - en DBApguin: `auto_explain.log_min_duration` (por ejemplo `1s`), `auto_explain.log_format = json` (la vista dibuja el JSON como árbol) y en PostgreSQL 16+ `auto_explain.log_verbose = on` (el identificador de consulta dentro del plan). Esto requiere una recarga, no un reinicio.
3. **Escribir el registro donde SQL pueda leerlo** - en DBApguin: `logging_collector = on` (reinicio) y `log_destination` con `jsonlog` añadido en PostgreSQL 15+, `csvlog` antes. stderr también se lee, tan bien como permite su `log_line_prefix`.
4. **Identidad de la sentencia** - en DBApguin: `compute_query_id = on` (PostgreSQL 14+). Opcional y recomendado: `track_io_timing = on`.
5. **El privilegio para leer el registro** - un superusuario lo lee tal cual. Para un inicio de sesión de monitorización lo mínimo que funciona es ser miembro de `pg_monitor` (lista el directorio del registro) y `EXECUTE` sobre `pg_read_file(text, bigint, bigint, boolean)`, que solo lee dentro del directorio de datos y del directorio del registro. DBApguin muestra los dos GRANT y ofrece Aplicar cuando el inicio de sesión conectado puede concederlos (se le pregunta al servidor, no se adivina por el nombre de un rol); si no, los ejecuta un superusuario. Un inicio de sesión que puede concederlos - normalmente un superusuario - recibe además **Conceder a un login...**: lista los inicios de sesión que aún no pueden leer el registro (no los superusuarios, que no necesitan nada), muestra las dos sentencias para el elegido y las ejecuta, con un registro en el log de auditoría. `pg_read_server_files` o un superusuario leerían cualquier archivo que pueda leer el usuario del sistema del servidor - mucho más de lo que necesita un plan.
6. **O pg_store_plans** - en un terminal: instale el paquete (`dnf install pg_store_plans_NN` del repositorio PGDG en RHEL, Rocky y Fedora; para Debian y Ubuntu no hay paquete apt, así que compílelo con `make USE_PGXS=1` y `make USE_PGXS=1 install`). En DBApguin: Aplicar lo añade a `shared_preload_libraries`, un reinicio lo carga y después **Instalar…** ejecuta `CREATE EXTENSION pg_store_plans` en cada base de datos donde quiera leerlo.

**Qué existe en qué versión** (comprobado con la documentación de PostgreSQL de cada versión y en servidores de 9.4 a 19):

| Nombre | PostgreSQL | Notas |
| --- | --- | --- |
| `auto_explain` en `shared_preload_libraries` | todas | reinicio para cargarlo |
| `auto_explain.log_min_duration`, `log_analyze`, `log_timing`, `log_buffers`, `log_verbose`, `log_format`, `log_triggers`, `log_nested_statements` | todas | recarga; los cambia un superusuario |
| `auto_explain.sample_rate` | 9.6+ | |
| `auto_explain.log_level`, `auto_explain.log_settings` | 12+ | |
| `auto_explain.log_wal` | 13+ | |
| `auto_explain.log_parameter_max_length` | 16+ | valores de los parámetros de sentencias preparadas, completos por defecto (-1) |
| `auto_explain.log_io`, `auto_explain.log_extension_options` | 19 | |
| `compute_query_id` | 14+ | antes de 14 un plan se asocia por su texto |
| Identificador de consulta escrito en el plan registrado (`log_verbose` respeta `compute_query_id`) | 16+ | en 14 y 15 el identificador solo está en la columna `query_id` de csvlog / jsonlog |
| `logging_collector` | todas | reinicio |
| `log_destination = csvlog` | todas | columna `query_id` desde 14 |
| `log_destination = jsonlog` | 15+ | una entrada por línea, con `query_id` |
| `pg_current_logfile()`, `pg_ls_logdir()` | 10+ | `pg_ls_logdir()`: superusuario y `pg_monitor` |
| rol `pg_monitor` | 10+ | |
| `pg_read_file(text, bigint, bigint, boolean)` | 9.5+ | `GRANT EXECUTE` posible desde 11; antes solo superusuario |
| rol `pg_read_server_files` | 11+ | más de lo que necesita un plan |
| `pg_store_plans` | de terceros | probado con 1.10 en 17 y 18, donde su query id es el de `pg_stat_statements` |

Antes de PostgreSQL 10 no hay `pg_ls_logdir()`, así que allí el registro no se lee por SQL; la vista lo dice y ofrece pg_store_plans o EXPLAIN ahora. PostgreSQL 13 es la versión más antigua que DBApguin soporta por completo.

**Qué cuesta** - dicho de nuevo junto a cada ajuste, donde lo activa:

- **Un reinicio** para `shared_preload_libraries` (auto_explain, pg_store_plans) y para `logging_collector`. DBApguin nunca reinicia un servidor.
- **`auto_explain.log_analyze` instrumenta cada sentencia**, se registre o no, lo que puede ralentizar de forma notable un servidor ocupado. Mida antes el reloj con `pg_test_timing`; `auto_explain.log_timing = off` conserva el número real de filas (lo que revela una mala estimación) y elimina la mayor parte del coste; `auto_explain.sample_rate` por debajo de 1 instrumenta solo una parte de las sentencias, al precio de perder algunas lentas.
- **Volumen del registro.** Cada plan ocupa muchas líneas; un `log_min_duration` bajo en un servidor ocupado llena pronto el registro. Planifique la rotación (`log_rotation_age`, `log_rotation_size`) y el disco. jsonlog o csvlog es un segundo archivo junto a stderr.
- **El registro es sensible.** Contiene el texto de las sentencias y, desde PostgreSQL 16, los valores de los parámetros (`auto_explain.log_parameter_max_length = 0` los omite). Quien puede leer el registro lee también eso.
- **Leer el registro requiere privilegios de lectura de archivos.** `EXECUTE` sobre `pg_read_file` lee cualquier archivo del directorio de datos y del directorio del registro saltándose todos los privilegios de la base de datos; concédalo solo a un inicio de sesión de monitorización.
- **pg_store_plans** ocupa memoria compartida (`pg_store_plans.max` planes) y trabaja en cada sentencia; la mayoría de las nubes gestionadas no lo ofrecen.
- **Las nubes gestionadas** (RDS, Aurora, Cloud SQL, Azure y otras) no dejan que SQL lea el registro del servidor: el registro está en la consola del proveedor, y la vista lo dice y pasa a pg_store_plans donde se ofrece, y después a EXPLAIN ahora.

DBApguin lee el registro solo por su conexión, guarda lo leído en memoria mientras el plan está abierto y no envía nada a ninguna parte - vea el [aviso de privacidad](PRIVACY.md).

#### Alcance: su base de datos, con el servidor al lado {#awr-ash-scope}

ASH y AWR recogen datos de todo el servidor, como los de Oracle de la instancia: el muestreador lee cada sesión de cada base de datos, y cada instantánea guarda `pg_stat_statements` y `pg_stat_database` de todas. Ambos siguen en segundo plano cuando cambia a otra conexión o pestaña; la lista Background jobs los muestra.

Un informe dice primero de dónde vienen sus datos (**Collected for**: el servidor y la base de datos desde la que se iniciaron el muestreador o las instantáneas) y luego de qué trata. Por defecto es esa misma base de datos: Top SQL, esperas, hallazgos y tarjetas son suyos, y al lado está todo el servidor - una segunda fila en el **Window summary** de AWR, **Activity by database** para cada base de datos y un hallazgo **Elsewhere on the server** cuando otra base de datos hizo algo digno de mención (deadlocks, 50 o más rollbacks, una quinta parte del tiempo de ejecución Y al menos 10 s; en ASH una quinta parte de las muestras activas Y al menos 60, o 10 o más muestras esperando bloqueos). `*` en el campo **Database** muestra todo el servidor, y el nombre de otra base de datos muestra esa con el mismo resumen. La elección se hace al leer, no al capturar, así que las mismas instantáneas y muestras sirven para las tres. **Top tables by DML** es de la base de datos conectada (PostgreSQL lleva esos contadores por base de datos) y aparece solo cuando el informe trata de ella. **Plan** pide el plan en la base de datos donde se ejecutó la sentencia. Las muestras e instantáneas anteriores al 22-23 de octubre de 2026 no registran la base de datos por sentencia; para ellas el informe muestra todo el servidor y lo dice.

**pg_stat_statements was full and evicted statements** en una comparación significa que al Top SQL de AWR le falta lo que costaron esas sentencias: `pg_stat_statements` guarda `pg_stat_statements.max` sentencias y descarta el 5% menos usado (al menos 10) cada vez que se llena. PostgreSQL 14 y posteriores cuentan esas rondas; el hallazgo las da, con una estimación de las sentencias perdidas y lo lleno que está. Aumente `pg_stat_statements.max` (reinicio) - la tabla de ajustes del final tiene la fila. Una sentencia cuyos contadores retrocedieron (expulsada y de vuelta, o un reinicio) cuenta desde su regreso.

#### Tendencia: cada instantánea representada a lo largo del tiempo {#awr-trend}

`db-awr-snapshot` action=trend convierte todo el historial de instantáneas de un servidor en gráficos, en lugar de la vista de un par a la vez que da `compare`. Necesita al menos dos instantáneas - tómalas manualmente, o deja que el auto-scheduler construya historial en segundo plano.

En estos gráficos aparecen dos tipos distintos de números, y se dibujan de forma diferente a propósito:

- **Valores puntuales** - tamaño de la base de datos, bloat de heap estimado - son exactamente lo que se midió en el momento de esa instantánea. Un punto por instantánea, representado tal cual.
- **Tasas** - cache-hit%, rollback%, consultas principales por tiempo total - provienen de contadores de `pg_stat_database` y `pg_stat_statements`, que solo suben desde el último reinicio. El contador bruto de una sola instantánea no dice nada por sí solo; lo que se grafica es la **diferencia entre dos instantáneas consecutivas**, convertida en una tasa para esa ventana. Es el mismo cálculo que usa el `compare` de dos instantáneas, solo que repetido para cada par consecutivo del historial en vez de un único par elegido por el usuario.

Las **regresiones** se destacan encima de los gráficos, no se dejan para que el lector las note en una línea: una métrica se marca solo cuando se movió tanto en una cantidad relativa (un porcentaje) como en una cantidad absoluta (bytes, o puntos porcentuales) - la misma regla que usa cualquier umbral de esta aplicación, de modo que una base de datos pequeña que duplica su tamaño no supera a una grande que crece mucho más en cifras absolutas, y viceversa. El punto de comparación es aproximadamente una semana antes de la instantánea más reciente, o el punto disponible más antiguo cuando el historial es más corto que eso.

**Los huecos se muestran, nunca se suavizan.** Cuando el intervalo entre dos instantáneas es mucho más ancho que el intervalo habitual del servidor (2.5 veces la mediana, con un suelo de 3 horas para que dos instantáneas manuales tomadas con una hora de diferencia nunca se marquen entre sí), el gráfico rompe la línea ahí en vez de dibujar un borde recto sobre un tiempo del que en realidad no se capturó nada. Una banda sombreada marca exactamente dónde está el tramo que falta; pasar el cursor por encima dice cuán ancho es.

El gráfico de **consultas principales** sigue al puñado de `queryid`s que más tiempo total costaron sumado a lo largo de todo el historial, cada una como su propia línea de tiempo consumido por ventana - no el total acumulado único, que solo crece y sería apenas una línea diagonal recta que no dice nada sobre *cuándo* se pagó el coste.

Una lista de **tablas más grandes** bajo el gráfico de tamaño nombra las tablas más grandes según la última instantánea - "tamaño de la base de datos" por sí solo no dice qué tabla creció realmente. No se rastrea por tabla a lo largo del tiempo (una tabla puede salir del top 10 y volver a entrar mientras otras crecen a su alrededor), solo la imagen actual, igual que `db-table-sizes`.

#### Informes guardados {#saved-reports}

Cada captura de carga, informe AWR (un informe de dos instantáneas, una comparación de periodos o una tendencia), informe ASH y traza de sesión se guarda cuando termina. *Ajuste y diagnóstico* > **Informes guardados** los lista de todos los servidores en un solo sitio, los más recientes primero: cuándo, el tipo, el servidor, la base de datos de la que trata (*todo el servidor* para un informe sobre todas) y qué cubre (la ventana, las dos instantáneas, el usuario trazado), con su tamaño. El enlace **Informes guardados** de las pestañas AWR, ASH y traza de sesión abre la misma lista filtrada por esa herramienta y ese servidor; desde el resultado de *Captura* de la pestaña AWR muestra las capturas de carga.

- **Filtros**: tipo, servidor, base de datos y un rango de fechas (el selector de fecha subraya los días que tienen un informe). **Quitar filtros** vuelve a mostrarlo todo.
- **Abrir** muestra el informe tal como se guardó, de solo lectura: no se consulta la base de datos y no están los botones que la cambiarían. Su pie tiene la fila de exportación que tenía el informe (texto, CSV, JSON, PDF), escrita con el servidor y la hora de ejecución del propio informe, no los de hoy.
- **Exportar** abre los mismos cuatro formatos en la fila, sin abrir el informe.
- **Eliminar** pregunta antes y dice qué se va: el informe con sus secciones y su texto, y su entrada en Trabajos en segundo plano; para una traza, también su entrada en el Historial de la traza. Una copia exportada antes no se ve afectada. El registro de auditoría anota la eliminación.

Los informes se guardan aparte de los demás resultados que conserva Trabajos en segundo plano: quedan los 200 más recientes, se ejecuten las comprobaciones que se ejecuten (el almacén de trazas conserva las 500 más recientes por servidor). La biblioteca no guarda nada propio, así que la exportación e importación de los datos de la aplicación la llevan consigo: los informes guardados viajan con el elemento de informes, las trazas con el historial AWR y ASH del servidor.

### 6.7 Exportación lógica de un subconjunto (una rebanada que carga sola) {#logical-subset-export-a-slice-that-loads-on-its-own}

Una copia de seguridad copia tablas enteras; esto copia una rebanada de una.
Nombras las filas de partida con una cláusula WHERE - un cliente, un inquilino,
los pedidos del mes pasado - y la exportación sigue las claves foráneas desde
ahí: los padres, porque una fila a la que le falta su cliente no cargará, y los
hijos, porque un pedido sin sus líneas no es el mismo pedido. Sale un único
archivo `.sql`, escrito primero los padres, que carga solo en una base vacía sin
un solo error de clave foránea.

El motivo habitual para quererlo: una base de pruebas que se comporta como
producción y cabe en un portátil - el inquilino 42 en vez de 400 GB.

1. Explorador de esquema -> clic derecho en una tabla -> Subconjunto.
2. Escribe una cláusula WHERE para la tabla de partida.
3. **Descubrir el cierre** recorre las claves foráneas sin materializar ninguna
   fila e informa de cuántas aportaría cada tabla. Ejecútalo primero: una
   cláusula WHERE que parece estrecha puede arrastrar casi toda la base a través
   de una sola tabla de unión ancha, y es mejor enterarse aquí que a mitad de la
   exportación.
4. La exportación escribe las filas en la ubicación que elijas - sentencias
   INSERT, o bloques COPY cuando importa más la velocidad de carga que poder
   editarlas.

La profundidad (cuántas claves foráneas sigue el recorrido) y un tope de filas
por tabla detienen ambos el recorrido. Alcanzar cualquiera de ellos se informa
en vez de aplicarse en silencio: un subconjunto truncado que parece completo es
peor que uno que dice que no lo está.

Las filas se copian tal cual, **datos personales incluidos**. El enmascarado
está en Copy table, más abajo.

### 6.8 Enmascarar datos personales en una copia (anonimización de PII) {#masking-personal-data-in-a-copy-pii-anonymisation}

Los datos personales - nombres, direcciones de correo, teléfonos, números de
identidad, fechas de nacimiento - son lo que impide entregar una copia de
producción a un desarrollador o a un proveedor. Explorador de esquema -> clic
derecho en una tabla -> **Copy table** -> PII anonymization sustituye esos
valores al leer las filas: la columna se cambia por una expresión dentro del
SELECT, de modo que el valor real nunca llega al destino ni se escribe en disco.
La tabla de origen no se modifica. Las columnas las nombras tú - no se detecta
nada automáticamente, y una columna que olvides se copia en claro.

| Regla | Escribe | Qué sigue funcionando |
| --- | --- | --- |
| `NULL` | nada | No hay nada que se escape. Falla en una columna NOT NULL. |
| `@HASH` | `sha256` del valor (`md5` antes de PostgreSQL 11) | Los valores iguales siguen siendo iguales, así que los joins y el GROUP BY siguen funcionando. Añade una sal (abajo) para que el hash no pueda emparejarse con producción. |
| `@HASH:<sal>` | como `@HASH`, con esta sal por columna | Para el caso raro en que una columna necesita su propia sal además de la de toda la copia. |
| `@SEQ` | `nombre_1`, `nombre_2`, ... | Los valores siguen siendo únicos. Cualquier coincidencia con otra tabla desaparece. |
| `@EMAIL` | `user1@example.invalid` | Sintaxis válida, y a `.invalid` no se puede entregar nada - ninguna prueba puede escribirle a un cliente. |
| `@PHONE` | `0000000000` | La longitud, así que las validaciones de formato siguen pasando. |
| `@DATE` | el primer día del mes | Los informes mensuales y las franjas de edad. El día exacto desaparece. |
| `@REDACT` | `REDACTED` | Solo que había un valor. |
| `@EXPR:<sql>` | lo que devuelva el SQL | Tu propia regla, p. ej. `left(nombre,1) || '***'`. No se comprueba - una expresión errónea hace fallar la copia. |
| cualquier otro texto | ese texto, en cada fila | Un marcador fijo. La columna deja de variar por completo. |

#### La sal

Un hash sin sal es un disfraz, no una prueba. Cuando el conjunto de entradas
posibles es pequeño y conocido - un número de identidad, un teléfono, un
código postal -, cualquiera con la misma función de hash puede hashear cada
posibilidad y emparejar los valores directamente. Una **sal** es una cadena
secreta mezclada con el valor antes de hashearlo, de modo que el hash no se
puede reproducir sin ella. El control **Sal para @HASH** ofrece:

- **ninguna** - el hash del valor solo (reidentificable como arriba).
- **aleatoria, solo esta copia** - se genera una sal aleatoria nueva para esta
  copia y se descarta después. Las entradas iguales siguen produciendo el
  mismo hash *dentro* de esta copia, así que los joins y el GROUP BY sobre
  columnas enmascaradas siguen funcionando, pero la copia no puede
  emparejarse con producción ni con otra copia.
- **fija** - tú indicas la sal. Úsala cuando hagas copias **incrementales**
  (una segunda copia debe coincidir con la primera) o cuando necesites
  **comparar dos copias** - ambas necesitan la misma sal. Guárdala a buen
  recaudo; quien la tenga puede deshacer la protección de la sal.

La sal se envía al servidor como un ajuste de transacción
(`set_config('dbapguin.anon_salt', ...)`) y se lee de vuelta mediante
`current_setting()` desde la expresión de enmascarado. **Nunca** se escribe
en el SQL generado, en el registro de auditoría de DBApguin ni en la tabla de
destino. Sí llega al servidor de origen como una llamada `set_config` - si ese
servidor registra cada sentencia (`log_statement = all`), la sal aparece ahí
una vez.

`@SEQ`, `@REDACT` o `NULL` siguen siendo la opción correcta cuando la copia no
debe poder reidentificarse en absoluto, con o sin sal. Y el enmascarado va
columna a columna: una fila sin nombre pero con la fecha de nacimiento y el
código postal intactos a menudo sigue señalando a una sola persona.

Una columna que no necesitas para nada es mejor quitarla del todo, en Data
filters -> Skip columns.

### 6.9 Restricciones - quitarlas y devolverlas {#constraints-take-them-off-put-them-back}

A veces las restricciones tienen que salir de una tabla y volver después. El
caso habitual es una herramienta de replicación o CDC haciendo su sincronización
inicial contra el destino: no arranca mientras allí se impongan claves foráneas.
Una carga masiva es el otro - va más rápida sin las comprobaciones y las
recupera al final.

Hecho a mano, este es el punto donde los datos dejan de ser consistentes en
silencio, porque hay tres cosas fáciles de equivocar y PostgreSQL solo te frena
en una:

- `DROP ... CASCADE` quita las claves foráneas que otras tablas apuntan a esta y
  no deja constancia de lo que quitó. Sus definiciones se van con ellas.
- Una definición reescrita de memoria pierde `ON DELETE CASCADE`, o `DEFERRABLE
  INITIALLY DEFERRED`, o un `WHERE` de un índice único parcial. Parece
  restaurada. Impone otra cosa.
- Devolver una clave foránea por la vía normal recorre toda la tabla bajo un
  bloqueo que detiene lecturas y escrituras.

Explorador de esquema -> clic derecho en una tabla -> **Restricciones - quitar y
restaurar**. Lee cada restricción de `pg_constraint` mediante
`pg_get_constraintdef`, que es la propia representación de PostgreSQL, y escribe
dos scripts: uno que las quita en un orden que funciona y otro que las devuelve
exactamente como estaban. La aplicación no ejecuta nada. Obtienes SQL y lo
ejecutas cuando lo decidas.

Tres opciones cambian el script de restauración:

- **Añadir NOT VALID y validar aparte.** `ADD CONSTRAINT ... NOT VALID` se salta
  el recorrido; `VALIDATE CONSTRAINT` lo hace después bajo `SHARE UPDATE
  EXCLUSIVE`, por el que lectores y escritores pasan de largo. Medido sobre una
  tabla de 200.000 filas: 139 ms de bloqueo se convirtieron en 1,2 ms más una
  pasada de 131 ms que nadie espera. Hasta que VALIDATE se ejecute, la
  restricción solo se aplica a las filas nuevas, y `pg_constraint.convalidated`
  lo dice.
- **Reconstruir en línea los índices de clave primaria y únicos.** `CREATE UNIQUE
  INDEX CONCURRENTLY` y luego `ADD CONSTRAINT ... USING INDEX`, que adopta ese
  índice en lugar de construir otro - así que el resultado es idéntico a volver
  a añadirla sin más. El índice se construye con el nombre propio de la
  restricción, a partir de la definición que da el propio servidor
  (`pg_get_indexdef`) - de modo que una clave con INCLUDE, un fillfactor, un
  tablespace o `NULLS NOT DISTINCT` vuelven tal como estaban, y una restricción
  aplazable conserva su aplazabilidad. `CREATE INDEX CONCURRENTLY` no puede
  ejecutarse dentro de un
  bloque de transacción, y por eso el script de restauración no debe envolverse
  en `BEGIN`/`COMMIT`.

  Es además la única sentencia del script que puede fallar y dejar algo detrás:
  una fila duplicada o una ejecución cancelada dejan un índice que existe y es
  inservible, y nada en la lista de restricciones lo explica. El script elimina
  por sí mismo un índice así antes de reconstruir, de modo que tras un fallo
  basta con volver a ejecutarlo. Sólo se elimina un índice que PostgreSQL haya
  marcado como no válido - uno que esté en uso nunca se toca.
- **Incluir las claves foráneas que apuntan a esta tabla.** Desactivado, el
  script de quitado falla en la clave primaria: la clave foránea de otra tabla
  depende de su índice. Activado, esas tablas dejan de imponer brevemente su
  referencia a esta - el intercambio que pide la situación, dicho en voz alta en
  lugar de escondido dentro de un CASCADE.

Ambos scripts empiezan con `SET lock_timeout = '3s'`. Cada `ALTER TABLE` que
contienen necesita `ACCESS EXCLUSIVE`, y esperar ese bloqueo es lo que convierte
una reconstrucción en línea en una caída: la petición se pone en cola detrás de
lo que ya se está ejecutando, y todo lo que llegue después se pone en cola
detrás de la petición. Tres segundos significan que el peor caso es una
sentencia que hay que repetir, no una tabla que nadie puede leer. Borre la línea
para esperar lo que haga falta. El editor de DBApguin fija dos segundos en cada
conexión de todos modos, así que esto importa sobre todo cuando el archivo se
ejecuta meses después desde `psql`.

Guarda el script de restauración antes de ejecutar el de quitado. Es la única
copia de lo que la tabla imponía.

#### Alcance: una tabla, un esquema o varios {#scope-one-table-one-schema-or-several}

El mismo diálogo responde a los tres. Clic derecho en una **tabla** para esa
tabla; clic derecho en un **esquema** → Transferencia → *Restricciones - quitar
y restaurar* para todas sus tablas. Una migración o un cambio a replicación es
una pregunta sobre el esquema, y hacerlo tabla por tabla es como se olvida una
restricción.

**Qué restricciones** filtra por tipo - claves primarias, únicas, foráneas,
check, exclude. Marcar solo *claves foráneas* es lo habitual: una herramienta
de CDC suele objetar solo a esas, y dejar las claves primarias significa que el
destino conserva su identidad y sus índices.

A nivel de esquema el orden importa más que con una sola tabla, y el script se
ocupa: cada clave primaria y única vuelve antes que cualquier clave foránea que
la referencie, y el script de quitado hace lo contrario. Ordenado por nombre de
tabla - como hacía la primera versión - una clave foránea de `items` se añade
antes de que `orders` tenga su clave primaria, y PostgreSQL lo rechaza con
*there is no unique constraint matching given keys*.

#### Copiar restricciones a otro esquema {#copying-constraints-onto-another-schema}

Nombra un **esquema destino** y el diálogo deja de escribir un script de
quitado y escribe uno que pone estas restricciones sobre las tablas de ese
esquema. El caso para el que sirve: una copia de los datos cargada sin
restricciones - porque así carga rápido, o porque la replicación no habría
arrancado de otro modo - que ahora necesita las mismas reglas que el original.

Los nombres de tabla y columna deben coincidir. Solo viajan las restricciones:
sin datos, sin columnas, sin más índices que los que una clave necesita.

Las definiciones se leen desde dentro del esquema de origen, así que una
referencia a una tabla de ese esquema sale sin cualificar - 
`REFERENCES customers(id)` - y el `SET search_path` del script la apunta al
destino. Una referencia a *otro* esquema se queda cualificada y sigue apuntando
adonde apuntaba, lo cual también es correcto: una clave foránea a un esquema de
referencia compartido no debe redirigirse.

Los nombres de las restricciones pasan sin cambios, porque un nombre de
restricción es único por tabla, no por base de datos. Si el destino ya tiene
una con ese nombre en esa tabla, la sentencia falla - el fallo seguro, ya que
la alternativa es reemplazar algo en silencio.

### 6.10 Búsqueda por similitud: pg_trgm y pgvector {#similarity-search}

Dos extensiones responden a «encuentra filas como esta», de dos maneras
distintas:

- **pg_trgm** compara cómo *se ve* el texto. Corta el texto en trozos de tres
  letras (trigramas), así que una errata o parte de un nombre todavía
  coincide: `LIKE '%pad%'`, `ILIKE`, el operador de similitud `%` y la
  distancia de trigramas `<->`. Viene con los módulos contrib de PostgreSQL.
- **pgvector** compara lo que un modelo dice que el texto *significa*. Un
  modelo convierte cada texto en una lista de números (un embedding, guardado
  en una columna `vector(n)`, `halfvec(n)` o `sparsevec(n)`); números cercanos
  significan significado cercano. Las consultas piden las filas más cercanas:
  `ORDER BY embedding <=> $1 LIMIT 10`.

Necesitan índices distintos, y un btree no sirve a ninguna de las dos: ni
`ILIKE` ni un patrón que empieza con comodín, ni una distancia. Lo que hace la
aplicación con cada una:

**Base de datos -> Gestionar extensiones** muestra las dos con la versión
instalada y la más nueva que trae el servidor; **update** ofrece cada versión
posterior, en orden de versión (0.10 después de 0.9).

**EXPLAIN** lee los índices que usa un plan, no solo sus nombres:

- un escaneo de índice con **HNSW** o **IVFFlat** se nombra como búsqueda
  aproximada de vecinos más cercanos, con la clase de operador para la que se
  creó y el ajuste que cambia velocidad por recall (`hnsw.ef_search`,
  `ivfflat.probes`) tal como lo tenía la sesión que ejecutó el plan;
- un filtro en el mismo paso se señala: el índice entrega primero sus
  candidatos y el filtro se aplica después, así que pueden volver menos filas
  que el `LIMIT`. Desde pgvector 0.8.0 la indicación ofrece
  `hnsw.iterative_scan`;
- un **Sort por una distancia sobre un escaneo secuencial** es una búsqueda
  exacta sobre todas las filas. La indicación dice cuál de tres casos es: no
  hay índice aproximado en la columna; hay un índice creado para *otro*
  operador (un índice con `vector_cosine_ops` solo sirve `<=>`, así que
  `ORDER BY embedding <-> ...` lee toda la tabla); o hay un índice adecuado que
  el planificador no usó, porque solo se usa para
  `ORDER BY columna <op> valor` en orden ascendente con un `LIMIT`;
- un **índice de trigramas** (GIN `gin_trgm_ops`, GiST `gist_trgm_ops`) se
  explica como trozos más una comprobación, con cuántas de las filas
  encontradas no coincidían;
- un escaneo secuencial filtrado por `LIKE`/`ILIKE` sobre una columna que solo
  tiene un btree dice que el btree no puede servirlo y da el índice de
  trigramas; `similarity(a, b) > x` en un filtro nunca se indexa, y la
  indicación dice que se escriba `a % b`.

Los nodos que un `LIMIT` detuvo antes (una búsqueda top-10 lee 10 de unas
20.000 filas estimadas) ya no se marcan como la peor estimación del plan.

**Asesor de índices** (desde pg_stat_statements): para `LIKE`, `ILIKE`, `%` y
`similarity()` sobre una columna propone un índice GIN de trigramas, nunca un
btree, con `CREATE EXTENSION pg_trgm` delante donde la base aún no lo tiene (el
diálogo Run muestra y ejecuta ambos). Para `ORDER BY columna <distancia> ...`
propone un índice HNSW con la clase de operador del operador que usan las
consultas, leída del catálogo del propio servidor, y dice el precio:
aproximado, puede perder algunas de las filas realmente más cercanas. Nunca
propone un btree sobre una columna vectorial. Una columna con más dimensiones
de las que admite un índice (2.000 para `vector`, 4.000 para `halfvec`) o sin
dimensión declarada recibe la explicación en vez de una sentencia.

**Ajustes clave**, donde pgvector está instalado en la base: `hnsw.ef_search`,
`hnsw.iterative_scan`, `hnsw.max_scan_tuples`, `ivfflat.probes` e
`ivfflat.iterative_scan` (los iterativos desde pgvector 0.8.0), y
`pg_trgm.similarity_threshold` donde está pg_trgm. `maintenance_work_mem` se
compara con el mayor índice HNSW: el README de pgvector dice que construir un
HNSW es bastante más rápido cuando el grafo cabe en él, y que en un contenedor
una construcción en paralelo necesita un `/dev/shm` al menos así de grande.
`max_parallel_maintenance_workers` dice que pgvector construye sus índices con
ellos.

**Tamaños.** Esquema -> una tabla -> **Almacenamiento** muestra cada columna
vectorial con los bytes que ocupa un valor (fórmula de pgvector: `vector`
4 x dimensiones + 8, `halfvec` 2 x dimensiones + 8), lo que ocupa una muestra
de valores guardados y, en PostgreSQL 17, cuántos de ellos viven en la tabla
TOAST: un `vector(768)` ocupa unos 3 kB, más de lo que una fila guarda dentro,
y por eso una tabla así muestra un heap pequeño y una tabla TOAST grande.
Debajo, los índices de la tabla con método, clase de operador, opciones (`m`,
`ef_construction`, `lists`) y tamaño. La comprobación **Tamaños de índice**
nombra el método y la clase de operador de cada índice y el tamaño TOAST de su
tabla.

**Crear índice** ofrece `hnsw` e `ivfflat` como métodos y una clase de operador
para las columnas (`vector_cosine_ops`, `gin_trgm_ops`, ...); ASC/DESC y NULLS se
ofrecen solo para btree, el único método que los acepta.

**Copias.** DDL, Comparar y Copiar conservan clases de operador y opciones de
índice. Una clonación de esquema con filas crea los índices HNSW e IVFFlat
después de las filas (un índice IVFFlat calcula sus listas con las filas
presentes al crearlo), y Copiar tabla reconstruye un índice IVFFlat cuando las
filas ya están dentro. Comparar añade una nota encima de un cambio de dimensión
de una columna vectorial: los vectores guardados no se convierten a otra
dimensión, así que hay que volver a generar los embeddings de las filas.

## 7. Ajustes

El engranaje de arriba a la derecha abre los Ajustes:

- Tema: oscuro / claro
- **Idioma: English, Hrvatski, Deutsch, Español.** La elección se guarda en el
  servidor (`app_prefs.json`), así que te acompaña entre reinicios en vez de vivir
  en un perfil de navegador. Se aplica al instante, sin reiniciar, incluso a los
  menús y diálogos ya abiertos.
- Tamaño de letra (10-30 px)
- **Inicio -> Puerto:** el puerto en el que escucha el backend propio de
  DBApguin - solo en este equipo, `127.0.0.1` - 3236 si no lo cambia. Cámbielo
  si otro programa necesita el 3236; se aplica la próxima vez que inicie
  DBApguin. Lo que recuerda la ventana (tema, favoritos, historial SQL,
  pestañas abiertas) se traslada con él.
- Registro de auditoría: activar / desactivar (ver más abajo)
- Herramientas: rutas a psql / pg_dump / pg_restore / pg_repack (sustituyen al descubrimiento automático)
- Certificados: subir certificados raíz en PEM para `sslmode=verify-*`
- Certificados y rutas como arriba; **Acerca de se ha movido a Ayuda -> Acerca de DBApguin**, junto a los documentos legales

### Registro de auditoría {#audit-log}

Activado hasta que lo desactives - el interruptor está en la propia ventana
del registro. Cada sentencia que ejecuta el editor SQL (salvo si la pestaña es
de solo lectura) y cada cambio hecho con las herramientas de DBApguin se añade a
`audit/audit-YYYY-MM.jsonl` (un archivo por mes) - la hora, el destino, tu
usuario del sistema operativo y el usuario de la base de datos, la sentencia
con las contraseñas censuradas, la duración y el resultado. Ábrelo desde
**Ver -> Registro de auditoría**:
busca por rango de fechas, acción o una subcadena del SQL, exporta un rango
como JSONL, o lee los archivos directamente con cualquier editor de texto o
`jq`.

**Cada registro está encadenado al anterior.** Cada línea lleva un `hash` de
su propio contenido más el `hash` de la línea anterior (`prev_hash`), así que
cambiar o borrar una línea pasada cambia cuál debería haber sido el hash de la
siguiente línea - **Verificar cadena** en el visor del registro recalcula toda
la cadena y dice o bien "intacta" o exactamente qué archivo y línea es el
primero que ya no coincide. Un registro escrito antes de que esto existiera no
tiene ninguno de los dos campos; el verificador trata esos registros como
anteriores a la cadena en vez de fallar con ellos, e indica dónde empieza la
cadena en sí. La cadena también sobrevive a la rotación mensual - el primer
registro de un mes nuevo enlaza de vuelta al último registro del archivo
anterior, así que dividir el registro en archivos no divide la cadena.

**Lee con cuidado lo que esto demuestra, porque es fácil sobrestimarlo.**
Demuestra que nadie hizo una edición casual e inadvertida a una cadena que
verifica - útil por sí solo, y la razón de que exista el botón. **No**
demuestra que nadie con acceso de escritura al archivo lo haya manipulado: el
hashing no necesita ningún secreto, el algoritmo es público, y el registro
está justo al lado de la aplicación que puede recalcularlo - así que quien
pueda editar `audit-2026-05.jsonl` también puede recalcular todos los hashes
posteriores a su edición, y la cadena volverá a verificar como si nada hubiera
pasado. La protección real contra eso necesita un anclaje fuera de esta
máquina - una firma con una clave que no posees, o el hash enviado a otro
lugar en el momento en que se escribe - y esta función no hace eso. Donde
necesites un registro que sobreviva a un administrador local comprometido, usa
el propio registro del servidor de base de datos (`pgaudit`,
`log_statement`) escribiendo a un destino separado y con acceso controlado,
no este archivo. Ver el [acuerdo de licencia](EULA.md), sección 7 (Ayuda -> Acuerdo de licencia).

### Datos de la app: exportar, importar y borrar {#app-data-backup-and-clear}

Todo lo que DBApguin conserva entre ejecuciones vive en un directorio (su ruta aparece en ambos diálogos). Ajustes -> Datos de la app tiene tres botones.

**Exportar datos de la app** escribe un `.tar.gz` de lo que marques: conexiones guardadas, preferencias, consultas guardadas, el espacio de trabajo SQL, la lista de tareas, informes guardados, las instantáneas AWR, los días ASH y las trazas de sesión guardados de cada servidor, el registro de auditoría, los plugins instalados y sus datos, los certificados TLS. Por defecto todo está marcado. La clave de datos va con las conexiones y los datos de los plugins, porque sus contraseñas no se pueden leer sin ella.

**Importar datos de la app** lee un archivo así, muestra lo que contiene junto a lo que ya hay aquí y toma lo que marques (de nuevo, todo por defecto). Combina en lugar de reemplazar: el historial guardado (instantáneas AWR, días ASH, trazas de sesión, informes guardados, registros de trabajos, el registro de auditoría, plugins) se añade y nunca sobrescribe un fichero que ya existe; las conexiones guardadas se combinan por id, un nombre ya usado recibe un número y sus contraseñas se vuelven a cifrar con la clave de esta instalación; un fichero suelto como las preferencias o las consultas guardadas reemplaza el tuyo. Un archivo sin clave, o con una protegida por contraseña maestra, importa las conexiones sin contraseñas y lo dice. Así un DBA lleva el historial recogido y su configuración a otro equipo; no hay un almacén compartido entre instalaciones.

> La copia **incluye `key` y `connections.json`** - la clave de datos y tus contraseñas de conexión cifradas. Es intencionado: una copia de tu configuración sin ellas no lo es. Pero significa que el archivo es sensible. Guárdalo en un lugar privado y **nunca en una carpeta que se sincronice con un servicio en la nube** (Dropbox, OneDrive, iCloud, Nextcloud, una carpeta Documentos sincronizada). Si hay una contraseña maestra, las contraseñas del archivo siguen cifradas con ella; sin una, solo están ligeramente ofuscadas.

La copia también lleva tus **plugins instalados** (`plugins/`) y lo que guardan (`plugin-data/`): sus ajustes y las contraseñas que almacenan, cifradas con la misma `key`, de modo que al restaurarla los plugins vuelven funcionando. La carpeta en la que escribes tus propios plugins, si lo haces, no está incluida: está donde tú elegiste.

Exportar conexiones desde el gestor de conexiones es otra cosa: escribe solo las conexiones y sus grupos, sin contraseñas, ajustes ni plugins.

**Borrar datos de la app** muestra una lista con el tamaño de cada elemento. Informes y registros - snapshots AWR, registros de trabajos, informes de comprobación guardados, el registro de errores - están marcados por defecto; las preferencias, las consultas guardadas y el espacio de trabajo SQL se listan sin marcar. Las conexiones guardadas (`connections.json` + `key`) están en una sección aparte y requieren escribir la palabra `DELETE` para confirmar, porque borrarlas pierde cada conexión guardada y su contraseña. Los plugins instalados aparecen sin marcar; sus ajustes y contraseñas guardadas (`plugin-data/`) están en la misma sección protegida que las conexiones, porque borrarlos pierde esas contraseñas. Un directorio que aún tiene un trabajo en segundo plano en ejecución se rechaza; cancela el trabajo primero. Cada borrado se escribe en el registro de auditoría.

### Plugins {#plugins}

**Plugins → Gestionar plugins…** muestra lo instalado: activar o desactivar un plugin, quitarlo o instalar uno desde un archivo `.dbapg-plugin`. Un plugin añade comprobaciones, entradas de menú y pestañas sin una nueva versión de DBApguin.

**Cualquiera puede crear un plugin**: la aplicación incluye una herramienta para ello (activa *Desarrollar plugins en esta aplicación* en la misma pestaña; la *Guía de plugins* explica el resto). Un plugin se ejecuta con tus permisos: puede acceder a cada base de datos a la que te conectes y a cualquier otro servicio para el que esté configurado. **Usa solo plugins de editores en los que confíes.** El acuerdo de licencia (sección 15) y el aviso de privacidad dicen lo mismo con sus palabras.

**Información** en cada plugin lo explica en dos partes. Primero lo que el plugin dice de sí mismo - qué hace, qué lee, qué envía y adónde - en tu idioma cuando trae uno. Después lo que DBApguin lee por sí mismo de los archivos del plugin, diga lo que diga el plugin: si ejecuta Python dentro de la aplicación o JavaScript en la ventana, qué comprobaciones y direcciones añade, qué guarda, su carpeta y cada archivo.

**Certificado por DBApguin** junto al nombre de un plugin significa que lo firmó el autor de DBApguin y que ninguno de sus archivos ha cambiado desde entonces. *Cambiado tras la certificación* significa que hay archivos distintos de los firmados (Información los nombra); *firma no válida* significa que la firma no se sostiene; *no certificado* significa que nada garantiza de dónde viene, que es como se ve también un plugin escrito por tu equipo. Un plugin no certificado se instala y funciona igual: la marca te dice de dónde viene un plugin, no decide por ti. Instalar uno no certificado pregunta antes, y la pregunta muestra qué ejecutaría y cada archivo que trae.

Quitar un plugin borra también sus ajustes y contraseñas guardadas. Una copia de los datos de la app lleva los plugins y sus ajustes.

## 8. Notificaciones

Esquina inferior derecha. Con código de color:
- Azul (info): estado neutro
- Verde (ok): éxito
- Rojo (error): la operación falló

Un clic en la pila la expande y permite desplazarse. La rueda desplaza. Un clic fuera la contrae.

Mientras hay una ventana abierta, las notificaciones dejan libres sus botones. Una nueva se coloca encima de ellos o en el espacio junto a la ventana; si no hay sitio en ninguno, pasa directamente al contador, cuyo número y color siguen indicando que llegó. El contador también queda por encima de los botones de la ventana; si ni él cabe, se oculta hasta que la ventana se cierra. Al cerrarla, todo vuelve a la esquina.

Una sola acción ofrece «Deshacer»: eliminar una conexión guardada, que el aviso puede restaurar durante unos seis segundos. Un aviso no puede deshacer lo que llegó a una base de datos; un ajuste cambiado aquí se puede deshacer desde la vista de ajustes, y una eliminación, un movimiento o un cambio de propietario deja una copia - en una eliminación con las filas, si se eligieron, y ninguna si elige eliminar sin copia - que devuelve los objetos (ver *Una copia de las definiciones antes del cambio*). Lo que hace la aplicación en su lugar es mostrarte la sentencia antes de ejecutarla, preguntar una vez y registrarla en el log de auditoría.

### 8.1 Mientras algo está en curso {#while-something-runs}

Todo lo que has pedido y tarda más de un cuarto de segundo lo dice, siempre en los mismos sitios:

- Una barra fina se desliza por el borde superior de la ventana y un anillo gira en el extremo derecho de la fila de pestañas bajo los menús. Señala el anillo, o llega a él con Tab, para ver la lista de lo que está en curso, el paso en que está cada cosa y cuánto lleva; pasados tres segundos el anillo muestra el tiempo él mismo. Donde el trabajo se puede detener - una comprobación, una sentencia en el editor SQL - su línea en la lista tiene un Cancelar.
- El botón que has pulsado muestra un anillo en lugar de su texto y no admite una segunda pulsación hasta que el trabajo termina. Nada a su lado se mueve: el botón conserva su tamaño, y la barra, el anillo y la lista no le quitan sitio a la página.
- Una parte de la ventana que está cargando lo dice ("Leyendo los trabajos…", "cargando columnas…") en lugar de mostrar una lista vacía o valores que están a punto de cambiar, y pasados tres segundos muestra cuánto lleva esperando. Un ajuste cuyo valor guardado aún se está leyendo aparece atenuado y no se puede cambiar hasta que llega el valor.

Las exportaciones y los informes nombran cada paso: generando el informe, esperando tu elección en el diálogo de archivos (si el diálogo se abrió detrás de la ventana, esa línea es la pista), escribiendo el archivo o el PDF. El estado de trabajo termina en cuanto termina el trabajo, haya salido bien o mal, y una notificación dice adónde ha ido el archivo o por qué no.

Lo que la aplicación hace por su cuenta - la actualización del panel, las comprobaciones de conexión, el vigilante de trabajos, las consultas de los recolectores - nunca aparece aquí, así que un anillo que gira siempre significa que algo que has pedido sigue en camino.

## 9. Atajos de teclado

- `F5` (o `Ctrl/Cmd + Enter`): ejecutar la consulta en el editor SQL
- `F6` (o `Ctrl/Cmd + Mayús + Intro`): ejecutar solo la selección, o la sentencia donde está el cursor
- `Ctrl/Cmd + S`: guardar el SQL actual como fragmento
- `Ctrl/Cmd + E`: validar la consulta
- `Ctrl/Cmd + K`: paleta de comandos
- `Esc`: cerrar el diálogo o menú contextual actual, limpiar la búsqueda
- `Ctrl/Cmd + W`: cerrar la pestaña de trabajo actual (si no está fijada)
- `Mayús + Tab`: cambiar de pestaña, como Alt + Tab cambia de ventana. Se abre una ventana de tarjetas, una por pestaña abierta, cada una con una imagen del contenido tal como lo vio por última vez, y la siguiente pestaña marcada. Mantenga Mayús y pulse Tab o las flechas para mover la marca (Izquierda retrocede, Arriba y Abajo saltan una fila). Suelte Mayús para abrir la pestaña marcada; Esc cierra la ventana y se queda donde estaba. En un campo de texto o en el editor SQL, Mayús + Tab conserva su significado habitual (campo anterior, quitar sangría); use allí `Ctrl + Tab` y suelte Ctrl para abrir la pestaña.
- `Ctrl + Mayús + Tab`: cambiar de conexión en el mismo tipo de ventana: una tarjeta por conexión abierta con una imagen de lo que había en pantalla, su servidor, su estado y cuántas pestañas tiene. La conexión cambia al soltar Ctrl, no mientras mira. En macOS sigue siendo la tecla Control, porque macOS reserva Cmd + Mayús + Tab.
- `F10`, o una pulsación de `Alt` (Opción en un Mac): abrir la barra de menús con el teclado. Izquierda y Derecha cambian de menú, Arriba y Abajo recorren las entradas, Derecha abre un submenú e Izquierda lo cierra, una letra salta a la siguiente entrada que empieza por ella, Intro ejecuta la entrada (una comprobación se abre como con un clic), Esc cierra un nivel cada vez. Un menú abierto con el ratón acepta las mismas teclas.

Las teclas se muestran donde se usan, con los nombres de su sistema (en un Mac los símbolos de Comando, Opción y Control): en los botones del editor SQL y sus descripciones, en esta lista (tecla `?`) y como un breve consejo después de hacer lo mismo varias veces con el ratón. Cada consejo aparece como mucho dos veces; *No más consejos* los desactiva y el interruptor al final de la lista `?` los vuelve a activar.

## 10. Notas por proveedor

DBApguin detecta el proveedor del clúster al conectar y se adapta:

| Proveedor | Qué está restringido |
|----------|---------------|
| Aurora PostgreSQL | Sin ALTER SYSTEM, sin escritura directa de pg_hba, sin creación de roles de nivel superusuario, sin tablespaces propios, sin lectura directa de archivos |
| Amazon RDS PostgreSQL | Igual que Aurora más una lista blanca para CREATE EXTENSION |
| Google Cloud SQL | Igual que RDS |
| Google AlloyDB | Igual que Cloud SQL: los ajustes son los flags de base de datos de la instancia. Un servidor AlloyDB Omni que gestiona usted mismo tiene un superusuario real y cuenta como PostgreSQL propio |
| Azure Postgres Flexible | Igual que RDS |
| Supabase | Igual que RDS más restricciones de roles reservados |
| Neon | Inquilino de una sola base; sin CREATE DATABASE / DROP DATABASE |
| Heroku Postgres | Igual que Neon |
| Crunchy Bridge | Casi todas las funciones de empresa disponibles |
| DigitalOcean Managed | Igual que RDS |
| Timescale Cloud | Lista blanca de extensiones más la restricción del rol tsdbadmin |
| PostgreSQL propio | Todo disponible |

La interfaz atenúa las opciones bloqueadas con un tooltip que explica por qué. Cuando el rol de la sesión es un superusuario real, no se atenúa ni se rechaza nada: ninguna plataforma gestionada da uno a un cliente, así que en ese servidor esos límites no se aplican (por ejemplo, un servidor propio que importó los nombres de rol de una plataforma). La plataforma se sigue nombrando y su aviso dice que esta sesión es un superusuario. El vigilante `db-errors` recurre a las diferencias de contadores y a las indicaciones de acceso al log propias del proveedor cuando `pg_read_file` está denegado.

#### Guardarlas en un archivo .sql {#backing-them-up-as-a-sql-file}

**Script de quitado…** y **Script de restauración…** muestran primero el script
en pantalla, con las tres opciones que ofrece todo script generado en esta
aplicación: copiarlo, guardarlo o abrirlo en el editor SQL y ejecutarlo allí.
Guardar se niega a sobrescribir un archivo existente sin preguntar. El archivo
de restauración es la copia de seguridad: lleva las sentencias de creación y el
paso de validación, y es lo que hay que conservar antes de que corra el script
de quitado.

Cada archivo empieza con una cabecera - versión de la aplicación, servidor y base
de datos de origen, alcance, momento en que se tomó y cuántas restricciones
contiene. La contraseña no aparece nunca. Seis meses después, esa cabecera es la
diferencia entre un archivo utilizable y un montón anónimo de sentencias ALTER
TABLE.

Para una base entera en vez de un esquema, `pg_dump --section=post-data` llega al
mismo sitio por el otro lado: escribe las restricciones, los índices y los
disparadores de todo a la vez. Eso es Copia de seguridad → Opciones → Section →
post-data.

### Logins IAM y nombres generados

En una nube gestionada el rol de login suele **ser** el principal de IAM, así que
su nombre es una dirección de correo o una cuenta de servicio:

```
alice@example.com
123456-compute@developer.gserviceaccount.com
```

Son nombres de rol válidos en PostgreSQL: los identificadores entrecomillados
pueden contener `@`, `.` y `-`, y empezar por un dígito. DBApguin los acepta en
todos los sitios donde se pide un nombre de rol: crear, eliminar, conceder,
revocar, pertenencia, `search_path`, propiedad y cada pantalla de privilegios. El
SQL que emite los entrecomilla, así que una concesión se lee así:

```
GRANT SELECT ON TABLE "app-schema"."orders" TO "alice@example.com"
```

Lo mismo vale para nombres de bases y esquemas. Los proveedores gestionados
generan nombres como `b2b-ext-assistant-dev-pg-db` de forma rutinaria, y los
guiones son legales en un identificador entrecomillado.

Lo que sí se rechaza, a propósito, es un nombre con comillas dobles, NUL, espacios
en blanco o caracteres de control: nada de eso tiene un principal real de la nube,
y todos podrían terminar un identificador antes de tiempo.

Los usuarios IAM **no tienen contraseña**. Al crear uno, elige `Sin contraseña` en
el asistente y concede después la pertenencia IAM del proveedor (`rds_iam` en AWS,
`cloudsqliamuser` en Cloud SQL) desde la lista «Miembro de».

## 11. Resolución de problemas

### «Se requiere contraseña»
La conexión guardada no tiene contraseña almacenada. Primero se intenta así: un servidor local que no necesita contraseña (trust, peer, `.pgpass`) simplemente conecta. Un servidor que la pide abre el diálogo de refresco; escribe la contraseña y pulsa Conectar. Si la pestaña se conecta de otra forma mientras el diálogo está abierto, el diálogo se cierra solo.

### «Me niego a escribir dentro de <ruta>»
El destino de la copia de pg_dump está en un lugar de solo lectura (montaje del AppImage, directorio de datos de la aplicación). Elige una ruta bajo `~/Documentos` o similar.

### «pg_stat_statements debe cargarse mediante shared_preload_libraries»

pg_stat_statements tiene dos mitades, y ambas son necesarias:

1. la biblioteca debe figurar en `shared_preload_libraries`, y
2. la extensión debe estar creada en la base que consultas.

`CREATE EXTENSION pg_stat_statements` hace solo lo segundo. La extensión aparece
entonces en `pg_extension`, todo parece instalado, y cada lectura de la vista da
este error. Volver a ejecutar `CREATE EXTENSION` es lo siguiente que uno intenta,
y no cambia nada.

DBApguin reconoce ese estado y dice qué mitad falta. Toda comprobación que lea
estadísticas de consultas imprime el motivo y el remedio bajo el error, y
**db-setup-advisor** lo informa como *«Creada en esta base pero NO en
shared_preload_libraries»* en lugar de un simple «no instalada».

El remedio, como superusuario:

```
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';
```

y después **reinicia** PostgreSQL. `shared_preload_libraries` solo se lee al
arrancar el postmaster: `SELECT pg_reload_conf()` no lo recoge. En una nube
gestionada, fíjalo en el grupo de parámetros y reinicia la instancia.

El estado inverso - biblioteca cargada, extensión no creada - no necesita reinicio
ni intervención: las comprobaciones crean la extensión ellas mismas y siguen.

Comprobaciones afectadas mientras está a medias: db-stat-statements,
db-index-advisor (solo su heurística de pg_stat_statements), db-temp-queries,
db-workload-report, db-awr-snapshot y db-trace-session.

### El índice existe pero el planificador lo ignora {#leftover-damage}

Casi siempre significa que el índice es NO VÁLIDO, y ningún listado normal lo
dice. `CREATE INDEX CONCURRENTLY` y `REINDEX CONCURRENTLY` construyen por
fases, y una sesión que muere a medias - una caída, un kill, un tiempo de
espera de bloqueo - deja atrás el índice a medio construir. El planificador no
lo usa, sigue ocupando espacio, y `\d` lo marca sólo si se mira de cerca.

**Cluster health → Buscar daños residuales...** busca los estados que
PostgreSQL registra pero no muestra:

- índices no válidos, de una construcción CONCURRENTLY interrumpida
- restricciones `NOT VALID`, que no se aplican a las filas existentes: una
  restauración que aplazó la validación deja estas
- disparadores deshabilitados, que una restauración con `--disable-triggers`
  deja apagados si se interrumpió

Elija un esquema o deje *(toda la base de datos)*. Cada hallazgo viene con la
sentencia que lo corrige, y un índice no válido recibe un botón **Corregir
ahora** que ejecuta `REINDEX INDEX CONCURRENTLY` sobre él como tarea en
segundo plano.

Una cosa que esperar: un `REINDEX CONCURRENTLY` fallido deja su propia copia a
medio construir con el sufijo `_ccnew`, que es un objeto distinto del original.
Reparar el original no la elimina: vuelva a ejecutar la comprobación después y
corrija también esa.

### La ventana no aparece (Linux)
DBApguin abre un diálogo con órdenes de instalación por distribución cuando falta WebKit2GTK. Elige la pestaña de tu distribución, pulsa «Copiar comando», pégalo en un terminal y vuelve a lanzar el AppImage. El diálogo nunca recurre a un navegador: DBApguin funciona solo como aplicación independiente.

### La ventana no aparece (Windows)
DBApguin abre un diálogo con tres vías de instalación de Microsoft Edge WebView2 Runtime: el bootstrapper incluido en la carpeta de la aplicación, el instalador en línea y las indicaciones para despliegue en empresa (GPO/SCCM/Intune).

### La ventana se cuelga al cerrar (Windows)
Debería estar resuelto desde la compilación de junio de 2026. La aplicación se cierra a la fuerza 2 segundos después de pulsar la X. Si aún se cuelga, adjunta el contenido de `<DBCHECKS_HOME>\dbchecks_error.log` al informe de fallo.

### DBApguin no se inicia: el puerto está ocupado
- Cuando otro programa ya usa el puerto de DBApguin, la ventana sigue abierta y
  lo indica, con un campo para iniciar en otro puerto - ya trae uno libre.
  DBApguin conserva ese puerto; cámbielo después en **Ajustes -> Inicio**.
- Cuando indica que DBApguin ya se está ejecutando, sigue abierta una copia
  anterior: cambie a su ventana, o detenga el proceso que nombra y elija
  **Reintentar**.

### El explorador de esquema está vacío
- ¿Estás conectado? Mira el punto de estado junto a la conexión.
- ¿Cambiaste de base? Haz clic en otra base bajo **Bases de datos** para cambiar; el árbol se recarga con ella.
- Pasa el ratón sobre la cabecera **Bases de datos**, una base o un esquema y pulsa el icono de refresco que aparece, o usa **Esquema -> Recargar el árbol de esquemas**.

### No hay registro de auditoría
- Está activado salvo que se desactive: revisa el interruptor en Ver -> Registro de auditoría
- O revisa la clave `audit_enabled` en `app_prefs.json` dentro de tu directorio de datos
- Los archivos están en `<DBCHECKS_HOME>/audit/audit-YYYY-MM.jsonl`

### Dónde están los datos de la aplicación
En Linux: `~/.local/share/DBApguin/`. Se sustituye con `DBCHECKS_HOME=/ruta .venv/bin/python launcher.py`.

En Windows: por defecto la carpeta junto a `DBApguin.exe`.

## 12. Referencia de archivos de datos

| Archivo | Qué contiene | Permisos |
|------|--------------|------|
| `connections.json` | Conexiones guardadas con contraseñas cifradas con Fernet | 0644 |
| `key` | Clave aleatoria del cifrado de contraseñas, propia de cada instalación | 0600 |
| `app_prefs.json` | Preferencias de interfaz (font_size_px, audit_enabled, theme y demás) | 0644 |
| `certs/` | Certificados raíz TLS subidos | 0600 cada uno |
| `audit/audit-YYYY-MM.jsonl` | Registro de auditoría de DDL, un archivo por mes | 0600 |
| `dbchecks_error.log` | Registro de trazas (rota a 1 MiB, guarda 3 copias) | 0644 |
| `webview/` | localStorage / cookies de pywebview | (solo Linux) |

Haz copia de esto antes de reinstalar. Reinstalar DBApguin en el mismo directorio de datos conserva tus conexiones.

## 12.4 Dónde está la documentación

Tres vías, todas desde **Ayuda** en la barra de menú:

| | |
|---|---|
| **Guía de usuario** | este documento, incluido en la aplicación - funciona sin conexión |
| **Documentación (en línea)** | el sitio web, para cualquier cosa más reciente que tu compilación |
| **Acerca de DBApguin** | versión y compilación, con las dos anteriores al lado |

El panel de licencia del primer arranque y el aviso posterior enlazan ambos a
esto, que es donde la mayoría mira la primera vez.

Los documentos se muestran dentro de la aplicación, con su propio estilo:
encabezados, tablas y ejemplos de SQL en un bloque de código de verdad, no como
texto reflotado.

Hay una cuarta vía, y en la práctica es la que más usarás: la pequeña **i** en un
círculo junto al título de un diálogo. Abre esta guía en la sección que describe
ese diálogo, así que la explicación queda a un clic de la cosa que explica, en
lugar de a una búsqueda de distancia. La encontrarás en los diálogos de copia y
restauración, en renombrar y fusionar esquemas, y junto al modo de transacción
del editor SQL.

La copia incluida es la que viaja con tu compilación, así que siempre coincide
con la versión que ejecutas. La copia en línea puede ir por delante.

## 12.5 Aspectos legales, privacidad y software de terceros

Todo lo de abajo es accesible desde **Ayuda** en la barra de menú, y viaja con la
aplicación: el archivo que muestra es el archivo con el que se compiló.

- **Contrato de licencia (EULA)** - qué puedes y qué no puedes hacer con el
  software, la exención de garantía y el límite de responsabilidad. La sección 7
  merece una lectura aunque te saltes el resto: trata del hecho de que esta
  herramienta emite `DROP`, `TRUNCATE` y `pg_terminate_backend` contra servidores
  en producción por indicación tuya.
- **Aviso de privacidad** - qué se guarda en tu máquina y dónde. La versión corta:
  todo es local, no hay telemetría, y las únicas conexiones de red son a tus bases
  de datos y a `127.0.0.1`.
- **Avisos de terceros** - cada componente de código abierto que enlaza la
  compilación, con su licencia. Se genera desde la propia compilación
  (`tools/legal/gen_third_party.py`) en vez de mantenerse a mano, así que no puede
  describir un conjunto de dependencias que ya no existe.

Los documentos legales están escritos en inglés y el inglés prevalece. Las
traducciones de la interfaz no se extienden a ellos, y es a propósito: un contrato
de licencia traducido que no ha pasado por un abogado en ese idioma es peor que no
tener traducción, porque parece autorizado.

## 12.6 Por qué se ve igual en todas las máquinas

DBApguin incluye las dos tipografías que usa - Inter para la interfaz y Cascadia
Mono para todo lo monoespaciado - en lugar de nombrar tipografías y confiar en que
la máquina las tenga.

Eso no es cosmética. Una herramienta que muestra identificadores tiene que
mostrarlos exactamente: cuando la aplicación nombraba una tipografía no instalada,
el sistema sustituía por otra, y una de las sustitutas dibujaba el guion bajo por
debajo de la caja de línea, así que `pg_read_all_stats` podía leerse como
`pgreadallstats` en una máquina y bien en otra. Incluir la tipografía elimina la
adivinanza.

Ambas están bajo la SIL Open Font License 1.1 y viajan sin modificar, con sus
textos de licencia en `static/fonts/` y en **Ayuda → Componentes de terceros**.

## 13. Si vienes de Oracle {#coming-from-oracle}

Si llevas años administrando bases Oracle, casi todo lo que sabes se traslada. Lo
que sigue es la parte que no, y dónde viven los números equivalentes en esta
aplicación.

### 13.1 El mapa

| Oracle | PostgreSQL | Dónde en DBApguin |
| --- | --- | --- |
| Instancia + base de datos (una por instancia, normalmente) | Clúster + muchas bases; un grupo de procesos de servidor las atiende todas | Barra lateral de servidores; cada base es un nodo |
| Esquema == usuario | Esquema y rol son objetos distintos; un rol puede poseer muchos esquemas | Explorador de esquema; vista Roles |
| Tablespace (obligatorio, dimensionado) | Tablespace (opcional, solo un directorio) | `db-tablespaces` |
| Tablespace de undo, segmentos de rollback | No hay undo: las versiones antiguas de fila se quedan en la tabla hasta que vacuum las quita | Asesor de autovacuum, `db-bloat-breakdown` |
| Redo logs + archive logs | Segmentos de WAL + archivado | `db-checkpoints`, `db-settings` (GUC de WAL y archivado) |
| SGA / caché de buffers | `shared_buffers` + la caché de páginas del sistema | `db-settings`, `db-cache-advisor`, `db-buffer-cache` |
| PGA / área de ordenación | `work_mem`, por operación y por backend | `db-settings` (el AVISO de ahí suele ser sobre esto) |
| Informe AWR | `db-workload-report` sobre deltas de pg_stat_statements | Menú Almacenamiento y rendimiento |
| Instantánea / línea base AWR | `db-awr-snapshot` (tomar / listar / comparar) | Menú Almacenamiento y rendimiento |
| ASH / v$active_session_history | Muestreador `db-ash` sobre pg_stat_activity | Menú Conexiones y consultas |
| v$session | Vista Sesiones; clic en una fila para el detalle completo | Vista Sesiones |
| v$sql / library cache | `pg_stat_statements`, `db-stat-statements` | Menú Conexiones y consultas |
| SQL trace / 10046 | `db-trace-session` más `auto_explain` en el servidor | Menú Ajuste y diagnóstico, *Traza de sesión* |
| DBMS_STATS | `ANALYZE`, la mitad «analyze» de autovacuum | Pestaña Asesor de una tabla |
| Hints del optimizador | No hay hints. Cambias estadísticas, índices o la consulta | `db-index-advisor`, EXPLAIN en la vista SQL |
| RMAN | `pg_dump` / `pg_restore` (lógico), `pg_basebackup` (físico) | Copia y restauración |
| Data Guard | Replicación en streaming + slots de replicación | `db-replication`, `db-repl-slots` |
| Refresco de vista materializada (fast/complete) | `REFRESH MATERIALIZED VIEW [CONCURRENTLY]`, siempre una reconstrucción completa | Clic derecho en una matview |
| Vistas DBA_* / V$ | Vistas `pg_catalog.pg_*` y `pg_stat_*` | Cada comprobación imprime el SQL que ejecutó |
| Objetos inválidos, recompilar | No existe ese estado: una función se analiza cuando se ejecuta | la comprobación posterior a la importación, que se ejecuta automáticamente tras un restore |
| Sinónimos públicos | `search_path`, o una vista en un esquema compartido | `db-settings` |
| Secuencias con NOCACHE / ORDER | Las secuencias cachean por sesión; los huecos son normales y no un error | `db-sequences`, comprobación de salud de secuencias |
| Perfiles, resource manager | No viene de serie. Límites de conexión por rol, más un pooler | `db-connections` |

### 13.2 Las cinco diferencias que de verdad duelen

**1. No hay undo: vacuum es el precio de MVCC.** Un `UPDATE` escribe una nueva versión
de la fila y deja la antigua ahí, muerta pero todavía en la página. Autovacuum retira
las versiones muertas más tarde. Dos consecuencias sin equivalente en Oracle: una tabla
puede crecer aunque el número de filas no lo haga (bloat), y una transacción abierta
durante horas impide que vacuum limpie *ninguna* tabla, porque las versiones antiguas
podrían seguir siendo visibles para ella. `db-long-tx` y `db-idle-in-tx` son las
comprobaciones que las encuentran; la pestaña Asesor de una tabla muestra exactamente
cuántas filas muertas hacen falta antes de que autovacuum actúe.

**2. Los esquemas no son usuarios.** `CREATE USER app` no crea un sitio donde `app`
ponga tablas. Los objetos aterrizan en el primer esquema de `search_path` - normalmente
`public`-. Concede `USAGE` sobre el esquema *y* privilegios sobre los objetos: son dos
cosas separadas, y un `USAGE` de esquema ausente es el «permiso denegado» más habitual
de los que parecen inexplicables. La vista Roles muestra ambas capas, y el editor de
privilegios escribe las dos.

**3. El DDL es transaccional.** `BEGIN; ALTER TABLE ...; ROLLBACK;` deshace de verdad el
`ALTER`. Las migraciones pueden ser todo o nada, lo que es una ventaja real, pero también
significa que una migración que mantiene `ACCESS EXCLUSIVE` sobre una tabla caliente
bloquea a todos los lectores hasta que confirma. `db-locks` y la vista Sesiones muestran
quién espera detrás de quién.

**4. No hay hints, y las estadísticas son toda la historia.** Si un plan es malo, la
respuesta es `ANALYZE`, un índice mejor, un `default_statistics_target` ajustado o una
consulta reescrita. La pestaña Asesor informa de cuándo se analizó la tabla por última
vez; `db-index-advisor` informa de los índices que faltan, incluidas las claves foráneas
sin índice detrás - que en PostgreSQL hacen que los borrados en la tabla padre escaneen
la hija.

**5. Las conexiones son procesos, no hilos.** Cada conexión es un proceso del sistema con
su propia memoria. Varios cientos de conexiones inactivas cuestan RAM de verdad y hacen
más lenta cada instantánea de `pg_stat_activity`. Donde Oracle usaría servidores
compartidos, PostgreSQL usa un pooler externo (PgBouncer). `db-connections` muestra lo
cerca que estás de `max_connections` y cuántas de esas están inactivas.

### 13.3 Cosas sin equivalente en Oracle

- **TOAST.** Los valores de más de unos 2 KB se comprimen y se guardan en una tabla
  lateral. El tamaño propio de una tabla puede parecer pequeño mientras su tabla TOAST
  guarda la mayor parte de los datos; `db-bloat-breakdown` separa heap, TOAST e índices
  exactamente por eso.
- **Extensiones.** Mucho de lo que en Oracle viene integrado aquí es un módulo
  instalable: `pg_stat_statements`, `pgcrypto`, `postgis`. Ver la sección 5.14.
- **Wraparound del ID de transacción.** El contador de transacciones de 32 bits hay que
  congelarlo periódicamente o el clúster rechaza las escrituras. `db-vacuum`
  informa de la edad; es el único trabajo de autovacuum que nunca debes desactivar.
- **`NULL` y la cadena vacía son distintos.** Oracle trata `''` como `NULL`; PostgreSQL
  no. Los datos migrados desde Oracle arrastran esta diferencia a menudo sin avisar.

## 14. PostgreSQL para desarrolladores {#postgresql-for-developers}

No hace falta que te conviertas en DBA para mantener sana una base PostgreSQL. Hacen
falta un modelo mental y cinco hábitos.

### 14.1 El modelo: las filas son versiones

Cada `UPDATE` y cada `DELETE` deja en su sitio la versión antigua de la fila, marcada
como muerta. Los lectores nunca bloquean a los escritores porque cada transacción ve las
versiones que estaban vigentes cuando empezó. De ahí salen dos cosas, y casi todo en esta
aplicación trata de una de ellas:

- **Las filas muertas se acumulan** hasta que autovacuum las retira. Mientras tanto
  ocupan espacio y ralentizan los escaneos. Eso es el *bloat*.
- **Vacuum solo puede retirar versiones que ninguna transacción en curso pueda ver
  todavía.** Una transacción abierta y olvidada congela la limpieza de toda la base.

`ANALYZE` es la otra mitad: vuelve a muestrear los datos para que el planificador sepa
cuántas filas va a encontrar una condición. Las estadísticas caducadas son la causa
habitual de «esta consulta iba rápida la semana pasada».

### 14.2 Cuando alguien dice «la base va lenta»

1. **Vista Sesiones.** Cuántas conexiones, cuántas ejecutan de verdad, si algo está
   bloqueado. Una consulta bloqueada es un problema de bloqueos, no de rendimiento.
2. **`db-health`.** Una pantalla: conexiones, acierto de caché, transacciones largas,
   bloat, bloqueos.
3. **`db-stat-statements`.** Las 25 consultas que consumieron más tiempo. Casi siempre
   la respuesta está aquí, y suele ser una consulta llamada muchas más veces de lo que
   nadie esperaba.
4. **EXPLAIN de la consulta** en la vista SQL. Busca `Seq Scan` sobre una tabla grande, y
   una estimación de filas radicalmente distinta de la real - eso segundo significa que
   las estadísticas están caducadas.
5. **`db-index-advisor`.** Índices que faltan y claves foráneas sin indexar.

### 14.3 Cinco hábitos

1. **No dejes transacciones abiertas.** Una sesión de ORM que abrió una transacción y se
   fue a comer es lo más dañino que un desarrollador puede hacerle a una base PostgreSQL.
   `db-idle-in-tx` las encuentra.
2. **Indexa tus claves foráneas.** PostgreSQL no crea el índice por ti, y sin él cada
   borrado o actualización en la tabla padre escanea la hija.
3. **`ANALYZE` después de cargas masivas.** Un millón de filas insertadas sin
   estadísticas detrás producirá planes malos hasta que autovacuum se ocupe.
4. **Por lotes, pero no demasiado grandes.** Una sola transacción que borra diez millones
   de filas retiene bloqueos, genera WAL y deja diez millones de versiones muertas atrás.
   Itera en lotes de unos pocos miles y confirma.
5. **Usa un pool de conexiones.** Abrir una conexión por petición cuesta un fork de
   proceso. Lee `db-connections` antes de subir `max_connections`.

### 14.4 Las palabras que usa esta aplicación {#the-words-this-app-uses}

- **Bloat** - espacio ocupado por versiones muertas de fila y nunca devuelto al sistema.
- **Tupla muerta** - una versión antigua de fila esperando a vacuum. «Tupla» aquí
  significa «fila».
- **Autovacuum** - el proceso en segundo plano que retira tuplas muertas y refresca las
  estadísticas. Se dispara por tabla, sobre umbrales que puedes ver en la pestaña Asesor.
- **VACUUM frente a VACUUM FULL** - el `VACUUM` normal marca el espacio como reutilizable
  y nunca bloquea a los lectores. `VACUUM FULL` reescribe la tabla entera, devuelve
  espacio al sistema y retiene un bloqueo que detiene *todo* sobre esa tabla mientras
  dura. `REINDEX CONCURRENTLY` y `pg_repack` son las alternativas en línea.
- **WAL** - el registro de escritura anticipada. Cada cambio se escribe allí primero; la
  replicación y la recuperación tras caída lo leen.
- **Checkpoint** - el volcado periódico de páginas sucias a los archivos de datos.
  Demasiado frecuente significa picos de escritura; `db-checkpoints` informa de la
  proporción entre los programados y los solicitados.
- **search_path** - la lista de esquemas donde se busca un nombre sin cualificar. Decide
  dónde aterriza `CREATE TABLE x` y a qué función resuelve `f()`.
- **Rol** - el concepto único detrás de usuarios y grupos. Un rol con `LOGIN` es lo que
  otras bases llaman un usuario.
- **Los huecos en las secuencias son normales.** Las secuencias no son transaccionales:
  un insert revertido igualmente consumió su número.

## Informar de un error

**Ajustes -> Informar de un error** crea el informe por ti: un ID de error, la
versión, el commit, tu sistema operativo y las últimas cuarenta líneas del
registro de errores. Tú rellenas tres secciones - qué hiciste, qué esperabas,
qué ocurrió en su lugar - y pulsas uno de los tres botones.

**Abrir el correo** entrega el texto a tu propio programa de correo, dirigido a
**dbapguin@gmail.com**. La aplicación no envía nada por sí misma y nunca lo ha
hecho; primero ves el texto exacto y tú pulsas Enviar. **Copiar informe** y
**Guardar en archivo** están ahí por si prefieres adjuntarlo o pegarlo en otro
sitio.

**Qué se elimina antes de que nada salga de tu equipo**: contraseñas, nombres
de usuario, nombres de base de datos, nombres de host, URI de conexión,
direcciones IP y la ruta de tu carpeta personal. El diálogo enumera lo que ha
quitado, para que puedas verlo en vez de confiar sin más.

Cita el ID del error (`DBAP-261011-K4QP` y similares) en cualquier seguimiento:
es como se distingue un informe de otro.

## 15. Cómo obtener ayuda

- Documentación interna: este archivo más `docs/TECHNICAL.md` y `ARCHITECTURE.md`
- Procedencia de la compilación: **Ayuda -> Acerca de DBApguin** muestra versión, commit de git, marca de tiempo de compilación y el Python con el que se compiló. Inclúyelo en los informes de fallo.
- Registro de auditoría: `<DBCHECKS_HOME>/audit/audit-YYYY-MM.jsonl` es un archivo JSON Lines. Ábrelo con cualquier editor de texto, o con `jq` para analizarlo.
- Registro de errores: `<DBCHECKS_HOME>/dbchecks_error.log` para las trazas más recientes del lado del servidor. Las contraseñas se redactan antes de escribir. Rota a 1 MiB y guarda tres copias anteriores, y cada ejecución que registre algo escribe primero una cabecera - así que las entradas posteriores a la última línea `=== DBApguin ... started ...` son las de tu sesión actual. Las de más arriba pueden tener semanas.

Para soporte comercial, contacta con el proveedor incluyendo:
1. Versión de la compilación (desde Ayuda -> Acerca de DBApguin).
2. Clave de proveedor (desde los detalles de la conexión).
3. Versión del servidor PostgreSQL.
4. Una copia de la sección relevante del registro de auditoría + el registro de errores si está disponible.
