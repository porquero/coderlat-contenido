---
visible: true
title: 'Habilitar cambio de tabs remapeando teclado con ñ'
media_order: 'habilitar-cambio-de-tabs-remapeando-teclado-con-n.webm,remapeando-teclado-con-n.png'
date: '30-06-2026 16:23'
author: porquero
metadata:
    description: 'Habilita Ctrl+´ y Ctrl+ñ en EspoCRM con un script custom para cambiar de tab y buscar en listas usando un teclado en español.'
faq:
    -
        q: '¿Por qué no funciona CTRL + \ para cambiar de tab en EspoCRM con teclado en español?'
        a: 'EspoCRM usa e.code (la posición física de la tecla en un teclado US ANSI) en lugar del carácter producido. En teclados con ñ esas teclas están en otra posición, y el atajo nunca se dispara.'
    -
        q: '¿Qué atajos habilita este script?'
        a: 'CTRL + ´ salta entre tabs en la vista de detalle o edición de un registro, y CTRL + ñ lleva al campo de búsqueda en la vista de lista.'
taxonomy:
    tag:
        - espocrm
    tool:
        - espocrm
    resource:
        - tutorial
    country:
        - chile
sitemap:
    lastmod: '07-07-2026 12:46'
---

En teclados que están en español o que tienen ñ no es posible usar **CTRL +  \\** para intercambiar entre tabs de un registro en EspoCRM.

===

---
<video width="100%" controls preload="metadata" title="Demostración de CTRL + ´ y CTRL + ñ en EspoCRM" aria-label="Demostración de CTRL + ´ y CTRL + ñ en EspoCRM">
  <source src="/user/pages/02.blog/habilitar-cambio-de-tabs-remapeando-teclado-con-n/habilitar-cambio-de-tabs-remapeando-teclado-con-n.webm" type="video/webm">
  Tu navegador no soporta el formato de video.
</video>





---

El siguiente custom script habilita las combinaciones:

* CTRL + ´: Salto entre tabs en vista detalle/edición de un registro.
* CTRL + ñ: Ir a campo de búsqueda en vista lista de registros.

## Implementación

1. Crear o modificar _custom/Espo/Custom/Resources/metadata/app/client.json_ y agregar el llamado al script custom:

[codesh=json]
"scriptList": [
        "__APPEND__",
        "client/custom/src/keyboard-es-fix.js"
    ]
[/codesh]

2. Crear _/var/www/html/autofocus/client/custom/src/keyboard-es-fix.js_

[codesh=javascript]
/**
 * keyboard-es-fix.js
 *
 * Fixes EspoCRM keyboard shortcuts for Spanish keyboard layouts.
 *
 * Root cause: EspoCRM's ShortcutManager uses e.code (physical key position on a
 * US ANSI keyboard) instead of e.key (the actual character produced). On Spanish
 * keyboards several characters occupy different physical keys, so the shortcuts
 * Ctrl+/ and Ctrl+\ never fire as EspoCRM expects.
 *
 * Strategy: map ergonomic Ctrl+{digit/key} combos directly to the target shortcut
 * using e.code, without requiring Shift. This keeps shortcuts to 2 keys.
 *
 * Por qué Ctrl+7 no funciona:
 *   Los browsers (Chrome, Firefox) interceptan Ctrl+1..8 para cambiar de
 *   pestaña del browser. El evento de teclado nunca llega al JavaScript.
 *
 * Remappings aplicados (teclado QWERTY español):
 *
 *   Ctrl + Ñ  →  foco barra de búsqueda   (Ñ tiene e.code='Semicolon')
 *   Ctrl + '  →  cambiar tab en detalle   (´ tiene e.code='Quote')
 *
 *   Estas combinaciones no las usa ningún browser ni EspoCRM → seguras.
 *
 * Shortcuts que ya funcionan en teclado español (no necesitan fix):
 *   Ctrl + ,   →  filtro anterior   (EspoCRM usa Comma, no '<')
 *   Ctrl + .   →  filtro siguiente  (EspoCRM usa Period, no '>')
 *   Ctrl + Enter, Space, S, flechas, Delete
 */
(function () {
    'use strict';

    if (typeof Espo === 'undefined' || typeof Espo.Utils === 'undefined') {
        return;
    }

    /**
     * Code-based remappings: Ctrl+{código físico} → shortcut EspoCRM.
     *
     * Usamos e.code (posición física) para evitar depender de qué carácter
     * produce la tecla con o sin Shift. Estas combinaciones no las usa
     * ningún browser ni el sistema operativo.
     *
     * Teclado QWERTY español:
     *   'Semicolon' = tecla Ñ       → reemplaza Ctrl+/ (foco búsqueda)
     *   'Quote'     = tecla ´ / '   → reemplaza Ctrl+\ (cambiar tab detalle)
     */
    const CODE_REMAPS = {
        'Semicolon': 'Slash',        // Ctrl+Ñ  → foco barra de búsqueda
        'Quote':     'Backslash',    // Ctrl+´   → cambiar tab en vista detalle
    };

    const originalFn = Espo.Utils.getKeyFromKeyEvent;
    const isMac = (typeof IS_MAC !== 'undefined') ? IS_MAC : false;

    Espo.Utils.getKeyFromKeyEvent = function (e) {
        const isCtrl = isMac ? e.metaKey : e.ctrlKey;

        if (isCtrl && !e.shiftKey && !e.altKey && CODE_REMAPS[e.code]) {
            return 'Control+' + CODE_REMAPS[e.code];
        }

        return originalFn.call(this, e);
    };

})();
[/codesh]

3. Limpiar caché y probar.