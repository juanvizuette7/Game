# Juego 3D: Blender, React, Three.js y MongoDB

Documentación de mantenimiento del estado actual del repositorio. Última revisión: 11 de noviembre de 2026.

El proyecto permite recorrer tres mundos, recoger monedas y evitar enemigos. Blender prepara los modelos; React presenta el acceso y la carga; Three.js dibuja la escena; Cannon resuelve las colisiones; Express y MongoDB conservan usuarios, perfiles y registros del escenario.

## Cómo consultar esta documentación

1. Sigue **Instalación y base de datos** para ejecutar una copia nueva.
2. Sigue **Del mundo de Blender al juego** para modificar escenarios.
3. Consulta **Clases y responsabilidades** para localizar el módulo que debes cambiar.
4. Abre el [catálogo completo](docs/INDICE.md) para encontrar cada archivo, función y método.
5. Abre la referencia de ese archivo dentro de [docs/codigo](docs/codigo): contiene el código y la explicación de **cada línea**, incluidas las líneas estructurales, configuraciones y datos.
6. Revisa **Limitaciones conocidas** antes de modificar progreso, multijugador o scripts históricos.

Los comentarios del código propio están en español y son breves. Las explicaciones largas se mantienen aquí y en las referencias por archivo. No se insertan comentarios dentro de JSON ni dentro de textos JSX/plantillas que cambiarían su contenido; esas líneas se explican en las referencias. Los GLB, archivos Blender, imágenes, audio, fuentes, bibliotecas y decodificadores de terceros no se modifican para añadir comentarios: son recursos, no código propio de la aplicación.

## Contenido

- [1. Tecnologías y responsabilidades](#1-tecnologías-y-responsabilidades)
- [2. Organización del repositorio](#2-organización-del-repositorio)
- [3. Instalación y base de datos](#3-instalación-y-base-de-datos)
- [4. Variables de entorno](#4-variables-de-entorno)
- [5. Ejecución y comprobación inicial](#5-ejecución-y-comprobación-inicial)
- [6. Datos, autenticación y API](#6-datos-autenticación-y-api)
- [7. Del mundo de Blender al juego](#7-del-mundo-de-blender-al-juego)
- [8. Recursos y texturas](#8-recursos-y-texturas)
- [9. Físicas, colisiones y escalones](#9-físicas-colisiones-y-escalones)
- [10. Flujo completo de ejecución](#10-flujo-completo-de-ejecución)
- [11. Cámaras, entrada, interfaz, sonido y VR](#11-cámaras-entrada-interfaz-sonido-y-vr)
- [12. Optimización y propiedad de recursos](#12-optimización-y-propiedad-de-recursos)
- [13. Clases y responsabilidades](#13-clases-y-responsabilidades)
- [14. Cambios realizados](#14-cambios-realizados)
- [15. Pruebas y validación](#15-pruebas-y-validación)
- [16. Limitaciones conocidas](#16-limitaciones-conocidas)
- [17. Guía para cambios posteriores](#17-guía-para-cambios-posteriores)
- [18. Documentación, Git y publicación](#18-documentación-git-y-publicación)
- [19. Referencias técnicas](#19-referencias-técnicas)

## 1. Tecnologías y responsabilidades

Las versiones indicadas son las declaradas en los manifiestos no una promesa de compatibilidad con cualquier versión posterior. Los archivos package-lock fijan la instalación reproducible.

| Tecnología | Versión declarada | Para qué se utiliza |
| --- | --- | --- |
| Blender | Archivos exportados con glTF Blender I/O 5.1.1 | Modelar mundos, materiales, UV, posiciones y exportaciones individuales. No ejecuta la física del navegador. |
| Python y bpy | Python integrado en Blender | Recorrer mallas y ejecutar el exportador glTF. El archivo Text_3_cero.txt es Python. |
| JavaScript | Módulos del navegador y CommonJS del servidor | Implementar reglas del juego, API y utilidades. |
| React / React DOM | 19 | Formulario de acceso, estado del usuario, canvas y barra de carga. No mueve los cuerpos físicos. |
| Vite | 6 | Servidor de desarrollo, transformación de módulos y compilación a game-project/dist. No es el servidor de MongoDB. |
| Three.js | 0.175 | Escena, geometrías, materiales, cámara, iluminación, animaciones, cargadores y renderizador WebGL. |
| WebGL | API del navegador | Envía las órdenes gráficas a la GPU. Dibujar un objeto no crea su colisión. |
| cannon-es | 0.20 | Cuerpos, gravedad, contactos, rayos físicos y resolución de colisiones. No dibuja modelos. |
| cannon-es-debugger | 1 | Representaciones de depuración; su construcción se condiciona al modo debug. |
| Express | 5 | Rutas HTTP, lectura de JSON y respuestas de la API. |
| Node.js | Ver manifiestos; validado con Node 24 | Ejecuta el backend, herramientas npm y pruebas. No sustituye a Blender para importar bpy. |
| MongoDB | Instalación local o Atlas | Guarda documentos de usuarios, perfiles y bloques. No sirve los GLB al navegador. |
| Mongoose | 8 | Conecta con MongoDB y define esquemas y consultas. |
| bcrypt | 6 | Genera y verifica el hash de las contraseñas. |
| jsonwebtoken | 9 | Firma y verifica tokens de acceso. |
| dotenv | 16 | Lee backend/.env cuando arranca el servidor. |
| cors | 2 | Permite solicitudes de desarrollo desde el origen del frontend. |
| Socket.io / socket.io-client | 4 | Base para multijugador opcional; actualmente no está terminado ni activado. |
| Howler | 2 | Reproduce efectos y gestiona el contexto de audio. |
| Web Audio | API del navegador | Decodifica y reproduce el ambiente en AmbientSound. |
| GSAP | 3 | Anima la retirada de obstáculos. |
| lil-gui | 0.20 | Panel de depuración al abrir la aplicación con #debug. |
| ESLint y complementos React | 9 / según manifiesto | Análisis estático del JavaScript y JSX. |
| node:test / assert | Incluidos en Node | Pruebas físicas, de carga, optimización y cámaras. |
| node-fetch | 3 | Peticiones desde el script CommonJS de sincronización mediante importación dinámica. |

body-parser figura como dependencia histórica, pero app.js utiliza express.json(). El renderizador y Cannon representan el mismo mundo con estructuras distintas: mallas para ver y cuerpos para colisionar.

## 2. Organización del repositorio

~~~text
/
├── README.md                     Guía principal
├── backend/
│   ├── app.js                    Entrada real del servidor
│   ├── .env-example              Plantilla sin credenciales
│   ├── controllers/              Lógica de rutas
│   ├── middlewares/              Verificación JWT
│   ├── models/                   Esquemas Mongoose
│   ├── routes/                   Endpoints HTTP
│   ├── data/                     Posiciones y catálogos intermedios
│   ├── scripts/                  Exportador Blender y utilidades históricas
│   └── seed.js                   Semilla destructiva de demostración
├── game-project/
│   ├── index.html                Entrada HTML utilizada por Vite
│   ├── vite.config.js
│   ├── eslint.config.js
│   ├── public/
│   │   ├── PrimerMundo.blend
│   │   ├── SegundoMundo.blend
│   │   ├── TercerMundo.blend
│   │   ├── models/               GLB/GLTF y archivos asociados
│   │   ├── textures/             Imágenes externas y cubemap
│   │   ├── sounds/
│   │   ├── data/                 Respaldo local combinado y copias por nivel
│   │   └── config/               Listas históricas de colisión
│   ├── src/
│   │   ├── main.jsx              Montaje de React
│   │   ├── App.jsx               Acceso y carga
│   │   ├── Experience/           Núcleo del juego
│   │   │   ├── Utils/            Físicas, recursos y utilidades
│   │   │   └── World/            Entidades y reglas del mundo
│   │   ├── controls/            Ajustes y entrada táctil
│   │   ├── integrations/         WebXR
│   │   └── network/              Cliente multijugador opcional
│   └── tests/                    Pruebas automatizadas
├── docs/
│   ├── INDICE.md                 Catálogo de todos los archivos documentados
│   ├── codigo/                   Referencias de cada línea y método
│   ├── modulos.json              Descripciones mantenidas de módulos y métodos
│   └── cobertura.json            Rutas, líneas y huellas de documentación
└── scripts/document-code.mjs     Generación y comprobación de documentación
~~~

src/index.html es una copia histórica, no la entrada utilizada por Vite. App.css e index.css conservan estilos de plantilla y no se importan en el flujo actual. El estilo general activo es src/style.css y la carga utiliza src/styles/loader.css.

public/models/toycar es la carpeta utilizada por el catálogo activo para los tres niveles. toycar2 contiene copias históricas; no se selecciona automáticamente por tratarse del segundo nivel. Los archivos .blend1 son copias de seguridad de Blender y no intervienen en la ejecución.

## 3. Instalación y base de datos

### 3.1. Requisitos y preparación

Instala Node.js con npm, Git, MongoDB local o acceso a Atlas y Blender si vas a editar escenarios. Para regenerar comentarios del exportador desde Windows se utiliza también el lanzador Python con Python 3.12. Para ejecutar el juego no hace falta instalar Blender ni ese Python.

Los comandos siguientes están escritos para PowerShell. npm.cmd evita el bloqueo del archivo npm.ps1 por la política de ejecución de Windows. En otros sistemas puedes utilizar npm.

~~~powershell
git clone https://github.com/PATINO-0/Game.git
Set-Location '.\Game'
node --version
npm.cmd --version
npm.cmd --prefix backend ci
npm.cmd --prefix game-project ci
~~~

npm ci utiliza package-lock y reconstruye node_modules. Después de cambiar deliberadamente dependencias, utiliza npm install en la aplicación correspondiente y publica también su package-lock.

### 3.2. Opción A: MongoDB como servicio de Windows

Tras instalar MongoDB Community como servicio:

~~~powershell
Get-Service -Name MongoDB
Start-Service -Name MongoDB
~~~

Start-Service puede requerir una terminal con privilegios de administrador. El nombre del servicio debe coincidir con el de tu instalación. Si ya está en ejecución, no necesitas iniciarlo otra vez.

### 3.3. Opción B: MongoDB sin servicio

Ejecuta mongod en una terminal dedicada. Mantén los datos fuera del repositorio:

~~~powershell
$dbDir = Join-Path $env:LOCALAPPDATA 'GameMongoData'
New-Item -ItemType Directory -Path $dbDir -Force | Out-Null
mongod --dbpath "$dbDir" --bind_ip 127.0.0.1 --port 27017
~~~

Otra alternativa, si ya utilizas Docker, es:

~~~powershell
docker run --name game-mongo --publish 127.0.0.1:27017:27017 --volume game-mongo-data:/data/db --detach mongo:8.0
~~~

El contenedor se crea una vez. Para volver a iniciarlo utiliza docker start game-mongo.

### 3.4. Opción C: MongoDB Atlas

1. Crea o utiliza un despliegue de Atlas.
2. Crea un usuario de base de datos con acceso a la base del juego.
3. Autoriza la dirección de red desde la que se conecta el backend.
4. Copia la URI de conexión proporcionada por Atlas.
5. Sustituye usuario, contraseña y nombre de base en backend/.env.
6. Si la contraseña contiene caracteres reservados, codifícalos para una URI.
7. Mantén la URI completa solo en el servidor.

Ejemplo sin credenciales reales:

~~~env
MONGO_URI=mongodb+srv://USUARIO:CLAVE_CODIFICADA@HOST/threejs_blocks
~~~

La conexión local documentada utiliza mongodb://127.0.0.1:27017/threejs_blocks. El nombre después del puerto selecciona la base; la URI SRV y la URI estándar tienen formatos distintos. Consulta los [formatos oficiales de conexión](https://www.mongodb.com/docs/manual/reference/connection-string-formats/).

### 3.5. Verificar la base

Si tienes mongosh:

~~~powershell
mongosh "mongodb://127.0.0.1:27017/threejs_blocks" --eval "db.runCommand({ ping: 1 })"
~~~

El resultado esperado contiene ok: 1. También puedes conectarte con MongoDB Compass usando la misma URI local. Una instalación vacía no contiene los bloques hasta importarlos.

## 4. Variables de entorno

### 4.1. Backend

Copia la plantilla únicamente si todavía no tienes configuración local:

~~~powershell
if (-not (Test-Path -LiteralPath '.\backend\.env')) {
    Copy-Item -LiteralPath '.\backend\.env-example' -Destination '.\backend\.env'
}
~~~

Configura:

~~~env
PORT=3001
MONGO_URI=mongodb://127.0.0.1:27017/threejs_blocks
API_URL=http://localhost:3001/api/blocks/batch
JWT_SECRET=REEMPLAZAR_POR_UN_SECRETO_LOCAL
JWT_EXPIRES_IN=7d
~~~

| Variable | Consumidor | Significado |
| --- | --- | --- |
| PORT | app.js | Puerto HTTP y Socket.io; por defecto 3001. |
| MONGO_URI | app.js; seed.js | Conexión de Mongoose con la base. |
| API_URL | sync_blocks.js | URL completa del endpoint de inserción por lotes. |
| JWT_SECRET | authController y authMiddleware | Secreto compartido para firmar y comprobar JWT. |
| JWT_EXPIRES_IN | authController | Duración del token; por defecto 7d. |

Puedes generar un secreto local con:

~~~powershell
node -e "console.log(require('node:crypto').randomBytes(32).toString('hex'))"
~~~

Copia el resultado en tu archivo .env. No lo pegues en el catálogo frontend, el README ni los archivos que publiques.

### 4.2. Frontend

~~~powershell
if (-not (Test-Path -LiteralPath '.\game-project\.env')) {
    Copy-Item -LiteralPath '.\game-project\.env-example' -Destination '.\game-project\.env'
}
~~~

Contenido de ejemplo:

~~~env
VITE_API_URL=http://localhost:3001
VITE_BACKEND_URL=http://localhost:3001
VITE_ENEMIES_COUNT=3
~~~

VITE_API_URL es el origen del backend, sin añadir /api/blocks. ToyCarLoader conserva VITE_BACKEND_URL como alternativa; si configuras las dos, utiliza el mismo origen. El número de enemigos debe ser un entero positivo; el código vuelve a tres si recibe cero o un valor no válido.

Las variables VITE_ quedan disponibles para el navegador y son públicas. No coloques allí MONGO_URI ni JWT_SECRET. Reinicia Vite después de cambiar su entorno y vuelve a compilar si estás sirviendo dist. Este comportamiento corresponde a la [documentación de variables de Vite](https://vite.dev/guide/env-and-mode).

### 4.3. Jugar desde otra máquina

El script dev ya utiliza --host. Configura VITE_API_URL con la IP LAN del servidor, por ejemplo http://192.168.1.100:3001. localhost en el teléfono se refiere al propio teléfono. Comprueba acceso de red y permisos del firewall. VR inmersivo requiere además un contexto seguro compatible con WebXR.

## 5. Ejecución y comprobación inicial

### 5.1. Terminal del backend

Desde la raíz:

~~~powershell
Set-Location '.\backend'
node app.js
~~~

La entrada real es app.js. package.json conserva main: index.js, pero ese campo no cambia el comando de arranque. No hay un script npm start configurado actualmente.

El servidor debe informar que escucha en 3001 y que se conectó a MongoDB. Puede comenzar a escuchar antes de que termine la conexión a la base.

### 5.2. Terminal del frontend

Desde la raíz, en otra terminal:

~~~powershell
Set-Location '.\game-project'
npm.cmd run dev
~~~

Abre http://localhost:5173. Si el puerto está ocupado, Vite puede elegir otro; usa la dirección que muestra la terminal.

### 5.3. Comprobar la API

~~~powershell
Invoke-RestMethod 'http://localhost:3001/api/blocks/ping'
Invoke-RestMethod 'http://localhost:3001/api/blocks?level=1'
~~~

ping responde con message: pong; esa ruta por sí sola no demuestra conexión con MongoDB. La segunda consulta debe responder con un arreglo. En una base vacía puede devolver [].

### 5.4. Primer acceso

1. Regístrate con correo y contraseña.
2. App envía el correo también como username.
3. El servidor crea usuario y perfil y devuelve token y usuario.
4. App guarda esos datos en memoria, mediante estado React y window.auth.
5. Se crea Experience y comienza la carga del primer nivel.
6. El botón Iniciar juego activa cámara, cronómetro y enemigos.

El modo sin conexión aparece tras un error de red de acceso o registro. Un error HTTP de credenciales no activa esa alternativa. Sin token no se envía progreso, pero el escenario puede cargarse desde el JSON local.

### 5.5. Compilar y servir la compilación

Desde game-project:

~~~powershell
npm.cmd run build
npm.cmd run preview
~~~

Vite genera dist. preview permite comprobar esa compilación; no sustituye el backend ni es una configuración de despliegue de producción. Actualmente queda un aviso de tamaño del bundle, aunque la compilación termina correctamente.

## 6. Datos, autenticación y API

### 6.1. Qué se guarda y dónde

| Información | Ubicación y responsable |
| --- | --- |
| Geometría, materiales y clips exportados | Archivos GLB/GLTF en public/models. |
| Imágenes externas | public/textures. |
| Audio | public/sounds. |
| Nombre, posición, nivel y rol del escenario | Colección de Block y JSON exportados. |
| Usuario y hash de contraseña | User. |
| Niveles, monedas y mejores tiempos | PlayerProfile, con las limitaciones de sincronización descritas abajo. |
| Usuario y JWT activos | Estado React y window.auth; no se restauran al recargar. |
| Modo de cámara | localStorage, clave cameraMode. |
| Cinco mejores tiempos locales | localStorage, clave bestTimes. |

Mongoose utiliza sus nombres de colección convencionales para los modelos; confirma los nombres en tu base si los administras desde otra herramienta.

### 6.2. Estructura de un bloque

~~~json
{
  "name": "mi_edificio_lev1",
  "x": 10,
  "y": 0,
  "z": -5,
  "level": 1,
  "role": "default"
}
~~~

name conecta el registro con sources.js. level es obligatorio y por defecto vale 1. role admite default y finalPrize. El esquema actual no exige name ni cada coordenada, por lo que valida esos campos antes de importar.

Un bloque no contiene una textura, una malla ni un cuerpo Cannon serializado. Los objetos estáticos conservan su transformación dentro del GLB; las monedas usan las coordenadas del registro para ubicar su pivote.

### 6.3. Endpoints de bloques

| Método y ruta | Entrada | Salida |
| --- | --- | --- |
| GET /api/blocks/ping | Ninguna | Comprobación del servidor. |
| GET /api/blocks?level=1 | Nivel por query; por defecto 1 | Arreglo con name, x, y, z, level y role, sin _id. |
| POST /api/blocks | Un bloque JSON | Registro creado y mensaje. |
| POST /api/blocks/batch | Arreglo de bloques | Mensaje y cantidad insertada. |

El controlador de lotes llama a insertMany: repetirlo añade documentos; no reemplaza objetos por nombre ni limpia un nivel. Las rutas de bloques no requieren JWT actualmente.

Ejemplo de inserción individual, que escribe en la base:

~~~powershell
$block = @{
    name = 'mi_edificio_lev1'
    x = 10
    y = 0
    z = -5
    level = 1
    role = 'default'
} | ConvertTo-Json
Invoke-RestMethod 'http://localhost:3001/api/blocks' -Method Post -ContentType 'application/json' -Body $block
~~~

No ejecutes el ejemplo con un nombre inexistente en el catálogo si esperas ver un modelo nuevo.

### 6.4. Autenticación

| Ruta | Cuerpo | Resultado |
| --- | --- | --- |
| POST /api/auth/register | username, email, password | Crea User y PlayerProfile; devuelve token y usuario. |
| POST /api/auth/login | usernameOrEmail, password | Comprueba contraseña y devuelve token y usuario. |

bcrypt.hash usa un factor de trabajo 10. El JWT incluye sub, username y role. El middleware requireAuth espera Authorization: Bearer TOKEN, verifica el secreto y consulta que el usuario siga existiendo.

### 6.5. Perfil

| Ruta protegida | Función |
| --- | --- |
| GET /api/profile/me | Devuelve el perfil y datos básicos del usuario. |
| PUT /api/profile/progress | Espera level, timeSeconds y coinsCollected. Actualiza nivel, acumula monedas y conserva el mejor tiempo del nivel. |

El acumulado de monedas no es idempotente: repetir la misma actualización suma otra vez. La selección actual de cámara se guarda localmente; el campo settings.cameraMode del perfil no está conectado al selector.

## 7. Del mundo de Blender al juego

### 7.1. Orden completo

~~~mermaid
flowchart LR
    A["Editar el .blend"] --> B["Seleccionar mallas y materiales"]
    B --> C["Ejecutar exportador Python"]
    C --> D["Un GLB por malla y JSON de posiciones"]
    D --> E["Copiar JSON a backend/data"]
    E --> F["Reconstruir respaldo y catálogo"]
    F --> G["Importar registros a MongoDB si se usa API"]
    G --> H["Cargar el nivel y sus recursos"]
    H --> I["Crear colisiones y lotes visuales"]
    I --> J["Jugar y validar"]
~~~

El juego no importa directamente .blend. Exportar un único GLB de toda la escena es posible, pero no coincide con el cargador actual, que relaciona registros y modelos individuales.

### 7.2. Preparación en Blender

1. Abre PrimerMundo.blend, SegundoMundo.blend o TercerMundo.blend.
2. Trabaja en modo Objeto.
3. Revisa ubicación, orientación y escala de las mallas.
4. Comprueba que las imágenes de los materiales existan y estén guardadas.
5. Comprueba UVs y conecta las imágenes a un material exportable.
6. Para color utiliza normalmente Image Texture conectado a Base Color del Principled BSDF.
7. Hornea a imágenes los materiales procedurales que no soporte glTF.
8. Selecciona todas las mallas que pertenecen al nivel.
9. Guarda el .blend con selección y objeto activo si vas a exportar en segundo plano.

El exportador existente solo procesa objetos MESH. Omite cámaras, luces y armatures; no sirve para exportar el rig de un personaje. Los personajes animados deben conservar su esqueleto y clips mediante un flujo de exportación apropiado.

### 7.3. Configurar el exportador existente

Abre backend/scripts/Text_3_cero.txt. Es código Python aunque termine en .txt. Cambia las variables de configuración antes de ejecutarlo:

~~~python
nivel = 1
export_path = r"F:/RUTA/DE/TU/COPIA/Game/game-project/public/models/toycar"
json_path = os.path.join(export_path, f"toy_car_blocks{nivel}.json")
~~~

La ruta actual del archivo es una ruta histórica D:/.../toycar2. Debes sustituirla. Para otros mundos cambia nivel a 2 o 3 y abre el .blend correspondiente.

El script:

1. Obtiene el grafo evaluado de la escena.
2. Recorre la selección inicial.
3. Omite los objetos que no son mallas.
4. Lee su matriz mundial y extrae su posición.
5. Normaliza el nombre a minúsculas y sustituye espacios por guiones bajos.
6. Retira las apariciones del sufijo del nivel actual y añade ese sufijo al final.
7. Construye el registro de coordenadas convertido.
8. Selecciona únicamente esa malla.
9. Exporta su GLB.
10. Guarda todos los registros en un JSON.

El script modifica nombres dentro de la sesión de Blender y sobrescribe archivos coincidentes. No guarda automáticamente el .blend.

### 7.4. Nombres y roles

Debe coincidir el nombre del registro con el campo name del recurso del catálogo. La extensión .glb pertenece a path, no a name.

El script conserva sufijos de otros niveles. Por eso hay nombres válidos como plane_lev1_lev2. No los simplifiques sin actualizar los registros y el catálogo. El portal del segundo nivel busca específicamente baked_lev1_lev2.

Las monedas deben empezar con coin. El rol finalPrize por sí solo no convierte una estructura normal en moneda: la rama del cargador depende también de ese prefijo. El exportador asigna default a todos los registros; cambia el rol de la moneda final explícitamente si utilizas esa ruta de juego.

### 7.5. Coordenadas

~~~python
loc = obj.evaluated_get(depsgraph).matrix_world.to_translation()
pos = {
    "name": obj.name,
    "x": loc.x,
    "y": loc.z,
    "z": -loc.y,
    "level": nivel,
    "role": "default"
}
~~~

| Blender | Three.js / JSON |
| --- | --- |
| X | X |
| Z, altura | Y |
| Y | -Z |

El exportador GLB también convierte a la convención Y arriba. No vuelvas a rotar el modelo manualmente para repetir esa conversión.

El GLB de una estructura ya conserva su posición mundial. ToyCarLoader no vuelve a sumarle block.x/y/z. Prize centra la parte visible de la moneda y coloca el pivote con el JSON. Mantén las monedas como mallas sencillas por archivo; revisa su centrado si cambias la jerarquía.

### 7.6. Operador de exportación

Esta llamada explicita los parámetros compatibles con el flujo del proyecto. Sustituye la llamada dentro del bucle si necesitas configurarlos explícitamente:

~~~python
bpy.ops.export_scene.gltf(
    filepath=filepath,
    use_selection=True,
    export_format='GLB',
    export_yup=True,
    export_texcoords=True,
    export_normals=True,
    export_materials='EXPORT',
    export_image_format='AUTO',
    export_apply=False,
    export_draco_mesh_compression_enable=False
)
~~~

GLB empaqueta geometría e imágenes utilizadas por materiales compatibles. export_apply controla la aplicación de modificadores, no el traslado del objeto al origen. Si cambias ese parámetro para una malla estática, revisa el resultado, especialmente con shape keys o esqueletos. Consulta la [API oficial del exportador](https://docs.blender.org/api/5.0/bpy.ops.export_scene.html).

No actives Draco ni texturas KTX2 en este flujo sin configurar sus cargadores. Tener una carpeta public/draco no registra automáticamente un decodificador en Resources.

### 7.7. Ejecutar desde Blender

En la pestaña Scripting:

1. Abre Text_3_cero.txt.
2. Corrige configuración y ruta.
3. Selecciona las mallas en la vista 3D.
4. Pulsa Run Script o Alt+P con el cursor sobre el editor de texto.
5. Revisa los mensajes y archivos generados.

### 7.8. Ejecutar desde PowerShell

Desde la raíz del repositorio y con el .blend guardado con selección válida:

~~~powershell
$blender = (Get-Command blender.exe -ErrorAction Stop).Source
& $blender --background '.\game-project\public\PrimerMundo.blend' --python-exit-code 1 --python '.\backend\scripts\Text_3_cero.txt'
~~~

Si Blender no está en PATH, asigna a $blender la ruta absoluta de su ejecutable instalado. No uses el Python normal de Windows para ejecutar bpy: el comando anterior utiliza el Python integrado de Blender.

Para el segundo o tercer nivel cambia tanto el .blend del comando como la variable nivel del script. Espera GLB individuales y toy_car_blocksN.json en la carpeta de salida.

### 7.9. Copiar los registros exportados

Ejemplo para el primer nivel:

~~~powershell
Copy-Item -LiteralPath '.\game-project\public\models\toycar\toy_car_blocks1.json' -Destination '.\backend\data\toy_car_blocks1.json'
~~~

Repite con 2 y 3 cuando los exportes. Los datos actuales tienen 54 registros en el nivel 1, 54 en el nivel 2 y 38 en el nivel 3: 146 en total. Hay 116 objetos de ambiente y 30 registros con prefijo coin en esos archivos; el total visible puede ser menor si falta un recurso o existe un nombre duplicado.

### 7.10. Reconstruir el respaldo local

El juego lee public/data/toy_car_blocks.json, no las copias individuales por nivel. Desde la raíz:

~~~powershell
$blocks = @(foreach ($level in 1..3) {
    $levelBlocks = Get-Content -LiteralPath ".\backend\data\toy_car_blocks$level.json" -Raw | ConvertFrom-Json
    foreach ($block in $levelBlocks) { $block }
})
$utf8 = New-Object System.Text.UTF8Encoding($false)
$localPath = Join-Path (Get-Location).Path 'game-project/public/data/toy_car_blocks.json'
[System.IO.File]::WriteAllText($localPath, (ConvertTo-Json -InputObject $blocks -Depth 10), $utf8)
~~~

La iteración interior es necesaria para obtener un arreglo plano de objetos en PowerShell. Un arreglo con tres arreglos no tiene el formato esperado.

### 7.11. Registrar modelos en el catálogo

Una entrada en game-project/src/Experience/sources.js:

~~~javascript
{
    name: 'mi_edificio_lev1',
    type: 'gltfModel',
    path: '/models/toycar/mi_edificio_lev1.glb'
}
~~~

No incluyas public en la URL. Conserva también robotModel, foxModel, enemyModel y environmentMapTexture.

Para reconstruir las entradas de los mundos desde los tres JSON manteniendo las entradas comunes, puedes ejecutar desde la raíz:

~~~powershell
@'
import fs from 'node:fs';
import sources from './game-project/src/Experience/sources.js';
const files = [1, 2, 3].map(level => 'backend/data/toy_car_blocks' + level + '.json');
const blocks = files.flatMap(file => JSON.parse(fs.readFileSync(file, 'utf8')));
const core = sources.filter(source => !/_lev\d+/.test(source.name));
const names = [...new Set(blocks.map(block => block.name))];
const worlds = names.map(name => {
  const path = '/models/toycar/' + name + '.glb';
  if (!fs.existsSync('game-project/public' + path)) throw new Error('Falta: ' + path);
  return { name, type: 'gltfModel', path };
});
fs.writeFileSync('game-project/src/Experience/sources.js',
  'export default ' + JSON.stringify([...core, ...worlds], null, 2) + '\n', 'utf8');
'@ | node --input-type=module
~~~

Este comando reemplaza el catálogo: revisa git diff después y regenera sus comentarios y documentación. No lo ejecutes hasta que los tres JSON y los GLB sean coherentes.

El generador histórico backend/scripts/generate_sources.js no hace este proceso completo: lee una ruta relativa antigua, escribe sources_3.js en backend/data y genera URLs toycar aunque la entrada apunta a toycar2. Los archivos backend/data/sources_N.js son intermedios y no están importados por el frontend.

### 7.12. Enviar los registros a MongoDB

Si vas a utilizar la API:

1. Inicia MongoDB.
2. Inicia node app.js desde backend.
3. Configura API_URL en backend/.env.
4. Revisa filesToSend en sync_blocks.js.
5. Ejecuta desde backend:

~~~powershell
node scripts/sync_blocks.js
~~~

filesToSend contiene actualmente solo toy_car_blocks1.json. Para importar los tres, configura deliberadamente:

~~~javascript
const filesToSend = [
    'toy_car_blocks1.json',
    'toy_car_blocks2.json',
    'toy_car_blocks3.json'
];
~~~

Esto escribe datos nuevos. El servidor usa insertMany y puede duplicar los existentes. El script no comprueba res.ok antes de anunciar su resultado; confirma siempre con GET /api/blocks?level=N. No utilices seed.js como importador de mundos: elimina los bloques y carga solo dos posiciones de demostración.

### 7.13. Física después de exportar

No hay un comando que copie las físicas de Blender al navegador. Al cargar los registros y modelos, ToyCarLoader invoca createStaticBodiesFromModel para los objetos del ambiente y los registra en Physics.world.

No necesitas añadir nombres a precisePhysicsModels.json. El comando histórico node scripts/generatePhysicsModels.js solo genera una lista intermedia del nivel 3 y no participa en la decisión actual de colisión.

### 7.14. Comprobación final del mundo

1. Revisa nombres y archivos del catálogo.
2. Consulta la API o comprueba el respaldo combinado.
3. Abre el nivel en el navegador.
4. Comprueba texturas y escala.
5. Camina contra paredes y alrededor de objetos.
6. Comprueba suelo, bordes, plataformas y salto.
7. Deja que un enemigo siga al jugador.
8. Recoge monedas y cruza el portal.
9. Repite la transición y vuelve a un nivel anterior.
10. Ejecuta npm test y npm run build.

## 8. Recursos y texturas

### 8.1. Catálogo, carga y caché

sources.js es un catálogo, no una instrucción para precargar todos los mundos. Resources:

1. Extrae el último sufijo _levN del nombre.
2. Carga inicialmente los recursos del nivel 1 y los personajes y entorno comunes.
3. Utiliza cuatro trabajadores para limitar descargas simultáneas.
4. Comparte promesas en pending y recursos terminados en items.
5. Convierte materiales del ambiente después de cargar el GLTF.
6. Notifica resource-loading, resource-progress, resource-complete y resource-error.
7. Cuando se cambia de nivel, carga los nombres de sus bloques.
8. Tras retirar el mundo anterior, libera recursos de los niveles inactivos.

La medición de navegador realizada durante la optimización pasó de 148 a 57 peticiones iniciales de modelos. Con los datos actuales se mantienen 58 entradas cargadas en los niveles 1 y 2 y 42 en el 3. Estas cifras cambiarán si cambias el catálogo o los objetos.

### 8.2. Texturas dentro del GLB

GLTFLoader crea los materiales y carga las imágenes embebidas. Prepara el material en Blender y exporta el GLB con materiales e imágenes. El optimizador del ambiente conserva mapas de color, emisivos, alfa y la paleta; no conserva los efectos físicos de transmisión ni todos los mapas PBR.

### 8.3. Texturas externas

Copia la imagen a public/textures:

~~~powershell
Copy-Item -LiteralPath 'C:\RUTA\mi_textura.jpg' -Destination '.\game-project\public\textures\mi_textura.jpg'
~~~

Dentro de ToyCarLoader._processBlocks, después de clonar el modelo, una aplicación específica puede utilizar:

~~~javascript
this._applyTextureToMeshes(
    model,
    '/textures/mi_textura.jpg',
    child => child.name === 'NombreExactoDeLaMalla'
);
~~~

El helper busca primero las mallas, carga la imagen, usa espacio de color sRGB, configura flipY=false, limita anisotropía a cuatro y permite rotación, repetición, espejo y desplazamiento UV. Necesitas UV válidas; copiar una imagen no genera UV.

### 8.4. Reglas especiales existentes

| Regla | Resultado |
| --- | --- |
| Nombre del bloque contiene baked | Sustituye materiales por MeshBasicMaterial con /textures/baked.jpg. |
| Nombre de la malla coincide con el filtro de cilindros | Aplica /textures/ima1.jpg con rotación y espejo configurados. |
| Nombre de suelo compatible y malla grande horizontal | Floor aplica el acabado del nivel. |

Estas reglas pueden sobrescribir un material correctamente exportado. Si personalizas una fachada, revisa antes si coincide con alguno de los filtros. baked.jpg se comparte dentro del cargador.

## 9. Físicas, colisiones y escalones

### 9.1. Responsables

| Módulo | Responsabilidad física |
| --- | --- |
| Physics | Mundo Cannon, gravedad, materiales, solver, pasos fijos y aparición. |
| CharacterMotor | Movimiento, frenado, apoyo, salto, escalón y teletransporte de una esfera. |
| PhysicsShapeFactory | Construcción de cuerpos estáticos a partir de las mallas. |
| ToyCarLoader | Asociación del modelo importado con sus cuerpos. |
| Robot | Intención de movimiento del jugador y relación modelo/controlador. |
| Enemy | Intención de persecución y relación modelo/controlador. |
| EnemySteering | Dirección libre hacia el objetivo, sin atravesar paredes. |
| Floor | Cuerpo de respaldo permanente. |
| BlockPrefab / Raycaster | Bloques y obstáculos dinámicos creados en la partida. |
| World | Limpieza de cuerpos y sincronización visual después del paso físico. |

### 9.2. Configuración de Physics

- Gravedad: (0, -9.82, 0).
- Broadphase: SAPBroadphase.
- Solver: 20 iteraciones y tolerancia 1e-5.
- Paso fijo: 1/120 s.
- Acumulador: permite varios pasos físicos por cuadro.
- Delta de entrada: limitado a 0.1 s para evitar una acumulación extrema tras una pausa.
- Restitución: cero en los contactos configurados.
- Fricción entre personaje y entorno: cero; el controlador aporta frenado horizontal.
- Grupos: environment=1, character=2 y obstacle=4.

Los grupos son bits para filtros, no masas. Hay obstáculos históricos que conservan el grupo por defecto; revisa los filtros si amplías sus comportamientos.

No se modifica directamente el arreglo interno de cuerpos para limpiarlo. removeLevelBodies utiliza removeBody y la marca levelObject.

### 9.3. Conversión de una malla a colisión

createStaticBodiesFromModel:

1. Actualiza matrices mundiales.
2. Recorre cada Mesh que tenga posiciones.
3. Lee geometría indexada o no indexada.
4. Calcula un centro próximo al objeto.
5. Convierte los vértices con matrixWorld y los expresa respecto a ese centro.
6. Descarta coordenadas no finitas y triángulos degenerados.
7. Une posiciones coincidentes y evita caras duplicadas dentro de cada lote.
8. Divide la geometría antes de superar los índices de 16 bits con signo de Cannon.
9. Corrige orientación de planos horizontales y transformaciones reflejadas.
10. Crea Trimesh y un Body de masa cero.
11. Registra el vínculo con el modelo y la pertenencia al nivel.

No se utilizan las coordenadas JSON para desplazar otra vez la estructura. createBoxShapeFromModel sigue disponible como utilidad, pero no es la estrategia actual del cargador del ambiente.

Los personajes usan esferas, una combinación compatible con este flujo Trimesh. Cambiar su forma o usar cajas dinámicas exige verificar las combinaciones de colisión que soporta Cannon; no todas las formas se comportan igual con Trimesh.

### 9.4. Control del jugador y enemigos

| Parámetro | Jugador | Enemigo |
| --- | --- | --- |
| Radio | 0.4 | 0.24 |
| Masa | 2 | 2 |
| Giro físico | Bloqueado | Bloqueado |
| Velocidad objetivo máxima | 6 | Según dificultad |
| Salto | Tecla espacio, con apoyo | No se utiliza como mecanismo de seguimiento |
| Recuperación de caída | Posición Y menor que -25 | Posición Y menor que -25 |

CharacterMotor guarda la dirección horizontal y la velocidad objetivo. Antes de cada paso ajusta la velocidad horizontal de forma gradual y evita empujar contra normales de pared conocidas. No aplica fuerza con la posición mundial como brazo de palanca.

Después del paso lee los contactos. Una normal suficientemente orientada hacia arriba indica apoyo; también comprueba velocidad vertical y enfriamiento del salto. El salto fija una velocidad vertical de 4.5 y un enfriamiento de 0.2 s. Robot solicita el salto al pulsar espacio, no continuamente mientras permanece pulsado.

### 9.5. Escalones y plataforma del nivel 2

La plataforma obj_lev2 tiene una superficie inferior cercana a Y=0.157. El suelo de respaldo está en Y=-0.16. El borde puede superar el radio de un enemigo pequeño y actuar como una pared aunque visualmente sea un desnivel bajo.

tryStep se ejecuta cuando hay apoyo, intención de avance y contacto lateral:

1. Proyecta una búsqueda vertical un poco por delante.
2. Exige una superficie transitable.
3. Limita el desnivel a 0.45.
4. Comprueba paso a la altura del cuerpo y de la cabeza.
5. Comprueba también espacio vertical en el destino.
6. Eleva el cuerpo de forma acotada y conserva su avance.
7. Activa un breve enfriamiento para evitar repetir la corrección sin control.

No permite subir una pared alta ni una plataforma con techo insuficiente. EnemySteering proyecta sus rayos por encima del borde que el controlador puede superar para no rodearlo como si fuera una pared alta. Si cambias stepHeight, mantén coherentes CharacterMotor y EnemySteering.

### 9.6. Dificultad y navegación

| Nivel | Velocidad objetivo | Intervalo de navegación |
| --- | ---: | ---: |
| 1 | 2.4 | 0.18 s |
| 2 | 3.2 | 0.15 s |
| 3 | 4.1 | 0.12 s |

Difficulty.js es el lugar de configuración. La velocidad objetivo no aumenta por cercanía al jugador. La velocidad real puede disminuir al frenar, girar o colisionar; es el resultado esperado de respetar los contactos.

EnemySteering combina dirección al jugador y separación de vecinos. Mide espacio libre con tres rayos por dirección candidata, conserva preferencia de giro y cambia de lado si detecta atasco. No fija Y cada cuadro ni utiliza impulsos verticales para salir de estructuras. Es navegación reactiva, no búsqueda global de caminos ni un navmesh completo.

## 10. Flujo completo de ejecución

### 10.1. Arranque

~~~mermaid
sequenceDiagram
    participant U as Usuario
    participant A as App
    participant API as Express
    participant DB as MongoDB
    participant E as Experience
    participant R as Resources
    participant W as World
    U->>A: Login o registro
    A->>API: Credenciales
    API->>DB: Consulta o creación
    DB-->>API: Documentos
    API-->>A: Usuario y JWT
    A->>E: Crear experiencia
    E->>R: Cargar recursos iniciales
    R-->>W: ready
    W->>API: GET bloques del nivel 1
    Note over W: Si falla, lee el respaldo local
    W->>R: Asegurar recursos de los bloques
    W->>W: Modelos, cuerpos, monedas y personajes
    U->>E: Iniciar juego
~~~

La respuesta de bloques puede ser un arreglo o un objeto con blocks y spawnPoint. getLevelData normaliza ambos formatos y filtra el nivel solicitado. Una API vacía también provoca el uso del respaldo local.

### 10.2. Orden de cada cuadro

1. El ciclo de escritorio emite tick o WebXR entrega el delta de VR.
2. Experience ajusta la cámara VR si corresponde.
3. World procesa entradas, animaciones, enemigos, monedas y reglas.
4. Physics consume el acumulador en pasos fijos.
5. Antes de cada paso, cada motor aplica su intención de movimiento.
6. Cannon calcula contactos e integra los cuerpos.
7. Cada motor lee apoyo y paredes del paso.
8. World copia las posiciones físicas a modelos visibles.
9. Camera actualiza el modo seleccionado.
10. Renderer dibuja una vez la escena.

El evento preStep de Cannon no se utiliza para sobrescribir velocidades: en la versión instalada se emite después de resolver contactos y podría anular su resultado. El controlador se ejecuta explícitamente antes de world.step.

### 10.3. Escritorio y VR

Time mide milisegundos y Experience los convierte a segundos. El bucle de WebXR usa su reloj propio. VRIntegration retorna sin avanzar la partida cuando no hay sesión inmersiva; así no se ejecutan dos simulaciones de escritorio a la vez.

### 10.4. Monedas, portales y victoria

| Nivel | Meta | Consecuencia |
| --- | ---: | --- |
| 1 | 3 monedas normales | Crea portal hacia el nivel 2. |
| 2 | 5 monedas normales | Crea portal hacia el nivel 3. |
| 3 | 6 monedas normales | Termina la partida. |

La ruta principal no exige recoger todas las monedas disponibles. Existe otra ruta opcional con finalPrize que se hace visible al recoger todas las monedas normales.

World recoge una moneda si la distancia al pivote es menor que 1.2 y la velocidad del personaje supera 0.5. Prize marca collected y retira su pivote. La detección es por proximidad, no por un cuerpo físico de moneda.

El portal usa un radio de proximidad 1.8. En el primer nivel tiene una posición fija; en el segundo busca baked_lev1_lev2 y usa una alternativa si no encuentra el ancla.

### 10.5. Cambio de nivel

1. loadLevel evita solicitudes simultáneas.
2. Suspende recogida y detiene la intención del jugador.
3. Obtiene los registros del nuevo nivel.
4. Carga sus recursos antes de retirar el mundo visible.
5. Destruye enemigos y retira monedas y portal anteriores.
6. Retira los cuerpos levelObject.
7. Retira mallas y libera geometría generada por lotes y materiales de suelo propios.
8. Libera recursos de otros niveles.
9. Actualiza el nivel actual y el acabado del suelo.
10. Instancia el nuevo mundo y construye colisiones y lotes.
11. Restaura puntos y posición de aparición.
12. Crea enemigos con la dificultad del nuevo nivel.
13. Reactiva recogida y actualización normal.

LevelManager no añade un teletransporte retardado: la aparición depende del resultado de carga, no de un temporizador arbitrario.

### 10.6. Derrota, reinicio y resultados

World comprueba proximidad con enemigos durante la partida. Cuando detecta derrota detiene las intenciones, para el cronómetro y muestra opciones de reintentar o salir. Reintentar vuelve al nivel 1, restaura puntos, revive al personaje y reinicia el cronómetro.

Al terminar se ordenan los cinco mejores tiempos en localStorage. El envío al backend conserva el desajuste de campos explicado en Limitaciones conocidas.

## 11. Cámaras, entrada, interfaz, sonido y VR

### 11.1. Selector de cámara

La rueda de Ajustes contiene tres botones y guarda la preferencia bajo cameraMode. Los valores internos son drone, thirdPerson y firstPerson.

| Modo | Comportamiento | Controles |
| --- | --- | --- |
| Dron | Cámara libre con OrbitControls, desplazamiento y altura independientes del robot | WASD, Q/E, Shift para mayor velocidad, ratón y rueda. |
| Tercera persona | Sigue detrás del robot, respeta altura del terreno y aproxima la cámara ante estructuras | Flechas y espacio para el personaje. |
| Primera persona | Mira hacia +Z relativo al robot, a la altura de los ojos; oculta el modelo propio | Flechas, espacio y arrastre del ratón. |

Camera coordina el modo y FOV. ThirdPersonCamera y FirstPersonCamera calculan sus vistas. El indicador isThirdPerson de Experience permanece como adaptador de compatibilidad.

La primera persona no genera bloques al hacer clic, para permitir mirar sin modificar el escenario. La cámara dron no tiene un cuerpo físico: explorar libremente con ella no equivale a mover al robot.

### 11.2. Controles de personaje y móvil

KeyboardControls mantiene up, down, left, right y space. Robot utiliza avance y retroceso relativos a su orientación y giro en Y. MobileControls crea controles solo cuando detecta capacidad táctil y actualiza las mismas teclas lógicas.

### 11.3. Ajustes y HUD

CircularMenu crea:

- Botón de ajustes.
- Selector de cámaras con estado aria-pressed.
- Ayuda de controles correspondiente.
- Audio, pantalla completa, VR y cancelación.
- Indicadores de tiempo, puntos y jugadores.

El nombre y correo personales se retiraron de la interfaz. El contador de jugadores es local mientras no se active el multijugador real.

ModalManager presenta mensajes y botones. Sus callbacks ejecutan la acción y ocultan el modal. GameTracker administra cronómetro, resultados y controles de repetición.

### 11.4. Audio

Sound utiliza Howler. Antes de reproducir comprueba interacción del usuario y reanuda el contexto si es necesario. Los sonidos están asociados a moneda, portal, derrota, victoria, movimiento, salto y proximidad de enemigos.

AmbientSound descarga y decodifica el audio ambiental y utiliza Web Audio para alternar reproducción desde el botón de audio. Los navegadores pueden bloquear audio hasta recibir un gesto.

### 11.5. Realidad virtual

VRIntegration crea el botón de WebXR, solicita sesión inmersiva y configura controladores y rayos. El movimiento solicita dirección al robot mediante su motor físico; no desplaza directamente el dolly para atravesar colisiones.

También conserva selección de premios por láser y paneles de registro. Esa selección debe revisarse junto con la lógica de puntos si amplías VR. La validación automatizada de navegador no sustituye una prueba con un dispositivo inmersivo real.

## 12. Optimización y propiedad de recursos

### 12.1. Carga y memoria

Resources conserva un catálogo completo, pero carga el primer nivel y recursos comunes al inicio. No se crean tres copias independientes del archivo del enemigo: SkeletonUtils.clone genera esqueletos propios y comparte geometría y materiales.

Al abandonar un nivel se retiran sus instancias antes de liberar los recursos de su catálogo. No destruyas geometrías compartidas al recoger una moneda o destruir un enemigo: otros objetos pueden seguir utilizándolas.

### 12.2. Materiales

MaterialOptimizer convierte materiales estáticos costosos a MeshLambertMaterial. Mantiene color, mapa de color, emisivo, alfa, opacidad, caras y colores de vértice, según sus propiedades. Los mapas que no conserva se retiran de la estrategia gráfica del ambiente.

Esto reduce el coste respecto a materiales físicos de transmisión. Los personajes y otras mallas especiales no se convierten de forma indiscriminada.

### 12.3. Lotes visuales

SceneBatcher:

1. Revisa mallas estáticas opacas de paleta.
2. Omite personajes, arrays de materiales, mapas y colores de vértice.
3. Agrupa por material y celdas de 32 unidades.
4. Solo combina grupos de al menos tres mallas.
5. Transforma y combina geometrías para el dibujo.
6. Oculta las mallas originales de esa instancia sin eliminar sus datos físicos.
7. Marca la geometría generada como propia para liberarla al salir.

En los mundos actuales se ahorraron 22 llamadas de dibujo por agrupación en cada uno de los niveles 1 y 2. El nivel 3 se beneficia sobre todo de materiales más ligeros, porque utiliza más texturas y no produce esos mismos grupos de paleta.

### 12.4. Renderizador

Renderer utiliza preferencia de GPU de alto rendimiento, sin antialias ni sombras dinámicas. Limita la densidad base a 1.25 y ajusta resolutionScale entre 0.65 y 1.

Cada 90 muestras válidas de partida:

- Si el tiempo medio supera 1/40 s, reduce la escala en 0.1.
- Si baja de 1/58 s, intenta recuperarla en 0.05.
- No ajusta durante la carga de un nivel.

La resolución adaptativa mejora el coste por cuadro reduciendo píxeles; no cambia el paso físico ni la velocidad objetivo del enemigo. La anisotropía de los acabados y carteles se limita a cuatro.

No confundas menos tiempo de carga, menos llamadas de dibujo y FPS: son métricas distintas. Las cifras medidas dependen del equipo y de la cámara utilizada.

## 13. Clases y responsabilidades

Las tablas siguientes resumen los puntos de entrada. El [catálogo completo](docs/INDICE.md) añade todos los archivos, métodos, parámetros y referencias por línea.

### 13.1. Núcleo y entidades

| Clase o módulo | Se encarga de |
| --- | --- |
| Experience | Construcción global, ciclo, cámaras, reinicio y coordinación. |
| World | Niveles, reglas, entidades, portales, puntos y limpieza. |
| Camera | Modos, controles y preferencia local. |
| Renderer | Renderizado WebGL y resolución adaptativa. |
| Robot | Personaje jugable, clips y controles. |
| Enemy | Persecución, animación y audio por enemigo. |
| Fox | Acompañante visual animado sin cuerpo físico. |
| Floor | Respaldo físico y acabado del suelo. |
| Prize | Pivote y estado de una moneda. |
| LevelManager | Metas y transición. |
| Environment | Iluminación y entorno. |
| FirstPersonCamera | Posición y orientación desde los ojos. |
| ThirdPersonCamera | Seguimiento y aproximación ante paredes. |
| BlockPrefab | Instancias de bloques dinámicos. |
| Sound / AmbientSound | Efectos y música. |
| debugScene | Cajas visuales de depuración; no está activado en el flujo normal. |

### 13.2. Utilidades

| Clase o módulo | Se encarga de |
| --- | --- |
| Physics | Mundo Cannon y pasos fijos. |
| CharacterMotor | Movimiento y contactos de un personaje. |
| PhysicsShapeFactory | Construcción de colisionadores. |
| EnemySteering | Evitación local y separación. |
| Difficulty | Configuración por nivel. |
| Resources | Carga, promesas compartidas y liberación. |
| MaterialOptimizer | Materiales difusos y mapas conservados. |
| SceneBatcher | Agrupación de mallas estáticas. |
| EventEmitter | Registro, emisión y retirada de eventos. |
| Time | Reloj y tick de escritorio. |
| Sizes | Tamaño de ventana y resize. |
| Debug | Activación de lil-gui. |
| Device | Detección móvil disponible como utilidad. |
| KeyboardControls | Estado de teclas. |
| Raycaster | Clics y obstáculos. |
| GameTracker | Tiempo, ranking y resultados. |
| progressSync | Petición de progreso autenticada. |
| ModalManager | Ventanas y callbacks. |
| FinalPrizeParticles | Efectos temporales de portales. |

### 13.3. Interfaz e integraciones

| Módulo | Se encarga de |
| --- | --- |
| main.jsx | Montar React. |
| App.jsx | Autenticación, acceso offline y eventos de carga. |
| CircularMenu | Ajustes, cámaras y HUD. |
| MobileControls | Joystick y salto táctil. |
| VRIntegration | WebXR y controladores. |
| ToyCarLoader | Registros, instancias, texturas, cuerpos y lotes. |
| SocketManager | Cliente opcional de jugadores remotos, actualmente inactivo. |

### 13.4. Backend y mantenimiento

| Módulo | Se encarga de |
| --- | --- |
| app.js | Express, Mongoose, HTTP y Socket.io. |
| Block / User / PlayerProfile | Esquemas de persistencia. |
| blockController | Consultar e insertar objetos. |
| authController | Registrar, autenticar y firmar tokens. |
| playerProfileController | Consultar y actualizar perfil. |
| authMiddleware | Verificar JWT. |
| blockRoutes / authRoutes / profileRoutes | Enlazar rutas con controladores. |
| Text_3_cero.txt | Exportar mallas y registros desde Blender. |
| sync_blocks.js | Enviar JSON al servidor. |
| generate_sources.js | Generador histórico de catálogo intermedio. |
| generatePhysicsModels.js | Lista histórica de nombres. |
| renameToLowercase.js | Renombrado histórico en ruta fija. |
| seed.js | Demostración destructiva de posiciones. |
| document-code.mjs | Comentarios, referencias por línea y comprobación de cobertura. |

## 14. Cambios realizados

| Problema anterior | Cambio | Dónde localizarlo |
| --- | --- | --- |
| Fuerzas aplicadas con posición mundial y giro físico producían movimientos inestables | Motor de esfera sin giro y control horizontal antes del solver | CharacterMotor, Robot, Physics. |
| Suelo importado no se aprovechaba correctamente y había superficies solapadas | Colisiones desde GLB y respaldo inferior, con acabados por nivel | PhysicsShapeFactory, Floor, ToyCarLoader. |
| Geometría expandida e índices sin control | Limpieza, soldado, centrado y división de índices | PhysicsShapeFactory. |
| Enemigos fijaban su altura y usaban correcciones verticales | Gravedad, contactos y navegación reactiva | Enemy, EnemySteering. |
| Dos bucles avanzaban escritorio | El bucle de WebXR retorna fuera de VR | VRIntegration, Experience. |
| Transiciones esperaban un objeto y la API devolvía un arreglo | Normalización compartida de ambos formatos | ToyCarLoader.getLevelData, World.loadLevel. |
| Respaldo conservaba referencias antiguas | Combinación de los tres JSON actuales | public/data/toy_car_blocks.json. |
| Retirar un tick podía borrar el ciclo principal | Retirada selectiva de callbacks | EventEmitter.off. |
| Modo de cámara limitado | Selector dron, tercera y primera persona | Camera y CircularMenu. |
| El menú mostraba identidad personal | Se retiró el apartado de nombre y correo | CircularMenu. |
| Velocidad enemiga dependía de distancia | Velocidad objetivo constante por dificultad | Difficulty y Enemy. |
| El borde de obj_lev2 bloqueaba seguimiento | Asistencia de escalón con comprobación de techo | CharacterMotor y EnemySteering. |
| Se precargaban todos los niveles y se repetían recursos del enemigo | Carga por nivel, descarga y esqueleto independiente con recursos compartidos | Resources y Enemy. |
| Materiales físicos y muchas mallas pequeñas encarecían el dibujo | Materiales ligeros, lotes y resolución adaptativa | MaterialOptimizer, SceneBatcher, Renderer. |
| Documentación dispersa y comentarios antiguos | Guía, catálogo, comentarios en español y referencia por línea | README, docs y document-code.mjs. |

La última tarea de documentación añade comentarios y referencias sin cambiar las instrucciones ejecutables del juego. El generador comprueba el árbol sintáctico antes y después de anotar JavaScript, CSS y Python.

## 15. Pruebas y validación

### 15.1. Pruebas automatizadas

Desde game-project:

~~~powershell
npm.cmd test
npm.cmd run build
~~~

Las 20 pruebas actuales verifican:

1. Reconocimiento del último sufijo de nivel.
2. Solicitudes compartidas y ausencia de precarga del nivel 2.
3. Conservación de mapas y paleta al simplificar materiales.
4. Lotes con límites mundiales y jerarquía física conservados.
5. Primera persona alineada con el movimiento.
6. Apoyo sobre los tres planos GLB reales.
7. Contacto contra una pared a 30, 60 y 144 cuadros/s simulados.
8. Salto desde una plataforma elevada.
9. Contacto contra una casa GLB real sin lanzamiento.
10. División de mallas grandes con índices válidos.
11. Navegación alrededor de un edificio.
12. Acabados de suelo distintos por nivel.
13. Colisionadores finitos para los 116 objetos de ambiente.
14. Aparición de enemigos fuera de edificios.
15. Retirada de eventos sin borrar el ciclo principal.
16. Subida a una plataforma baja con ambos radios.
17. Rechazo de paredes altas.
18. Rechazo de un escalón sin altura libre.
19. Entrada a la plataforma real obj_lev2 con ambos personajes.
20. Velocidad de crucero constante y dificultad creciente.

Las pruebas cargan geometría GLB sin imágenes de navegador y ejecutan Cannon real. No escriben en MongoDB ni ejecutan los scripts destructivos.

### 15.2. Documentación y sintaxis

Desde la raíz:

~~~powershell
node scripts/document-code.mjs --check
node --check backend/app.js
py -3.12 -c "import ast,pathlib; ast.parse(pathlib.Path('backend/scripts/Text_3_cero.txt').read_text(encoding='utf-8')); print('Sintaxis Python correcta')"
~~~

No existe una suite de backend configurada: npm test dentro de backend conserva el mensaje de plantilla y termina con error. Eso no equivale a un fallo de las pruebas del frontend.

### 15.3. ESLint

npm run lint revisa el alcance completo de la configuración, que incluye archivos de terceros de public y utilidades históricas. Para revisar el código de juego sin decodificadores externos:

~~~powershell
Set-Location '.\game-project'
npx.cmd eslint src/App.jsx src/Experience/Camera.js src/Experience/Renderer.js src/Experience/Experience.js src/Experience/Utils/CharacterMotor.js src/Experience/Utils/EnemySteering.js src/Experience/Utils/Difficulty.js src/Experience/Utils/Resources.js src/Experience/Utils/MaterialOptimizer.js src/Experience/Utils/SceneBatcher.js src/Experience/Utils/Raycaster.js src/Experience/World/Enemy.js src/Experience/World/Floor.js src/Experience/World/World.js src/Experience/World/ThirdPersonCamera.js src/Experience/World/FirstPersonCamera.js src/Experience/World/Prize.js src/controls/CircularMenu.js src/loaders/ToyCarLoader.js
~~~

### 15.4. Comprobación de navegador

Se verificaron en Chrome sin interfaz:

- Acceso sin conexión.
- Recursos del primer nivel.
- Las tres cámaras y sus estados de controles y visibilidad.
- Persistencia del modo al recargar.
- Ausencia de nombre y correo personales en el menú.
- Cambios repetidos de nivel.
- Respuestas de API simuladas en forma de arreglo.
- Cantidad de recursos acotada.
- Velocidades configuradas por nivel.
- Ausencia de errores JavaScript de página.

Las respuestas simuladas no demuestran una operación real de MongoDB. No se ejecutó una nueva exportación de Blender en esta máquina ni una sesión con visor VR.

## 16. Limitaciones conocidas

Estas observaciones describen el código actual y deben considerarse antes de ampliarlo.

| Área | Estado y acción necesaria |
| --- | --- |
| Progreso | GameTracker envía bestTimeSeconds, pero progressSync reconoce elapsedSeconds o timeSeconds. Actualmente puede enviar tiempo cero. |
| Monedas persistidas | progressSync envía points y el backend espera coinsCollected; no se acumulan como espera el perfil. |
| Reanudación | El acceso no carga automáticamente el perfil ni restaura el nivel guardado. |
| Preferencias | Camera usa valores locales drone/thirdPerson/firstPerson; settings.cameraMode del perfil conserva otro valor inicial y no está conectado. |
| Lotes de bloques | insertMany agrega documentos y no evita duplicados. |
| Validación de datos | El esquema Block admite campos de nombre o posición ausentes. |
| Seguridad de desarrollo | Bloques sin autenticación y CORS abierto; revisar antes de exponer un editor o servidor público. |
| Salud | ping no comprueba conexión con MongoDB. |
| Multijugador | El cliente está inactivo y app.js no implementa manejadores de jugadores; crear Socket.io no completa esa funcionalidad. |
| Importadores antiguos | Rutas D:/ y salidas fijas requieren configuración manual. |
| seed.js | Elimina la colección de bloques y no importa modelos de un mundo. |
| VR | Conserva lógica de premios por láser que debe coordinarse con puntuación; requiere prueba real. |
| Navegación | Evitación local, sin garantía de resolver laberintos o rutas largas bloqueadas. |
| Físicas | No importa restricciones, masas o materiales físicos de Blender; los configura JavaScript. |
| Pausas | El delta limitado evita acumulación extrema, pero no pretende simular en tiempo real una pestaña suspendida. |
| Lint global | La configuración incluye dependencias públicas y código histórico que no corresponde a los archivos activos validados. |
| Bundle | Queda aviso de tamaño del JavaScript compilado; la carga de modelos se optimiza por separado. |
| Colisiones de nuevas formas | Validar soporte de Cannon antes de sustituir las esferas o añadir nuevas combinaciones con Trimesh. |

No se corrigieron esas funciones al documentar, para no mezclar una actualización explicativa con cambios funcionales no probados.

## 17. Guía para cambios posteriores

### Añadir o sustituir una estructura

1. Edita Blender y conserva una escala coherente.
2. Exporta una malla por GLB.
3. Actualiza el JSON del nivel.
4. Actualiza sources.js.
5. Reconstruye el respaldo combinado.
6. Sincroniza MongoDB deliberadamente si lo utilizas.
7. Comprueba colisión y textura.
8. Ejecuta pruebas y regenera documentación.

### Cambiar velocidad o cantidad de enemigos

- Velocidad e intervalo: Difficulty.js.
- Cantidad: VITE_ENEMIES_COUNT; reinicia Vite.
- Radio y altura visual: Enemy.
- Evitación y separación: EnemySteering.
- Aceleración y apoyo compartidos: CharacterMotor.

Si aumentas velocidad, valida de nuevo el paso físico, bordes y paredes.

### Cambiar salto, fricción o colisión

- Gravedad, solver y contactos: Physics.
- Salto, frenado y escalón: CharacterMotor.
- Transformaciones de la malla y división de triángulos: PhysicsShapeFactory.
- Creación y vínculo del cuerpo: ToyCarLoader.
- Limpieza: World.clearCurrentScene y Physics.removeLevelBodies.

No cambies solo el modelo visible esperando modificar la colisión de un cuerpo ya creado.

### Añadir un cuarto mundo

1. Exporta nivel 4 y añade sus registros al catálogo y respaldo.
2. Revisa sourceLevel y la carga por bloques.
3. Amplía totalLevels y pointsToComplete de LevelManager.
4. Añade dificultad en Difficulty.
5. Añade un acabado a FLOOR_THEMES o define explícitamente el deseado.
6. Añade ubicación de portal y reglas de finalización en World.
7. Comprueba que el antiguo último nivel ya no termine antes de tiempo.
8. Añade pruebas y actualiza esta guía.

Añadir archivos GLB por sí solo no amplía el flujo de juego.

### Cambiar cámaras o controles

- Modos, FOV, persistencia y entrada de dron: Camera.
- Seguimiento: ThirdPersonCamera.
- Ojos, orientación y pitch: FirstPersonCamera.
- Botones y ayuda: CircularMenu.
- Movimiento del personaje: Robot y KeyboardControls.
- Entrada móvil: MobileControls.
- Entrada VR: VRIntegration.

Mantén la orientación de primera persona coherente con +Z relativo al robot.

### Cambiar materiales o rendimiento

- Conservación de mapas y conversión: MaterialOptimizer.
- Criterio de lotes: SceneBatcher.
- Precarga, concurrencia y liberación: Resources.
- Densidad de píxeles y adaptación: Renderer.
- Acabados del suelo: Floor.

Mide carga, memoria, llamadas de dibujo y tiempo por cuadro por separado y utiliza la misma escena y cámara para comparar.

### Cambiar el contrato de la API

Modifica conjuntamente modelo, controlador, rutas, consumidor frontend y ejemplos. Para progreso, alinea GameTracker, progressSync y playerProfileController. Añade pruebas que verifiquen los campos enviados antes de afirmar que el guardado funciona.

## 18. Documentación, Git y publicación

### 18.1. Mantener comentarios y referencias

Desde la raíz:

~~~powershell
node scripts/document-code.mjs --annotate
node scripts/document-code.mjs --check
~~~

El primer comando:

1. Lee las descripciones de docs/modulos.json.
2. Analiza el código propio con Babel y los estilos con PostCSS.
3. Inserta comentarios breves en posiciones seguras.
4. Usa ast de Python para comprobar el exportador.
5. Verifica que los árboles sintácticos no cambien al añadir comentarios.
6. Genera un documento por archivo con métodos, parámetros y explicación por línea.
7. Genera el índice y las huellas de cobertura.

El segundo comando no modifica fuentes: comprueba que las referencias correspondan al código actual. Si modificas un módulo, revisa también sus descripciones mantenidas en modulos.json. El generador no sustituye la revisión humana de una nueva regla de negocio.

Para usar --annotate necesitas las dependencias de desarrollo del frontend y, en Windows, py -3.12. --check no necesita ejecutar el exportador ni conectarse a MongoDB.

Los archivos package-lock son generados por npm y documentan las versiones exactas; no se comentan manualmente. Tampoco se alteran los decodificadores Draco ni bibliotecas de terceros. .gitignore se conserva sin cambios.

### 18.2. Antes de publicar

~~~powershell
git status --short
node scripts/document-code.mjs --check
npm.cmd --prefix game-project test
npm.cmd --prefix game-project run build
git diff --check
~~~

Revisa especialmente que no se incluyan .env, credenciales, contraseñas ni volcados privados de base de datos. Las plantillas contienen solo ejemplos.

### 18.3. Commit y subida

El mensaje solicitado para esta entrega es exactamente:

~~~text
Cambius finales y ajustes colision
~~~

Flujo habitual:

~~~powershell
git add -A
git diff --cached --stat
git commit -m "Cambius finales y ajustes colision"
git push origin main
git rev-parse HEAD
git ls-remote origin refs/heads/main
~~~

La referencia remota debe coincidir con el hash del commit local antes de considerar terminada la publicación. No se necesita modificar .gitignore para documentar o publicar estos cambios.

### 18.4. Cierre del trabajo y apagado solicitado

El apagado es una acción operativa de cierre, no parte del juego ni una función que deba ejecutarse al abrir el repositorio. En esta entrega se solicita únicamente después de verificar la subida.

Para permitir que se reciba el resultado antes de apagar, el cierre puede iniciarse mediante un proceso auxiliar con una espera breve y después shutdown /s /t 0 /soft. El proceso no utiliza /f. Durante esa espera se cancela deteniendo el proceso auxiliar; si Windows ya tiene un apagado con plazo propio, se utiliza shutdown /a. No se instala ningún apagado automático al arrancar el juego.

## 19. Referencias técnicas

La descripción funcional de esta guía se basa en el código del repositorio. Para ampliar las APIs externas:

- [Exportador glTF de Blender](https://docs.blender.org/api/5.0/bpy.ops.export_scene.html).
- [Materiales glTF en Blender](https://docs.blender.org/manual/en/5.1/addons/import_export/scene_gltf2.html).
- [Variables de entorno de Vite](https://vite.dev/guide/env-and-mode).
- [Formatos de conexión de MongoDB](https://www.mongodb.com/docs/manual/reference/connection-string-formats/).
- [GLTFLoader de Three.js](https://threejs.org/docs/pages/GLTFLoader.html).
- [Clonación de esqueletos de Three.js](https://threejs.org/docs/pages/module-SkeletonUtils).
- [Optimización por combinación de geometrías](https://threejs.org/manual/pages/optimize-lots-of-objects.html).
- [Cuerpos de Cannon](https://pmndrs.github.io/cannon-es/docs/classes/Body.html).
- [Mallas triangulares de Cannon](https://pmndrs.github.io/cannon-es/docs/classes/Trimesh.html).

Consulta el [catálogo completo de código](docs/INDICE.md) para continuar desde esta visión general hasta cada instrucción.
