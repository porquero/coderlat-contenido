---
visible: true
title: 'Activar el modo DEBUG en EspoCRM'
date: '29-07-2026 16:16'
author: porquero
metadata:
    description: 'Cómo activar el modo DEBUG en EspoCRM desde data/config.php, filtrar el log con tail y grep, y volver a NOTICE al terminar el diagnóstico.'
faq:
    -
        q: '¿Cómo activo el modo DEBUG en EspoCRM?'
        a: 'Edita data/config.php y cambia el valor de ''level'' a ''DEBUG'' dentro de la clave ''logger''. No requiere reconstruir metadata ni reiniciar servicios: el log del día (data/logs/espo-AAAA-MM-DD.log) empieza a registrar todo el detalle.'
    -
        q: '¿Debo dejar DEBUG activo en producción?'
        a: 'No. Los archivos de log crecen mucho más rápido y las trazas pueden incluir información sensible. Devuelve el nivel a NOTICE o WARNING apenas termines de diagnosticar.'
    -
        q: '¿Cómo filtro el log de EspoCRM en vivo?'
        a: 'Combina tail -f con grep para acotar por el término que te interesa, por ejemplo: tail -f data/logs/espo-AAAA-MM-DD.log | grep Voip | grep -v DEBUG.'
sources:
    -
        title: 'Documentación oficial de logs de EspoCRM'
        url: 'https://docs.espocrm.com/administration/log'
taxonomy:
    tool:
        - espocrm
    resource:
        - tutorial
    country:
        - chile
sitemap:
    lastmod: '29-07-2026 16:16'
slug: activar-el-modo-debug-en-espocrm
---

Hay un momento muy específico en el que todo implementor de EspoCRM se topa con la misma pared: un flujo de trabajo falla, una fórmula no hace lo esperado, o un endpoint de la API responde con un error genérico — y el log, en su configuración por defecto, no dice nada útil. Por defecto, [EspoCRM](https://www.espocrm.com/) solo registra lo que considera relevante para operación normal, y eso deja fuera justo lo que necesitas para diagnosticar un problema puntual.

El modo DEBUG existe para ese momento. Activarlo no es una configuración que deba quedar activada  siempre — es una herramienta que se enciende cuando necesitas ver el detalle completo de lo que está pasando, y se apaga apenas terminas de diagnosticar. Entender esto evita dos errores comunes: dejar DEBUG activo en producción de forma permanente (los logs crecen rápido y pueden exponer datos sensibles), o nunca activarlo y quedarte adivinando qué falla.

## Cómo funciona el sistema de logs de EspoCRM

EspoCRM usa la librería [Monolog](https://github.com/Seldaek/monolog) para gestionar sus logs, y por defecto escribe en `{ESPO_ROOT}/data/logs/`. El nivel de detalle que se registra se controla con el parámetro `level`, que acepta cinco valores en orden creciente de severidad: `DEBUG`, `INFO`, `NOTICE`, `WARNING` y `ERROR`. Por defecto, EspoCRM viene configurado en `NOTICE` — suficiente para operación normal, insuficiente para investigar un problema específico.

Toda la configuración vive en `data/config.php`, bajo la clave `logger`:

```php
'logger' => [
    'level' => 'NOTICE', // DEBUG, INFO, NOTICE, WARNING, ERROR
    'maxFileNumber' => 30,
],
```

Bajar el nivel a `DEBUG` hace que el sistema registre absolutamente todo lo que ocurre internamente, incluyendo trazas que en operación normal serían ruido pero que, cuando estás cazando un bug, son exactamente lo que necesitas.

## Activar DEBUG

El cambio es directo: editar `data/config.php` y modificar el valor de `level`:

```php
'logger' => [
    'level' => 'DEBUG',
    'maxFileNumber' => 30,
],
```

Guardar el archivo es suficiente — no requiere reconstruir metadata ni reiniciar servicios. A partir de ese momento, el archivo de log del día (`data/logs/espo-AAAA-MM-DD.log`) empieza a registrar todo con el detalle completo.

Una práctica que vale la pena adoptar antes de reproducir el problema: renombrar o mover el log del día actual. Así, cuando reproduces la falla, el archivo nuevo contiene únicamente las líneas relacionadas con esa ejecución específica, sin mezclarse con el tráfico normal del resto del día. Esto es especialmente útil al diagnosticar procesos programados (workflows con cron) donde el log puede acumular cientos de líneas de otras tareas antes de llegar a la que te interesa.

## Generar tus propias trazas desde código custom

### Logging en código custom
Si estás desarrollando hooks, controllers o cualquier personalización en PHP, puedes escribir directamente al log de Espo usando el logger global del sistema:

```php
$GLOBALS['log']->debug('_LOG_' . __DIR__ . __FILE__ . __LINE__ . var_export($scope, true));
```

Este patrón — prefijo fijo, ruta del archivo, línea, y el contenido de la variable exportado con `var_export` — es simple pero efectivo: te dice exactamente dónde se generó la traza y qué contenía la variable en ese momento, sin necesidad de un debugger.

### Loggin con trazabilidad
Cuando el flujo que estás rastreando pasa por varios métodos o archivos y necesitas seguirle el rastro a una ejecución específica en medio de un log con mucho tráfico, generar un identificador corto al inicio y reutilizarlo en cada traza relacionada simplifica la búsqueda:

```php
$hashtra = substr(str_shuffle('ABCDEFGHJKLMNPQRSTUVWXYZ23456789'), 0, 12);

$GLOBALS['log']->debug('_LOG_' . $hashtra . __DIR__ . __FILE__ . __LINE__ . var_export($scope, true));

// resto del código

$GLOBALS['log']->debug('_LOG_' . $hashtra . __DIR__ . __FILE__ . __LINE__ . var_export($scope, true));
```

Buscar ese identificador en el log te da únicamente las líneas de esa ejecución puntual, incluso si el sistema procesó docenas de solicitudes similares en el mismo minuto.

### Archivo log único 

Si prefieres no mezclar tus trazas con el log general, también puedes escribir a un archivo propio junto al hook o script que estás depurando:

```php
file_put_contents(dirname(__FILE__) . '/dump.txt', var_export($variable, true));
```

## Depurar fórmulas, workflows y flowcharts

El logging por código cubre PHP, pero buena parte de la lógica de negocio en EspoCRM vive en fórmulas, workflows y flowcharts — donde no hay forma directa de insertar un `debug()`. Para esos casos, EspoCRM incorporó su propia librería de log para fórmulas, documentada en [docs.espocrm.com/administration/formula/log](https://docs.espocrm.com/administration/formula/log/), que permite volcar el valor de una expresión al log antes de que altere un registro o se asigne a un campo. Es la forma recomendada actualmente — versiones anteriores de la comunidad resolvían esto con extensiones de terceros, que hoy son innecesarias frente a la solución nativa.

## Filtrar el ruido al revisar el log en vivo

Con DEBUG activo, el archivo de log crece rápido, así que seguirlo en vivo sin filtrar se vuelve poco práctico. Combinar `tail -f` con `grep` para acotar por el término que te interesa (el nombre de una integración, un scope, tu identificador de rastreo) es la diferencia entre encontrar la línea relevante en segundos o desplazarte por miles de líneas:

```bash
tail -f data/logs/espo-2025-02-19.log | grep Voip | grep -v DEBUG
```

En este ejemplo, la última parte (`grep -v DEBUG`) excluye justamente las líneas de depuración cuando ya sabes qué estás buscando y quieres quedarte solo con eventos de nivel superior — útil una vez que ya usaste DEBUG para localizar el flujo y ahora quieres ver únicamente lo relevante de ese componente.

## Volver a la normalidad

Terminado el diagnóstico, el paso que más se olvida es revertir el nivel de log a `NOTICE` o `WARNING`. Dejar DEBUG activo de forma permanente en un ambiente de producción tiene dos costos reales: los archivos de log crecen mucho más rápido de lo normal, y las trazas de depuración pueden incluir datos de negocio sensibles que no deberían quedar acumulados en texto plano indefinidamente.

La disciplina completa es simple: activar DEBUG antes de reproducir el problema, renombrar el log del día para aislar la ejecución, revisar con `tail -f` y `grep` para llegar rápido a lo relevante, y devolver el nivel a su configuración original apenas termines. Ese ciclo, más que cualquier herramienta puntual, es lo que convierte un log opaco en información que realmente sirve para resolver.

**Referencia oficial:** [docs.espocrm.com/administration/log](https://docs.espocrm.com/administration/log)
