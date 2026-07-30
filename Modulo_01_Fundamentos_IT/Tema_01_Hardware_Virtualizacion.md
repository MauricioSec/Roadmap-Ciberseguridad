# Módulo 01 - Fundamentos IT
## Sesión 01 - Preparación del Laboratorio, Virtualización e Instalación de Ubuntu

**Fecha:** 30 de julio de 2026  
**Estado:** Completado ✅

---

# Objetivo de la sesión

Preparar un laboratorio local de ciberseguridad basado en virtualización, comprendiendo los fundamentos de los sistemas operativos, la arquitectura Host/Guest, las buenas prácticas de seguridad (Zero Trust) y el despliegue de una máquina virtual Linux completamente funcional.

---

# Objetivos alcanzados

- Comprender los conceptos fundamentales de virtualización.
- Verificar la compatibilidad del hardware con virtualización asistida por CPU.
- Preparar un laboratorio aislado utilizando Oracle VM VirtualBox.
- Implementar buenas prácticas de verificación criptográfica mediante SHA-256.
- Instalar Ubuntu Desktop como primer sistema operativo invitado.
- Resolver un incidente relacionado con la emulación gráfica durante la instalación.
- Implementar una estrategia de respaldo mediante Snapshots.
- Establecer una metodología de documentación técnica continua en GitHub.

---

# Fase 1: Diagnóstico Inicial

Antes de comenzar el laboratorio se realizó una evaluación del punto de partida.

## Resultado

Se determinó que aún no poseo conocimientos sólidos en:

- Redes
- Sistemas Operativos
- Linux
- Línea de comandos (CLI)

A partir de este diagnóstico se definió una ruta de aprendizaje progresiva desde los fundamentos hasta escenarios reales de ciberseguridad.

---

# Fase 2: Diseño del laboratorio

Debido a mi modalidad laboral (22 días embarcado y 20 días de descanso), se decidió construir un laboratorio completamente local.

## Arquitectura seleccionada

```
Windows 11 (Host)

        │

VirtualBox (Hipervisor Tipo 2)

        │

Ubuntu Desktop (Guest OS)

        │

CLI • Redes • Python • Ciberseguridad
```

Esta arquitectura permite continuar el aprendizaje incluso sin acceso a Internet.

---

# Política de aprendizaje

Se adoptó una política de inmersión gradual en inglés técnico.

En lugar de evitar documentación en inglés, cada nuevo concepto será incorporado a un glosario técnico personal dentro del repositorio.

---

# Fase 3: Virtualización

## Conceptos aprendidos

Durante esta sesión se estudiaron los siguientes conceptos:

- Virtualización
- Host Operating System
- Guest Operating System
- Hipervisor Tipo 1
- Hipervisor Tipo 2
- Snapshot

También se comprendió la diferencia arquitectónica entre un hipervisor Bare Metal y uno Hosted.

---

# Verificación de AMD-V

Se ingresó al firmware UEFI/BIOS del equipo.

## Equipo utilizado

**Modelo**

HP EliteBook 845 G8

## Resultado

Se verificó que la característica:

```
SVM (Secure Virtual Machine)
```

se encontraba habilitada.

Esto confirma que el procesador soporta virtualización por hardware (AMD-V), requisito indispensable para ejecutar sistemas operativos invitados de manera eficiente.

---

# Fase 4: Control de versiones y organización del proyecto

Se creó el repositorio público:

```
Roadmap-Ciberseguridad
```

Se adoptó una estructura modular para documentar cada módulo del programa.

También se identificó una limitación importante de Git:

> Git no almacena carpetas vacías.

Para resolverlo se creó directamente el archivo:

```
Modulo_01_Fundamentos_IT/README.md
```

desde la interfaz web de GitHub.

---

# Política de laboratorio

Se estableció como norma **no utilizar Windows Subsystem for Linux (WSL)** para prácticas de ciberseguridad.

## Motivos

- menor aislamiento
- ausencia de snapshots
- menor control del entorno
- arquitectura menos representativa de un laboratorio profesional

VirtualBox será la plataforma principal para todas las prácticas.

---

# Fase 5: Integridad del software (Zero Trust)

Antes de instalar Ubuntu se aplicó el principio de **Never Trust, Always Verify**.

## Procedimiento

1. Descarga de Ubuntu desde la página oficial de Canonical.
2. Obtención del hash SHA-256 oficial.
3. Verificación local mediante PowerShell.

Comando utilizado:

```powershell
Get-FileHash ubuntu-24.04.2-desktop-amd64.iso -Algorithm SHA256
```

## Incidente

Durante la primera ejecución se produjo un error de sintaxis debido a la omisión de un espacio entre el parámetro y el algoritmo.

Ejemplo incorrecto:

```powershell
-AlgorithmSHA256
```

Una vez corregido el comando, el hash generado coincidió exactamente con el publicado por Canonical.

## Resultado

La imagen ISO fue validada exitosamente.

Esto garantiza la integridad del archivo descargado y reduce el riesgo de ataques de cadena de suministro.

---

# Fase 6: Configuración de VirtualBox

Se creó la primera máquina virtual aplicando una política conservadora de asignación de recursos.

## Configuración

| Recurso | Valor |
|----------|------:|
| RAM | 2048 MB |
| Disco virtual | 25 GB |
| Tipo de disco | VDI |
| Asignación | Dinámica |
| Video | VBoxSVGA |
| VRAM | 128 MB |

La asignación de memoria se mantuvo dentro del límite recomendado para evitar degradar el rendimiento del sistema anfitrión.

---

# Incidente técnico

Durante el inicio del instalador apareció el siguiente error relacionado con el subsistema gráfico:

```
vmwgfx seems to be running on an unsupported hypervisor
```

El Kernel iniciaba correctamente, pero la interfaz gráfica quedaba bloqueada.

## Análisis

El problema estaba asociado al controlador gráfico emulado utilizado durante el arranque.

## Resolución

Se modificó la configuración de la máquina virtual:

- aumento de VRAM a 128 MB
- controlador gráfico VBoxSVGA

Después del cambio la instalación continuó normalmente.

---

# Instalación de Ubuntu

Finalizada la corrección del adaptador gráfico, Ubuntu Desktop fue instalado exitosamente.

El sistema inició correctamente y quedó listo para comenzar los laboratorios del siguiente módulo.

---

# Snapshot inicial

Se creó la primera instantánea del laboratorio.

**Nombre**

```
Instalación Limpia - Base
```

Este Snapshot permitirá regresar rápidamente a un estado estable antes de realizar futuras prácticas.

---

# Lecciones aprendidas

Durante esta sesión se reforzaron varios principios importantes.

## 1. Verificar antes de confiar

Toda descarga crítica debe validarse mediante mecanismos criptográficos.

---

## 2. Documentar inmediatamente

La documentación debe realizarse el mismo día del laboratorio para conservar el contexto técnico y las decisiones tomadas.

---

## 3. Comprender antes de automatizar

No es recomendable instalar herramientas sin comprender previamente su propósito y funcionamiento.

---

## 4. La precisión importa

En la línea de comandos un solo carácter puede producir errores.

La sintaxis debe revisarse cuidadosamente antes de ejecutar cualquier instrucción.

---

# Dificultades encontradas

- Tendencia a acelerar el proceso antes de comprender completamente los conceptos.
- Error de sintaxis en PowerShell durante la validación del hash.
- Necesidad de fortalecer el hábito de documentar inmediatamente después de cada práctica.

---

# Próximos pasos

En la siguiente sesión se abordarán:

- Arquitectura de Linux
- User Space
- Kernel Space
- Sistema de archivos Linux
- Terminal Bash
- Primeros comandos del sistema

---

# Herramientas utilizadas

- Windows 11
- HP EliteBook 845 G8
- Oracle VM VirtualBox
- Ubuntu Desktop 24.04 LTS
- PowerShell
- GitHub
- Git

---

# Conclusión

La primera sesión permitió construir un laboratorio de ciberseguridad funcional y reproducible, establecer una metodología de trabajo basada en documentación continua y aplicar prácticas de seguridad como la verificación criptográfica de software y el uso de snapshots. Además de completar la instalación del entorno Linux, se resolvió un incidente relacionado con la virtualización gráfica, reforzando la importancia del análisis, la validación y la resolución estructurada de problemas durante el proceso de aprendizaje.
