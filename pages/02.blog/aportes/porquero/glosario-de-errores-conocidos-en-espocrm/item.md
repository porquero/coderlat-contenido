---
visible: true
title: 'Glosario de errores conocidos en EspoCRM'
date: '20-07-2026 15:01'
author: porquero
metadata:
    description: 'Glosario de errores de EspoCRM (403, Error 500, EspoClient Unknown Error) con su causa probable y cómo resolverlos: permisos, correo, base de datos y servidor.'
faq:
    -
        q: '¿Por qué recibo un error 403 en EspoCRM?'
        a: 'Casi siempre el rol del usuario no tiene habilitado el permiso para esa acción. Si ocurre al importar registros y los permisos están bien, puede ser una regla de ModSecurity (OWASP) bloqueando la petición en servidores cPanel o WHM.'
    -
        q: '¿Qué significa «EspoClient: Unknown Error»?'
        a: 'Casi siempre indica que el usuario de API no tiene permiso sobre alguna de las entidades que la llamada está tocando.'
    -
        q: '¿Qué revisar antes de sospechar de un bug de EspoCRM?'
        a: 'Los permisos del rol sobre la entidad o campo involucrado, el .htaccess en la raíz de la instalación y las reglas de ModSecurity del servidor.'
sources:
    -
        title: 'Troubleshooting oficial de EspoCRM'
        url: 'https://docs.espocrm.com/administration/troubleshooting/'
taxonomy:
    tag:
        - espocrm
    tool:
        - espocrm
    resource:
        - glosario
    country:
        - chile
sitemap:
    lastmod: '20-07-2026 15:01'
---

Cualquiera que haya operado EspoCRM por un tiempo conoce esa sensación: un mensaje de error en rojo, tres líneas de texto en inglés, y ninguna pista real de qué lo causó. `Error 500`. `Spoofing not allowed!`. `EspoClient: Unknown Error`.

Lo que hemos aprendido operando EspoCRM en distintos entornos — hosting compartido, servidores propios, con y sin extensiones de terceros — es que el mismo mensaje puede tener causas completamente distintas según el contexto. Un 403 puede ser un permiso mal configurado en un rol, o puede ser una regla de seguridad del servidor. Diagnosticar bien empieza por reconocer que el mensaje de error es apenas el punto de partida.

Este artículo reúne los errores que más veces nos ha tocado resolver, agrupados por dónde realmente se originan: permisos, correo, base de datos, automatización y servidor. La idea es que cuando veas uno de estos mensajes, no tengas que empezar desde cero.

## Permisos y rol

La mayoría de los errores 403 en EspoCRM tienen el mismo origen: el rol asignado al usuario no tiene el permiso habilitado para la acción que está intentando ejecutar. Antes de pensar en configuración de servidor o en un bug del sistema, vale la pena revisar la sección de asignación de permisos del rol correspondiente. Es la solución más simple y, sin embargo, la que más se pasa por alto.

Cuando ese 403 aparece específicamente al importar registros y ya confirmaste que los permisos están bien, el sospechoso cambia. En servidores cPanel o WHM con ModSecurity activo, una regla de OWASP puede estar bloqueando la petición sin que EspoCRM tenga nada que ver. La solución ahí pasa por entrar a ModSecurity Tools en WHM y deshabilitar el vendor OWASP para el dominio, o crear una excepción puntual para la ruta de importación.

Algo parecido ocurre con el temido `EspoClient: Unknown Error` al trabajar con la API: casi siempre significa que el usuario de API no tiene permiso sobre alguna de las entidades que la llamada está tocando. Y si te ha pasado que una vista simplemente no carga y la consola del navegador muestra algo como `Cannot destructure property 'optionsPath'`, la causa suele ser la misma familia de problema — el usuario no tiene permiso de lectura sobre un campo foráneo que la vista necesita renderizar. En los tres casos, la solución vive en el mismo lugar: **el rol**.

## Correo y autenticación

El correo en EspoCRM tiene sus propias trampas, y casi todas se resuelven con paciencia más que con conocimiento profundo del sistema.

Si al enviar correos masivos te encuentras con `Spoofing not allowed!`, el sistema está detectando que la dirección FROM no coincide con la cuenta autenticada — y el responsable casi siempre es la opción VERP activada en Administración → Salientes. Desactivarla resuelve el problema en la mayoría de los casos.

Cuando el error es `Incorrect authentication data`, conviene ir probando en orden en lugar de asumir: primero el tipo de conexión (TLS o SSL, según corresponda), después si el proveedor de correo tiene alguna restricción de seguridad activa, y por último si la contraseña sigue siendo válida. Un `OpenSSL decrypt failure` normalmente aparece después de una migración o un cambio en la clave de encriptación del sistema, y se resuelve simplemente volviendo a ingresar y guardar la contraseña de la cuenta.

Hay un caso particular que vale la pena conocer de antemano: si un usuario tiene doble factor de autenticación activado y luego cambia su correo, el sistema sigue enviando el código de verificación al correo anterior — el mensaje que verás es `Email address is not one of user's`. La solución tiene tres pasos: desactivar el doble factor, actualizar el correo, y reactivarlo. Recién ahí el sistema vuelve a capturar el correo vigente.

## Base de datos y metadata

Esta categoría agrupa los errores más difíciles de diagnosticar, porque el síntoma aparece en la interfaz pero la causa está en cómo se configuró la entidad.

Desde la [versión 7.2 de EspoCRM](https://github.com/espocrm/espocrm/releases?page=10#release-7.2.0), el sistema empezó a validar en el backend que un campo tipo lista (enum) solo acepte valores que estén en su listado de opciones. Si necesitas guardar valores fuera de ese listado cerrado, la solución es cambiar el tipo del campo de `enum` a `varchar` en el entityDefs de custom, agregando un `maxLength` para no perder el control de longitud.

Un caso más incómodo: entidades con muchos campos que, al intentar agregar uno más, arrojan `Error while rebuilding database` con un `Row size too large` en el log. Ajustar los parámetros de InnoDB que sugieren la mayoría de los foros no resuelve nada — el problema real es que el motor de base de datos tiene un límite de tamaño de fila, y entidades con muchos `varchar` lo acumulan sin que sea evidente. La solución práctica es convertir campos existentes a tipo `text` cuando no necesitan indexación y limitar el `maxLength` de los enum para que no se guarden con 255 caracteres por defecto. Si la entidad sigue creciendo, la solución que escala de verdad es mover los campos nuevos a una entidad relacionada.

También puede aparecer un `Error 500 No 'foreignMidKey' parameter defined in the relation` al filtrar sobre una relación uno-a-muchos donde el valor vacío no es reconocible por la función nativa de selección de EspoCRM. Este caso requiere extender un archivo específico del núcleo (`ItemGeneralConverter.php`) y luego reconstruir y limpiar caché.

Y si alguna vez editaste un JSON de custom a mano y terminaste con un error 500 sin ningún detalle útil en los logs de Apache o PHP-FPM, corre una validación de todos los `.json` bajo el directorio custom con `jq`; en segundos vas a saber exactamente cuál archivo tiene el problema de formato.

## Automatización y cron

Si trabajas con fórmulas en EspoCRM, tarde o temprano te vas a encontrar con `Before-save formula script failed: Could not handle 'order'` al usar funciones como `findRelatedMany` o `findRelatedOne`. La documentación oficial dice que el parámetro `order_by` es opcional — en la práctica, omitirlo produce el error igual. La solución es simplemente incluir siempre un criterio de orden explícito, por ejemplo `createdAt` con una dirección definida.

Cuando un flujo de trabajo programado aparece con estado "Failed", el método de diagnóstico es reproducir el problema fuera de EspoCRM: copiar el comando cron real de la instancia, ejecutarlo manualmente y revisar el log del día en `data/logs` (activando el modo DEBUG si no aparece nada). Casi siempre el log señala directamente una expresión CRON mal formada.

Y si una actualización por línea de comandos se corta por límite de memoria, no hay que investigar mucho: basta con ejecutar el comando fijando explícitamente el límite necesario, por ejemplo `php -d memory_limit=1024M command.php upgrade`.

## Errores de Servidor

Algunos de los problemas no vienen de EspoCRM, sino de la capa de servidor sobre la que corre.

Ante cualquier error 500 genérico después de un cambio de configuración, la disciplina que mejor funciona es aplicar un cambio a la vez, probar, y mantener un registro de lo modificado para poder revertir sin adivinar. El primer sospechoso, casi siempre, es el `.htaccess` — específicamente la opción `RewriteOptions inherit`, que suele venir heredada de una configuración anterior y es fácil de pasar por alto.

Si el mensaje es directamente `You need to configure your webserver in order to being able to run EspoCRM`, significa que el `.htaccess` en la raíz del dominio falta o está corrupto. La solución es recrearlo con las reglas de reescritura estándar de EspoCRM.

En hosting con Inmunify360 activo, un error 413 al subir archivos adjuntos suele ser una regla de ModSecurity bloqueando peticiones que superan cierto tamaño. Se resuelve ubicando la regla específica en WHM (ModSecurity Tools, filtrando por el código 413) y creando una excepción puntual para la ruta afectada, típicamente `/api/v1/Attachment`.

Fuera del ámbito web, si trabajas con integración VoIP y ves un error 21210 indicando que un número de origen "no está verificado", vale la pena revisar el proveedor configurado antes que la cuenta: es un síntoma clásico de que el número quedó apuntando a Twilio cuando la instalación en realidad opera sobre Asterisk.

## Antes de escalar cualquier error

Si tuviéramos que resumir este glosario en una sola idea: la mayoría de los errores de EspoCRM se resuelven revisando tres cosas antes de sospechar de un bug del núcleo — los permisos del rol sobre la entidad o campo involucrado, el `.htaccess` en la raíz de la instalación, y las reglas de ModSecurity u OWASP si el hosting corre sobre AWS, Google, Azure o cPanel/WHM. Empezar por ahí evita horas de búsqueda en la dirección equivocada.

Este glosario va a seguir creciendo con los casos que resuelva la comunidad. Si te topaste con un error que no está aquí, cuéntanoslo en nuestro [foro em Discourse](http://coderlat.discourse.group/) o en nuestro [grupo de WhatsApp](https://chat.whatsapp.com/Fa2odXpwZq6CgiobMGmVjm) — la próxima persona que lo busque en Google te lo va a agradecer.

Otros problemas y soluciones publicados por EspoCRM: [https://docs.espocrm.com/administration/troubleshooting/](https://docs.espocrm.com/administration/troubleshooting/)
