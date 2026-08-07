# 📁 Tema 3: Gestión de Permisos y Control de Acceso en Linux

> **Área:** Linux / Cybersecurity / Hardening  
> **Nivel:** Fundamentos – Administración Linux  
> **Entorno:** Ubuntu Guest OS · Oracle VM VirtualBox · Bash  
> **Estado:** Completado

---

## 🎯 Objetivo del Tema

Comprender y aplicar el modelo de control de acceso basado en permisos de Linux, utilizando el esquema **UGO (User, Group, Others)** y la notación simbólica y octal.

Al finalizar este tema se debe ser capaz de:

- Identificar el propietario y grupo asociado a un archivo.
- Interpretar correctamente la salida de `ls -l`.
- Comprender los permisos `Read`, `Write` y `Execute`.
- Calcular permisos utilizando notación octal.
- Modificar permisos mediante `chmod`.
- Comprender la función de `chown`.
- Diferenciar el comportamiento de los permisos en archivos y directorios.
- Identificar configuraciones de permisos inseguras.
- Comprender el riesgo asociado a archivos SUID.
- Aplicar el **Principio de Menor Privilegio (PoLP)**.
- Relacionar la administración de permisos con actividades de hardening, detección y respuesta ante incidentes.

---

# 🧠 1. Fundamentos de Identidad y Control de Acceso

Antes de analizar los permisos de un archivo es necesario conocer el contexto de ejecución del usuario. Linux utiliza identidades asociadas a:

- **UID (User ID):** identificador numérico del usuario.
- **GID (Group ID):** identificador numérico del grupo.
- **Groups:** grupos adicionales a los que pertenece el usuario.

Comandos básicos para consultar esta información:

```bash
whoami

```

Muestra el nombre del usuario actualmente utilizado.

```bash
id

```

Muestra información más completa, incluyendo UID, GID y grupos.

```bash
groups

```

Muestra los grupos a los que pertenece el usuario.

Estas herramientas son importantes durante la administración y el análisis de incidentes porque permiten determinar **con qué identidad se está ejecutando una sesión o proceso**.

---

# 👥 2. Modelo UGO: User, Group, Others

El sistema de permisos tradicional de Linux utiliza tres categorías principales:

| Categoría | Significado |
| --- | --- |
| `u` | User / propietario |
| `g` | Group / grupo propietario |
| `o` | Others / cualquier usuario que no sea el propietario ni pertenezca al grupo |

Linux determina qué conjunto de permisos debe aplicar según la identidad del proceso que intenta acceder al recurso. De forma simplificada:

```text
¿El usuario es el propietario?
       │
       Sí
       ↓
Aplicar permisos del User

       │ No
       ↓

¿Pertenece al grupo propietario?
       │
       Sí
       ↓
Aplicar permisos del Group

       │ No
       ↓

Aplicar permisos de Others

```

### Importante

Los permisos de `User`, `Group` y `Others` **no se suman entre sí**. Linux determina qué categoría corresponde y utiliza esa categoría para evaluar el acceso.

---

# 🔐 3. Matriz de Permisos

Al ejecutar:

```bash
ls -l archivo

```

Podemos obtener una salida similar a:

```text
-rwxr-x--- 1 usuario grupo 0 Aug 7 10:00 archivo

```

La primera columna contiene la información de tipo y permisos.

```text
-rwxr-x---
│├──┤├──┤├──┤
│    │    │
│    │    └── Others
│    └─────── Group
└──────────── User

```

La estructura puede dividirse de la siguiente manera:

* **Posición 1:** Tipo de archivo
* **Posiciones 2-4:** User
* **Posiciones 5-7:** Group
* **Posiciones 8-10:** Others

### Tipo de archivo

| Símbolo | Significado |
| --- | --- |
| `-` | Archivo regular |
| `d` | Directorio |
| `l` | Enlace simbólico |

---

# 📖 4. Permisos Read, Write y Execute

Linux utiliza tres permisos fundamentales:

| Permiso | Símbolo | Valor octal |
| --- | --- | --- |
| Read | `r` | 4 |
| Write | `w` | 2 |
| Execute | `x` | 1 |

Los valores se suman para obtener el valor octal.

### Ejemplos

* `r--` = 4
* `-w-` = 2
* `--x` = 1
* `rw-` = 4 + 2 = 6
* `r-x` = 4 + 1 = 5
* `rwx` = 4 + 2 + 1 = 7

---

# 📐 5. Notación Octal

Los permisos pueden representarse mediante tres números:

```text
USER | GROUP | OTHERS
  7  |   5   |   0

```

Por ejemplo, `750` significa:

* **User** → 7 = rwx
* **Group** → 5 = r-x
* **Others** → 0 = ---

Resultado: `-rwxr-x---`

---

# 📋 6. Permisos habituales

| Octal | Simbólico | Descripción |
| --- | --- | --- |
| `600` | `rw-------` | El propietario puede leer y escribir |
| `640` | `rw-r-----` | Propietario lee/escribe; grupo solo lee |
| `644` | `rw-r--r--` | Propietario lee/escribe; otros solo leen |
| `700` | `rwx------` | Solo el propietario tiene acceso completo |
| `750` | `rwxr-x---` | Propietario completo; grupo lee/ejecuta |
| `755` | `rwxr-xr-x` | Propietario completo; grupo y otros leen/ejecutan |
| `777` | `rwxrwxrwx` | Todos tienen acceso completo |

Los permisos apropiados dependen del propósito del recurso. No existe un único conjunto de permisos "correcto" para todos los archivos.

---

# 📂 7. Diferencia entre Archivos y Directorios

El significado de `r`, `w` y `x` cambia según el tipo de objeto.

## Archivos

| Permiso | Función |
| --- | --- |
| `r` | Permite leer el contenido |
| `w` | Permite modificar el contenido |
| `x` | Permite ejecutar el archivo si su formato y contexto lo permiten |

## Directorios

| Permiso | Función |
| --- | --- |
| `r` | Permite listar las entradas del directorio |
| `w` | Permite crear, eliminar o renombrar entradas cuando también se cumplen las condiciones necesarias de acceso |
| `x` | Permite atravesar el directorio y acceder a sus entradas |

### Punto crítico

El permiso `x` en un directorio **no significa ejecutar el directorio**. Significa que el usuario puede **atravesarlo (traverse)** y acceder a elementos dentro de él cuando los demás permisos requeridos están disponibles.

---

# ⚠️ 8. Caso Especial: Directorio con Permisos 666

Un directorio con `drw-rw-rw-` tiene permisos octales `666`. El problema es que carece del permiso `x`.

Por lo tanto, un usuario puede encontrar dificultades para acceder al directorio mediante:

```bash
cd directorio

```

Y recibir el error:

```text
Permission denied

```

Esto ocurre porque `x` es necesario para atravesar el directorio. Este ejemplo demuestra que no se puede interpretar `r`, `w` y `x` de la misma manera para archivos y directorios.

---

# 🛠️ 9. Herramientas CLI Fundamentales

## `ls`

Permite listar archivos y directorios.

```bash
ls
ls -l
ls -ld directorio

```

## `chmod`

`chmod` significa **Change Mode**. Permite modificar los permisos de un archivo o directorio.

**Notación octal:**

```bash
chmod 640 archivo

```

**Notación simbólica:**

```bash
chmod u+x script.sh
chmod g+w archivo
chmod o-r archivo
chmod go-rwx archivo

```

Donde: `u = User`, `g = Group`, `o = Others`, `a = All`.

## `chown`

`chown` significa **Change Owner**. Permite modificar el propietario y/o grupo asociado a un archivo. (Normalmente requiere privilegios administrativos).

```bash
sudo chown usuario archivo
sudo chown usuario:grupo archivo

```

## `stat`

`stat` permite consultar metadatos detallados de un archivo.

```bash
stat archivo

```

Puede mostrar información como: Tamaño, Inodo, Permisos, UID, GID, Access time (`atime`), Modify time (`mtime`) y Change time (`ctime`). Es útil tanto para administración como para troubleshooting y análisis forense.

---

# 🛡️ 10. Principio de Menor Privilegio

El **Principle of Least Privilege (PoLP)** establece que un usuario, proceso o servicio debe recibir únicamente los privilegios necesarios para realizar su función. No se deben conceder permisos adicionales sin una justificación operacional.

Ejemplo:

```text
Incorrecto: backup.sh → 777
Mejor: backup.sh → permisos mínimos necesarios

```

El objetivo es reducir la superficie de ataque y limitar el impacto de una eventual explotación.

---

# 🚨 11. Riesgo de `chmod 777`

Un archivo `-rwxrwxrwx` permite al propietario, al grupo y a otros usuarios leer, escribir y ejecutar. Esto puede convertirse en un riesgo importante cuando el archivo es utilizado por un proceso privilegiado.

### Escenario

Supongamos `backup.sh` con `-rwxrwxrwx` ejecutado periódicamente por `root`. Un usuario sin privilegios podría modificar el contenido del script. Si posteriormente `root` ejecuta el archivo, las instrucciones modificadas se ejecutarán con privilegios de root.

Flujo conceptual:

```text
Usuario sin privilegios
       │
       ↓
Modifica backup.sh
       │
       ↓
backup.sh pertenece a un proceso privilegiado
       │
       ↓
root ejecuta el script
       │
       ↓
Código alterado se ejecuta como root
       │
       ↓
Escalada de privilegios

```

Por esta razón, los scripts y archivos utilizados por procesos privilegiados deben tener permisos cuidadosamente restringidos.

---

# 🔎 12. SUID y Escalada de Privilegios

Linux dispone de permisos especiales además de `rwx`. Uno de ellos es **SUID (Set User ID)**. Un archivo con SUID puede ejecutarse utilizando la identidad efectiva del propietario del archivo.

Ejemplo:

```text
-rwsr-xr-x

```

La `s` en la posición correspondiente al permiso de ejecución del propietario indica que el bit SUID está activado. Si el archivo pertenece a `root`, el comportamiento puede provocar que el proceso se ejecute con privilegios efectivos de root.

### ¿Por qué es importante para ciberseguridad?

Un binario SUID propiedad de root puede convertirse en un objetivo de alto valor si contiene una vulnerabilidad explotable, presenta una configuración insegura, permite ejecutar funcionalidades peligrosas o puede ser manipulado indirectamente. Un atacante que consiga abusar de una vulnerabilidad de este tipo podría obtener una **escalada de privilegios**.

### Perspectiva SOC

Durante una investigación, la aparición inesperada de un nuevo binario SUID, o un cambio sospechoso en un binario existente, puede constituir un indicador que merece investigación. No todo SUID es malicioso; el análisis debe determinar si el archivo es esperado, legítimo y correctamente configurado.

---

# 🧪 13. Laboratorio Práctico

### 13.1 Preparación

Verificar la identidad, UID, GID y ubicación:

```bash
whoami
id
groups
cd ~
pwd

```

### 13.2 Crear archivo de prueba

```bash
touch informe.txt
ls -l informe.txt

```

### 13.3 Modificar permisos mediante notación simbólica

Eliminar todos los permisos del grupo y de otros:

```bash
chmod go-rwx informe.txt
ls -l informe.txt

```

Resultado esperado: `-rw------- 1 usuario usuario 0 Aug 7 10:01 informe.txt`

### 13.4 Modificar permisos mediante notación octal

Establecer permisos `rw-r-----` (640):

```bash
chmod 640 informe.txt
ls -l informe.txt

```

Resultado esperado: `-rw-r----- 1 usuario usuario 0 Aug 7 10:02 informe.txt`

### 13.5 Analizar los metadatos

```bash
stat informe.txt

```

---

# 🧪 14. Ejercicio Individual: Directorio de Auditoría

Crear el directorio y el archivo:

```bash
mkdir auditoria
cd auditoria
touch evidencia.log

```

El objetivo es establecer control completo para el usuario (7), lectura y ejecución para el grupo (5), y sin acceso para otros (0).

```bash
chmod 750 evidencia.log
ls -l evidencia.log

```

Resultado esperado: `-rwxr-x--- 1 usuario usuario 0 Aug 7 10:05 evidencia.log`

---

# 🔬 15. Verificación del Laboratorio

La práctica debe seguir el siguiente ciclo:

```text
Ejecutar → Observar → Interpretar → Verificar → Documentar

```

La salida obtenida debe compararse con el resultado esperado y debe poder explicarse técnicamente.

---

# 🔐 16. Relación con Ciberseguridad

La gestión de permisos está directamente relacionada con:

* **Hardening:** Reducir configuraciones excesivamente permisivas.
* **Escalada de privilegios:** Evitar que un usuario modifique recursos de procesos privilegiados.
* **Persistencia:** Evitar alteraciones en scripts de inicio del sistema.
* **Protección de información:** Bloquear accesos a información sensible.
* **SOC:** Investigar cambios inesperados de permisos, nuevos archivos, scripts modificados, binarios SUID inesperados y cambios de propietario.

---

# 🚨 17. Indicadores que Merecen Investigación

Eventos que justifican análisis en un entorno de seguridad:

```text
Archivo crítico → cambio inesperado de propietario
Archivo crítico → permisos excesivamente permisivos
Nuevo binario → SUID activado
Script privilegiado → permisos 777
Archivo del sistema → modificación inesperada
Servicio → ejecuta un archivo modificable por usuarios no privilegiados

```

---

# ❓ 18. Preguntas de Comprobación

### Directorio con permisos 666

**Pregunta:** Si un directorio tiene `drw-rw-rw-`, ¿qué problema técnico enfrentará un usuario al intentar ejecutar `cd`?

**Respuesta:** Recibirá `Permission denied` porque el directorio carece del permiso `x` (atravesar/traverse). Disponer de `r` y `w` no sustituye la ausencia de `x`.

### SUID propiedad de root

**Pregunta:** ¿Por qué un binario SUID propiedad de root representa un objetivo importante para un atacante?

**Respuesta:** Porque puede ejecutarse utilizando la identidad efectiva de su propietario (root). Una vulnerabilidad en ese programa podría permitir ejecutar operaciones con privilegios elevados.

---

# 🧠 19. Lecciones Aprendidas

* **Identidad:** Antes de analizar permisos es necesario conocer el usuario, UID, GID y grupos involucrados.
* **UGO:** Linux determina permisos basados en User, Group y Others.
* **Notación octal:** `r = 4`, `w = 2`, `x = 1`.
* **Archivos vs. directorios:** El significado de `x` cambia significativamente.
* **Principio de Menor Privilegio:** Los permisos deben limitarse a las necesidades operacionales.
* **Seguridad:** Configuraciones permisivas facilitan modificaciones no autorizadas, exposición, ejecución de código y escalada de privilegios.

---

# 📌 20. Conclusión

La administración de permisos constituye uno de los fundamentos de seguridad de Linux. Comprender esta arquitectura proporciona la base para la administración segura, hardening, detección de amenazas, respuesta ante incidentes y análisis forense. El objetivo es desarrollar la capacidad de responder tres preguntas ante cualquier recurso:

1. ¿Quién puede acceder?
2. ¿Qué puede hacer?
3. ¿Por qué tiene esos permisos?

---

# 📚 21. Referencias

* Linux manual pages: `man chmod`, `man chown`, `man ls`, `man stat`, `man id`, `man hier`.
* Linux Filesystem Hierarchy Standard (FHS).
* CIS Benchmarks para sistemas Linux.

---

## 📝 Registro del Laboratorio

| Campo | Información |
| --- | --- |
| **Módulo** | Módulo 1 – Fundamental IT Skills |
| **Tema** | Tema 3 – Gestión de Permisos y Control de Acceso en Linux |
| **Entorno** | Ubuntu Guest OS |
| **Hipervisor** | Oracle VM VirtualBox |
| **Shell** | Bash |
| **Nivel** | Fundamentos / Administración Linux |
| **Área relacionada** | Cybersecurity / SOC / Hardening |
| **Estado** | Completado |
