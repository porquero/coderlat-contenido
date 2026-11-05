---
visible: true
title: 'Kanboard: gestión de proyectos Kanban open source y autoalojada'
media_order: kanboard.png
date: '14-10-2026 12:00'
author: porquero
hero_image: kanboard.png
taxonomy:
    tool:
        - kanboard
    resource:
        - tutorial
    country:
        - chile
metadata:
    description: 'Qué es Kanboard, cuándo conviene usarlo y cómo levantarlo con Docker en minutos: un tablero Kanban gratuito, ligero y bajo tu control.'
faq:
    -
        q: '¿Qué es Kanboard?'
        a: 'Es un software de gestión de proyectos Kanban gratuito y de código abierto (licencia MIT) que instalas en tu propio servidor. Organiza el trabajo en tableros con columnas, tareas y swimlanes.'
    -
        q: '¿Kanboard es gratis?'
        a: 'Sí. No tiene planes de pago ni versión en la nube oficial: el código es libre y lo alojas tú, así que el único costo es el del servidor.'
    -
        q: '¿Qué base de datos usa Kanboard?'
        a: 'Funciona por defecto con SQLite, sin configuración adicional, y también admite MySQL/MariaDB y PostgreSQL para equipos más grandes.'
    -
        q: '¿Cuál es el usuario inicial de Kanboard?'
        a: 'La instalación nueva crea el usuario admin con contraseña admin. Cámbiala en el primer inicio de sesión, antes de exponer el sitio a internet.'
sources:
    -
        title: 'Sitio oficial de Kanboard'
        url: 'https://kanboard.org/'
    -
        title: 'Documentación de Kanboard'
        url: 'https://docs.kanboard.org/'
    -
        title: 'Repositorio en GitHub'
        url: 'https://github.com/kanboard/kanboard'
---

**Kanboard** es un tablero Kanban gratuito y de código abierto que instalas en tu propio servidor. Es ligero, no exige cuentas externas ni suscripciones y sirve para ordenar el trabajo de un equipo pequeño sin la complejidad de herramientas pesadas.

===

![Tablero Kanban de Kanboard con columnas y tareas](kanboard.png "Kanboard")

## Qué es Kanboard

Kanboard aplica el método Kanban: cada proyecto es un tablero con columnas (por ejemplo, *Pendiente*, *En curso*, *Hecho*) y cada tarea es una tarjeta que avanza de izquierda a derecha. Está escrito en PHP, se distribuye bajo licencia MIT y su filosofía es la simplicidad: poco consumo de recursos y una interfaz que se aprende en minutos.

## Qué incluye

- **Tableros Kanban** con columnas personalizables y **swimlanes** para separar equipos, clientes o prioridades.
- **Límites de trabajo en curso (WIP)** por columna, para evitar saturar al equipo.
- **Subtareas, comentarios, adjuntos y seguimiento de tiempo** en cada tarjeta.
- **Acciones automáticas**: por ejemplo, asignar o cerrar una tarea cuando cambia de columna.
- **Plugins** para ampliar funciones (integraciones, notificaciones, temas).
- **API JSON-RPC** para automatizar desde otros sistemas.

## Cuándo conviene usarlo

Kanboard encaja bien cuando quieres **control total de tus datos**, necesitas algo que corra en un servidor modesto, o buscas una alternativa sencilla a herramientas SaaS como Trello. Si tu equipo requiere diagramas de Gantt avanzados, reportes financieros o gestión de portafolios, probablemente necesites una herramienta más completa.

## Cómo levantarlo con Docker

Con Docker, tienes Kanboard funcionando en un par de comandos:

```bash
docker run -d --name kanboard -p 8080:80 \
  -v kanboard_data:/var/www/app/data \
  -v kanboard_plugins:/var/www/app/plugins \
  kanboard/kanboard:latest
```

Luego abre `http://localhost:8080` e inicia sesión con el usuario `admin` y contraseña `admin`. **Cambia esa contraseña de inmediato** desde tu perfil. Los volúmenes `kanboard_data` y `kanboard_plugins` conservan tus datos y plugins cuando actualices la imagen.

## Primeros pasos recomendados

1. Crea un proyecto y define tus columnas según cómo trabaja tu equipo realmente.
2. Agrega usuarios y asígnales rol (administrador, gestor de proyecto o miembro).
3. Configura un límite WIP en las columnas donde se acumula el trabajo.
4. Revisa las acciones automáticas para eliminar tareas manuales repetitivas.

## Recursos

Sitio, documentación y código fuente están enlazados al final de este artículo. Si lo pruebas en tu empresa, cuéntanos tu caso en los comentarios.
