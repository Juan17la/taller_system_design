# 2. Glosario

Cada término que usa el sistema, agrupado por tema. El **nombre en la UI** es lo
que lee el jugador; el **nombre en el código** es el que aparece en el código
fuente.

## Objetos principales

| Término | Significado |
|---|---|
| **Launcher** | La app de escritorio Udeos completa (ventana Wails + backend en Go). |
| **Perfil (Profile)** | Una identidad local de jugador: un nickname más su UUID offline, idioma, tema y skins. **No hay cuenta ni servidor de login**; "iniciar sesión" solo elige un nickname. Pueden existir muchos perfiles; uno está *activo*. |
| **Nickname** | El nombre de jugador de un perfil. También es la clave del perfil (`Instance.Owner` lo guarda). |
| **Instancia (Instance)** | Una instalación de Minecraft autocontenida que el launcher puede iniciar: un nombre, una versión de Minecraft, un loader, un icono, ajustes de lanzamiento y su propia carpeta de juego privada (mundos, mods, opciones). Dos instancias nunca comparten mundos ni mods. |
| **Instancia de juego** | Una instancia que *juegas* (`Server = false`). Aparece en la página **Instancias** (código: `dashboard`). |
| **Servidor (Server)** | Una instancia que *hospedas* (`Server = true`): su `.minecraft` es la carpeta del servidor. Aparece en la página **Servidores**. Mismo almacén que las instancias. |
| **Dueño (Owner)** | El perfil que posee una instancia. Cada perfil solo ve las suyas. |
| **Dashboard** | Nombre en el código de la página **Instancias** (cuadrícula de tarjetas de instancias + panel de "última partida"). |
| **Skin** | Una textura de jugador (PNG, modelo *classic* o *slim*) en la biblioteca de skins. Una skin está *equipada* por perfil. |

## Conceptos de Minecraft

| Término | Significado |
|---|---|
| **Versión de Minecraft** | El lanzamiento del juego que ejecuta una instancia (`1.20.1`, `26.3`, …). También llamada *versión del juego*. |
| **Release / snapshot** | Versión estable vs. build en desarrollo (`24w14a`). Los filtros de búsqueda y las etiquetas de resultados muestran **solo releases**. |
| **Vanilla** | Minecraft puro, sin loader. Solo admite resource packs. |
| **Loader** (mod loader) | Software que permite al juego cargar mods: **Fabric**, **Quilt**, **Forge**, **NeoForge**. Se instala automáticamente en el primer Jugar. |
| **Versión / build del loader** | El lanzamiento exacto del loader (`0.16.9`, `1.20.1-47.4.10`). Se guarda como `loaderVersion`. |
| **Quilt ⊇ Fabric** | Una instancia Quilt ejecuta mods de Quilt *y* de Fabric, por lo que coincide con ambos dondequiera que se comparan loaders. |
| **JRE** | El runtime de Java que necesita el juego; se descarga de Mojang por versión, nunca se le pide al jugador. |
| **Assets / libraries / natives** | Archivos del juego obtenidos del CDN de Mojang: sonidos y texturas / jars de Java / binarios específicos del SO. Compartidos por todas las instancias. |
| **CDN** | Los servidores públicos de descarga de Mojang de donde el launcher obtiene los archivos del juego. |
| **Mundo (World)** | Una carpeta de guardado (`saves/<nombre>/`) reconocida por su `level.dat`. |
| **`.minecraft`** | El directorio del juego *dentro* de una instancia (`instances/<id>/.minecraft/`). |
| **Modo offline / UUID** | Jugar sin cuenta de Microsoft; el UUID se deriva del nickname. |
| **Authlib-injector / Yggdrasil** | El agente Java + el pequeño servidor local de skins que hacen que las skins se vean sin cuenta. |

## Addons (contenido)

| Término | Significado |
|---|---|
| **Addon** | Palabra de la UI para *cualquier contenido descargable de Modrinth*: un mod, resource pack, shader o modpack. La entrada de navegación **Addons** abre la página de búsqueda (código: `search`). "Content" en el código/docs significa lo mismo. |
| **Mod** | Código que modifica el juego. Necesita un loader. Va a `mods/`. |
| **Resource pack** | Texturas/sonidos/idioma. Funciona también en Vanilla. Va a `resourcepacks/`. |
| **Shader** (shader pack) | Efectos de iluminación/visuales. Va a `shaderpacks/`. En este launcher necesita un loader. |
| **Modpack** | Un paquete (`.mrpack`) de muchos mods + ajustes. Puede convertirse en una **nueva instancia** (con el loader que declara el pack) o volcarse en una instancia existente compatible, sin sobrescribir nunca. |
| **Project** | La palabra de Modrinth para la *página* de un addon (id, slug, título, autor, icono). Un resultado de búsqueda es un Project. |
| **Project version** | Un lanzamiento de un project con las versiones de Minecraft y loaders que soporta. |
| **File** | El artefacto descargable de una version, verificado con SHA-1. |
| **Dependency** | Un enlace de una version a otro project: `required` (se instala automáticamente), `optional` (solo se menciona), `incompatible` (bloquea la instalación), `embedded` (ya incluida). |
| **Plan** | El resultado de *planificar* una instalación: qué version + qué dependencias requeridas se descargarán, o un error simple (sin build, incompatible con X). |
| **Instancia compatible** | Una instancia cuya versión de Minecraft *y* (para mods/modpacks) loader coinciden con el addon. El selector de Añadir lista solo estas. |
| **Bloqueado (modo instancia)** | Cuando Addons se abre desde la página de una instancia, la versión y el loader quedan fijados a esa instancia (se muestran deshabilitados) y **Añadir** instala de inmediato. |
| **Añadido (Added)** | Estado de la tarjeta que indica que el project ya está en la instancia (según `content.json`) o acaba de terminar de instalarse. |
| **`content.json`** | Registro por instancia de lo que instaló el launcher (project, version, file, SHA-1, *required by*, incompatibilidades, icono, descripción). Los archivos añadidos a mano no aparecen en él. |
| **Modrinth** | El marketplace público de addons y el único proveedor de contenido (`api.modrinth.com/v2`). |
| **Provider** | Interfaz de Go (`modsearch.Provider`) que implementa un marketplace; permite añadir otro sin tocar la UI. |

## Búsqueda e IA

| Término | Significado |
|---|---|
| **Query** | Entrada de búsqueda independiente del proveedor: tipo, texto, versión, loader, orden, categorías, offset, límite. |
| **Facet** | La sintaxis de filtros de Modrinth: un array JSON de grupos OR combinados con AND. El launcher lo construye a partir de una Query. |
| **Categoría** | Una etiqueta de Modrinth como `optimization`, `technology`, `magic`. Pertenece a un tipo de project. Se muestra como etiquetas removibles; aquí los loaders *no* son categorías. |
| **Orden (`index`)** | Orden de los resultados: `relevance` (por defecto), `downloads`, `newest`, `updated`. No existe orden ascendente. |
| **Página / offset** | Los resultados se obtienen de 30 en 30; una página nueva *reemplaza* la cuadrícula. |
| **Caché** | Copias guardadas de respuestas remotas (memoria + `cache/search/*.json`) para que la página sea rápida y funcione sin conexión. |
| **Índice / indexación** | Modrinth mantiene el índice de búsqueda en su servidor. El launcher **no construye ningún índice de búsqueda local**; su único "índice" local es `content.json` (lo que tiene una instancia). Ver [Búsqueda](04-search.md). |
| **Ask AI (Preguntar a la IA)** | Panel de chat en Addons que convierte una frase en una búsqueda de Modrinth y recomienda hasta 3 resultados. |
| **Intent (intención)** | La búsqueda estructurada que la IA leyó de un mensaje: tipo, query, categorías, versión, loader, orden. |
| **Listas permitidas** | Los únicos valores que puede contener un Intent (tipos de la página, categorías de Modrinth, versiones conocidas, cuatro loaders, cuatro órdenes). Cualquier otro se descarta. |
| **Pick (selección)** | Paso 2 de Ask AI: el modelo elige ≤3 de los 8 mejores resultados y da una **razón** de una línea. |
| **Proveedor (IA)** | Groq (integrado), Claude, OpenAI, Gemini o Grok. Distinto de un proveedor de contenido. |
| **Clave integrada** | Una clave de Groq ofuscada dentro de los binarios de release, nunca guardada en el repo. |
| **Plan gratuito / de pago** | Marca en cada modelo de IA: se puede usar con una clave gratuita del proveedor, o necesita crédito. |

## Mecánica de la app

| Término | Significado |
|---|---|
| **Wails** | Framework que empaqueta un backend en Go y una UI web en un único ejecutable de escritorio. |
| **Binding** | Un método de Go expuesto a JavaScript como una función que devuelve una promesa (`api.SearchContent(...)`). |
| **Evento (Event)** | Un mensaje push de Go → UI: `install:progress`, `content:progress`, `game:state`, `server:state`, `server:log`, `app:close`. |
| **Bridge** | `frontend/src/api/bridge.ts`, la única puerta tipada hacia los bindings/eventos; recurre a un **mock** en memoria en un navegador normal. |
| **Pantalla (Screen)** | Una página que la UI puede mostrar; un valor del tipo `Screen` en `state/index.tsx`. |
| **Historial de navegación** | La pila de pantallas anteriores (máx. 20) que **Volver** desapila. |
| **Pestaña (Tab)** | Una subpágina dentro de una página de instancia o servidor (Mods, Mundos, Ajustes, …). |
| **Toast / Notificación** | Mensaje en la esquina inferior derecha para progreso, éxito o errores (instalaciones de contenido, progreso de lanzamiento). |
| **Jugar / Lanzar (Play / Launch)** | Construir la línea de comandos de Java e iniciar el juego; muestra un modal de carga que se puede ocultar. |
| **Cola de contenido** | Cola serial de trabajos de Añadir para que los toasts de progreso sigan siendo legibles (`useContentQueue`). |
| **Tokens** | Variables de diseño (colores, radios, sombras) en `theme/tokens.css`; dos temas: *Pastel Overworld* (claro) y *Pastel End* (oscuro). |
| **i18n** | Diccionarios en inglés y español; todo texto visible sale de uno. |
| **Relay / Router (servidor)** | Dos formas de que los amigos lleguen a un servidor hospedado desde internet: un túnel relay público (por defecto) o un port-forward UPnP en el router. |
