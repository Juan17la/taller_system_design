# 1. Jerarquía

Importan dos jerarquías: la del **dominio** (lo que posee el jugador) y la del
**código** (qué capa puede llamar a cuál).

## Jerarquía del dominio — qué contiene a qué

```
Launcher
└── Perfil (un nickname; el activo es "quien está jugando")
    ├── Skins (biblioteca; una equipada por perfil)
    ├── Preferencias (idioma, tema)
    └── Instancias (cada una pertenece a exactamente un perfil)
        ├── Instancia de juego  (Server = false)      ← Página Instancias
        │   ├── Versión de Minecraft   (p. ej. 1.20.1)
        │   ├── Loader (+ build del loader)  Vanilla | Fabric | Quilt | Forge | NeoForge
        │   ├── Ajustes de lanzamiento (memoria, ruta de Java, argumentos JVM extra)
        │   └── Contenido  (.minecraft/)
        │       ├── Mods             mods/          ← requiere un loader
        │       ├── Resource packs   resourcepacks/
        │       ├── Shaders          shaderpacks/   ← requiere un loader
        │       ├── Mundos           saves/
        │       └── Capturas         screenshots/
        └── Instancia de servidor (Server = true)      ← Página Servidores
            ├── misma versión / loader / ajustes de lanzamiento
            ├── server.properties, listas de jugadores (whitelist / ops / baneados)
            ├── Backups, Consola, Acceso a internet (relay o router)
            └── Mods (solo cuando el loader no es Vanilla)
```

Reglas que se derivan de este árbol:

1. **Un perfil posee instancias; una instancia tiene un solo dueño.** Cambiar de
   perfil cambia qué instancias ves. Eliminar un perfil nunca borra nada: sus
   instancias pasan al perfil que lo sustituye.
2. **Un servidor es una instancia con `Server = true`.** Mismo almacén, mismo
   dueño, mismos ajustes de lanzamiento, misma carpeta `mods/`; solo cambian la
   página y el proceso. Las listas de instancias de juego omiten los servidores;
   las listas de servidores añaden estado en vivo (en ejecución, jugadores,
   dirección).
3. **El contenido pertenece a exactamente una instancia.** No se comparte nada
   entre instancias salvo el caché de descargas (los archivos son idénticos, así
   que se guardan una sola vez).
4. **Vanilla no tiene Mods ni Shaders.** Una instancia Vanilla solo admite
   resource packs; un servidor solo admite mods.

### Datos compartidos vs. privados

| Compartidos por todas las instancias (se pueden volver a descargar) | Privados de una instancia (tuyos) |
|---|---|
| `versions/`, `libraries/`, `assets/`, `runtimes/` | entrada en `instances.json` |
| `cache/` (páginas de búsqueda, listas de loaders, archivos de addons descargados) | `instances/<id>/.minecraft/` (mundos, opciones, logs) |
| | `instances/<id>/content.json` (lo instalado desde Modrinth) |

Estructura completa: [Datos en disco](../05-data-on-disk.md).

## Jerarquía del contenido — de qué está hecho un addon (lado Modrinth)

```
Project            "Sodium" — lo que se muestra como tarjeta (id, título, autor, icono)
└── Version        un lanzamiento: en qué versiones de Minecraft y loaders funciona
    ├── File       el .jar / .zip / .mrpack a descargar (verificado con SHA-1)
    └── Dependency required | optional | incompatible | embedded → otro Project
```

Al explorar se muestran **Projects**. Al añadir a una instancia se elige una
**Version** que coincida con la versión de Minecraft y el loader de la
instancia, se descarga su **File** y se recorren sus **Dependencies** requeridas.

## Jerarquía del código — qué capa llama a cuál

```
frontend/src/screens/*          ← lo que ve el jugador (páginas)
        │ usa
        ▼
components/  hooks/  state/     ← piezas compartidas, estado de la app, navegación
        │ llama
        ▼
api/bridge.ts                   ← la ÚNICA puerta a Go (tipada; mock en el navegador)
        │ bindings de Wails + eventos
        ▼
app*.go (package main)          ← bindings: delgados, un archivo por función
        │ llama
        ▼
internal/core                   ← orquestador (instalar → Java → loader → lanzar)
        │ usa
        ▼
internal/{install, jre, loader, launch, instance, profile, content,
          modsearch, modinstall, modpack, ai, server, skin, tunnel, …}
        │ lee/escribe               │ HTTP
        ▼                           ▼
   carpeta de datos           Mojang / Modrinth / proveedor de IA
```

La dirección es estrictamente descendente: los paquetes `internal/*` nunca
importan la UI y las pantallas nunca llaman a la red. Detalles de cada capa:
[Arquitectura](../01-architecture.md).

### Carpetas del frontend, por responsabilidad

| Carpeta | Rol | Regla |
|---|---|---|
| `screens/` | Un archivo (o carpeta) por página | Las páginas son dueñas de su diseño y sus piezas locales |
| `components/` | Reutilizados en más de una página (Nav, diálogos, etiquetas, botón Jugar) | Extraer solo al segundo uso |
| `ui/` | Primitivas de aspecto (Button, Field, Dialog, …) | Carpeta plana, sin lógica |
| `hooks/`, `state/` | Comportamiento compartido entre pantallas; estado de la app (perfil, instancias, navegación) | Las pantallas leen el estado, no lo poseen |
| `utils/` | Funciones puras, sin React | Se pueden probar de forma aislada |
| `api/` | Bridge + tipos + mock del navegador | Única frontera hacia Go |
| `i18n/` | Textos en inglés y español | Sin texto de UI escrito a mano |
