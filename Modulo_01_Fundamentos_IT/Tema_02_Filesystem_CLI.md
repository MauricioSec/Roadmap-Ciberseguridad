# 📁 Tema 2: Arquitectura del Sistema de Archivos Linux y Fundamentos de la CLI

## 🎯 Objetivo del Tema

Comprender la estructura jerárquica del sistema de archivos de Linux (**Filesystem Hierarchy Standard - FHS**) y aprender a navegar por él utilizando la **Command Line Interface (CLI)** mediante la terminal Bash. Este conocimiento constituye la base para la administración de sistemas, el análisis forense, la respuesta a incidentes y las operaciones de ciberseguridad.

---

# 🧠 Conceptos Fundamentales

## El principio "Todo es un archivo"

Linux sigue el principio arquitectónico de que **la mayoría de los recursos del sistema se representan mediante la interfaz del sistema de archivos**.

Esto significa que elementos como:

* Archivos
* Directorios
* Dispositivos de hardware
* Discos
* Terminales
* Procesos (a través de `/proc`)
* Interfaces del kernel

pueden administrarse utilizando las mismas herramientas básicas del sistema.

A diferencia de Windows, Linux no utiliza unidades independientes (`C:\`, `D:\`, etc.). Todo el sistema parte desde un único directorio denominado **Raíz (Root)**, representado por:

```bash
/
```

Toda la jerarquía del sistema cuelga de este punto.

---

# 📂 Directorios Fundamentales del FHS

## `/` — Root

Es el directorio principal del sistema.

Todos los archivos y directorios existentes dependen directa o indirectamente de esta ubicación.

---

## `/bin` y `/sbin`

Históricamente almacenaban los binarios esenciales del sistema y las utilidades administrativas.

> **Nota:** En muchas distribuciones modernas (Ubuntu, Debian, Fedora, entre otras), estos directorios son enlaces simbólicos hacia:

* `/usr/bin`
* `/usr/sbin`

No obstante, continúan formando parte del estándar FHS y es habitual encontrarlos en documentación técnica.

---

## `/etc`

Contiene los archivos de configuración del sistema operativo y de la mayoría de los servicios instalados.

Desde una perspectiva defensiva, suele ser uno de los primeros directorios revisados para:

* comprender la configuración del sistema;
* verificar servicios instalados;
* analizar cambios posteriores a un incidente;
* revisar configuraciones de autenticación y red.

---

## `/var`

Almacena información variable generada durante el funcionamiento del sistema.

Entre sus subdirectorios más importantes se encuentra:

```bash
/var/log
```

Aquí se almacenan gran parte de los registros (logs) utilizados durante:

* investigaciones forenses;
* respuesta a incidentes;
* monitoreo de sistemas;
* análisis SOC.

---

## `/tmp`

Directorio destinado al almacenamiento de archivos temporales.

Por defecto:

* cualquier usuario puede crear archivos dentro de este directorio;
* su contenido puede eliminarse automáticamente tras un reinicio o mediante tareas de limpieza del sistema.

Dependiendo de la configuración del servidor, `/tmp` puede montarse con opciones de seguridad como:

```text
noexec
nosuid
nodev
```

Estas opciones reducen la superficie de ataque impidiendo determinadas acciones, como la ejecución directa de binarios desde dicho directorio.

---

## `/home`

Contiene los directorios personales de los usuarios del sistema.

Ejemplo:

```text
/home/mauricio
```

El atajo:

```bash
~
```

representa el directorio personal del usuario actualmente autenticado.

---

# ⚠️ Reglas Fundamentales de Linux

## 1. Case Sensitive

Linux distingue entre mayúsculas y minúsculas.

Ejemplo:

```text
malware.sh
Malware.sh
MALWARE.SH
```

Son tres archivos completamente distintos.

### Importancia en Ciberseguridad

Ignorar esta característica puede provocar:

* búsquedas incompletas;
* errores durante investigaciones;
* reglas de firewall incorrectas;
* firmas SIEM ineficaces;
* falsos negativos en la detección de amenazas.

---

## 2. Uso de rutas

Linux utiliza la barra diagonal:

```text
/
```

como separador de directorios.

Ejemplo:

```text
/var/log
```

Windows utiliza:

```text
\
```

Ejemplo:

```text
C:\Windows\System32
```

---

# 💻 Comandos Básicos de Navegación

## Mostrar la ubicación actual

```bash
pwd
```

Muestra la ruta absoluta del directorio donde se encuentra el usuario.

---

## Listar contenido

```bash
ls
```

Lista el contenido del directorio actual.

Para obtener información detallada:

```bash
ls -l
```

Se muestran:

* permisos;
* propietario;
* grupo;
* tamaño;
* fecha de modificación.

---

## Cambiar de directorio

Ir a la raíz:

```bash
cd /
```

Ir al directorio personal:

```bash
cd ~
```

---

# 🛡️ Análisis de Seguridad: El Directorio `/tmp`

En numerosos incidentes de seguridad, un atacante con acceso inicial puede utilizar `/tmp` para almacenar herramientas o archivos temporales.

La razón es que este directorio permite la creación de archivos por cualquier usuario sin privilegios administrativos.

Sin embargo, la posibilidad de ejecutar dichos archivos dependerá de la configuración del sistema. En entornos endurecidos (*hardened*), `/tmp` suele montarse con opciones como `noexec`, impidiendo la ejecución directa de binarios desde ese directorio.

Por esta razón, un analista SOC nunca debe asumir que todo archivo ubicado en `/tmp` es malicioso; debe analizar su contexto, propietario, permisos, procesos asociados y registros del sistema antes de emitir una conclusión.

---

# 🔒 Sticky Bit: Protección de Directorios Compartidos

Al ejecutar:

```bash
ls -ld /tmp
```

es habitual obtener una salida similar a:

```text
drwxrwxrwt
```

La letra **`t`** indica que el directorio posee activado el **Sticky Bit**.

Su función consiste en permitir que todos los usuarios creen archivos dentro del directorio, pero restringiendo el borrado y el renombrado únicamente al:

* propietario del archivo;
* propietario del directorio;
* usuario `root`.

Es importante comprender que el Sticky Bit **no protege el contenido interno de los archivos**. Si un archivo posee permisos de escritura inseguros, otro usuario podría modificarlo. Su protección se limita a impedir la eliminación o el cambio de nombre de los archivos dentro del directorio compartido.

Por ello, el Sticky Bit constituye una medida básica de hardening que debe complementarse con una correcta gestión de permisos, políticas de seguridad y mecanismos de auditoría.

---

# 📚 Conceptos Clave

* Linux organiza todo el sistema a partir de un único directorio raíz (`/`).
* El estándar **Filesystem Hierarchy Standard (FHS)** define la función de cada directorio principal.
* Comprender la estructura del sistema de archivos es fundamental para la administración y la ciberseguridad.
* Linux distingue estrictamente entre mayúsculas y minúsculas.
* La navegación mediante la terminal constituye la base del trabajo de administradores, analistas SOC e ingenieros de ciberseguridad.
* El directorio `/tmp` es un área compartida que requiere comprender tanto sus permisos como las medidas de protección asociadas, como el Sticky Bit.
