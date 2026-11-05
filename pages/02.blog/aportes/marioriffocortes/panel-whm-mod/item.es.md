---
visible: true
title: 'Panel WHM en PHP para gestionar cuentas cPanel'
date: '14-10-2026 16:30'
author: MarioRiffoCortes
taxonomy:
    tool: [whm]
    resource: [tutorial]
    country: [chile]
metadata:
    description: 'Crea un panel PHP que lista, suspende y cambia claves de cuentas cPanel en varios servidores WHM vía API, con caché diaria y validación de DNS en Nic.cl.'
faq:
    -
        q: '¿Necesito acceso root para usar la API de WHM?'
        a: 'Necesitas un usuario root o reseller con un API Token creado en WHM (Development » Manage API Tokens). Un reseller solo verá y podrá modificar sus propias cuentas.'
    -
        q: '¿Por qué validar los DNS con el WHOIS de Nic.cl y no con dns_get_record()?'
        a: 'dns_get_record() pregunta a los servidores de nombres del propio dominio, que pueden seguir respondiendo con NS antiguos. El WHOIS de whois.nic.cl devuelve lo que está registrado en NIC Chile, que es lo que de verdad manda.'
    -
        q: '¿Cada cuánto se actualiza el listado de cuentas?'
        a: 'Una vez al día por servidor: la primera consulta del día llama a WHM y guarda un JSON en caché. El botón de recarga fuerza una nueva consulta con el parámetro force=1.'
    -
        q: '¿Puedo usarlo con dominios con tildes o ñ?'
        a: 'Sí. WHM devuelve esos dominios en Punycode (xn--) y el panel los convierte a UTF-8 para mostrarlos, y de vuelta a ASCII antes de consultar la API o el WHOIS.'
sources:
    -
        title: 'Repositorio mod-whm (código completo)'
        url: 'https://github.com/MarioRiffoCortes/mod-whm'
    -
        title: 'cPanel API: Return cPanel accounts (listaccts)'
        url: 'https://api.docs.cpanel.net/specifications/whm.openapi/account-management/accounts-listaccts'
    -
        title: 'cPanel Docs: Manage API Tokens in WHM'
        url: 'https://docs.cpanel.net/whm/development/manage-api-tokens-in-whm/'
    -
        title: 'PHP: idn_to_utf8'
        url: 'https://www.php.net/manual/es/function.idn-to-utf8.php'
    -
        title: 'NIC Chile'
        url: 'https://www.nic.cl/'
---

Si administras varios servidores WHM, puedes centralizar la gestión de cuentas cPanel en un panel PHP propio que habla con la **API 1 de WHM** usando un API Token por servidor. Con tres llamadas (`listaccts`, `suspendacct`/`unsuspendacct`/`removeacct` y `passwd`) obtienes el listado, las acciones de suspensión y el cambio de contraseña, sin entrar servidor por servidor a WHM.

## El problema que resuelve

Al administrar hosting, las tareas más repetidas son siempre las mismas: buscar en qué servidor vive un dominio, suspender o reactivar una cuenta, cambiar la clave de cPanel de un cliente y revisar si el dominio está apuntando a los DNS correctos. Con varios servidores WHM eso significa abrir varias sesiones y repetir búsquedas.

`mod-whm` junta todo en una sola pantalla, como un proyecto independiente con su propio login.

## Qué necesitas

- PHP 7.4 o superior con las extensiones `curl` e `intl` (para `idn_to_ascii`/`idn_to_utf8`).
- Apache con `mod_authz_core` o compatible con `.htaccess` (para bloquear carpetas sensibles).
- Uno o más servidores WHM y un API Token en cada uno (WHM » Development » Manage API Tokens).
- Salida hacia el puerto 2087 de cada servidor WHM y hacia `whois.nic.cl:43` si usarás la validación de DNS.

## Instalación paso a paso

### 1. Clona el repositorio

```bash
git clone https://github.com/MarioRiffoCortes/mod-whm.git
cd mod-whm
```

### 2. Configura los servidores WHM

Copia la plantilla y completa los datos de cada servidor. La clave del arreglo (`SERVIDOR1`) es el nombre que verás en el panel.

```bash
cp ADM/config.example.php ADM/config.php
```

```php
$WH_SERVERS = [
    'SERVIDOR1' => [
        'host'  => 'whm.ejemplo.com',
        'user'  => 'root',
        'token' => 'TU_API_TOKEN_AQUI',
    ],
];

// Opcional: solo si usarás el botón "Validar DNS"
$OFFICIAL_DNS = [
    'SERVIDOR1' => ['ns' => ['ns1.ejemplo.com', 'ns2.ejemplo.com'], 'estado' => 'vigente'],
];
```

### 3. Define los usuarios del panel

```bash
cp ADM/config-ad.example.php ADM/config-ad.php
```

Ahí defines `$USERS` como `usuario => contraseña`. Más abajo explico por qué conviene cambiar esto a hashes antes de usarlo en producción.

### 4. Verifica que las carpetas sensibles estén bloqueadas

`ADM/` y `scripts/cache/` traen un `.htaccess` que niega todo acceso web. Compruébalo desde fuera:

```bash
curl -I https://tu-dominio.com/mod-whm/ADM/config.php
## Debe responder 403 Forbidden
```

Si responde 200, tu servidor no está leyendo los `.htaccess` (revisa `AllowOverride`) y estarías exponiendo los tokens.

## Cómo funciona por dentro

### Autenticación contra WHM

Todas las llamadas usan el mismo encabezado con el token del servidor:

```php
$ch = curl_init("https://$host:2087/json-api/listaccts?api.version=1");
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER     => ["Authorization: whm $user:$token"],
    CURLOPT_TIMEOUT        => 20,
]);
$data = json_decode(curl_exec($ch), true);
$cuentas = $data['data']['acct'] ?? [];
```

El resultado de cada operación se revisa en `metadata.result` (1 = éxito) y el mensaje de WHM viene en `metadata.reason`.

### Endpoints del panel

| Endpoint | Llamada WHM | Qué hace |
|---|---|---|
| `scripts/listAccounts.php` | `listaccts` | Lista cuentas por servidor, activas primero, con caché diaria |
| `scripts/accountAction.php` | `suspendacct`, `unsuspendacct`, `removeacct` | Suspende (con motivo), reactiva o elimina una cuenta |
| `scripts/changePassword.php` | `listaccts` + `passwd` | Busca el usuario cPanel por dominio y cambia su contraseña |
| `scripts/checkDns.php` | — (WHOIS / DNS) | Compara los NS del dominio con los oficiales del servidor |
| `scripts/getServers.php` | — | Devuelve los servidores configurados |

Todos incluyen `ADM/auth.php` al inicio: sin sesión responden **401 en JSON** a las llamadas AJAX y redirigen al login en una navegación directa. En el navegador, `js/session-guard.js` detecta ese 401 y devuelve al usuario al login en vez de dejarlo con una pantalla rota.

### Caché diaria por servidor

`listaccts` en un servidor con cientos de cuentas tarda varios segundos. Por eso el resultado se guarda en `scripts/cache/data_<SERVIDOR>.json` y se reutiliza mientras la fecha del archivo sea la de hoy. Cuando suspendes o reactivas una cuenta, `accountAction.php` actualiza solo esa ficha dentro del JSON, así el cambio se ve al instante sin volver a consultar WHM.

### Validación de DNS contra Nic.cl

Este fue el detalle que más tiempo me costó. Al principio usaba `dns_get_record($dominio, DNS_NS)`, pero daba resultados distintos a los que el cliente veía en Nic.cl: si el cliente cambiaba los NS en Nic.cl y el servidor DNS antiguo seguía vivo, ese servidor respondía con su propia zona desactualizada.

La solución fue consultar directamente el WHOIS de NIC Chile para dominios `.cl`, y dejar `dns_get_record()` solo como respaldo:

```php
$fp = @fsockopen('whois.nic.cl', 43, $errno, $errstr, 5);
fwrite($fp, $dominio . "\r\n");
// Se leen las líneas "Name Server: nsX.dominio.cl"
```

Además, el script hace una búsqueda inversa: si los NS encontrados pertenecen a **otro** servidor conocido (por ejemplo uno marcado como `obsoleto`), el mensaje lo dice explícitamente: "está apuntando a X, cámbialo a los DNS de Y".

### Dominios con tildes y ñ

WHM entrega los dominios IDN en Punycode (`xn--...`). El panel los convierte a UTF-8 con `idn_to_utf8()` para mostrarlos y a ASCII con `idn_to_ascii()` antes de consultar la API o el WHOIS. Si el servidor no tiene la extensión `intl`, `listAccounts.php` incluye un decodificador Punycode manual como respaldo.

## Lo que probé y los resultados

- Lo uso a diario con varios servidores WHM, incluido uno ya dado de baja que se mantiene en modo solo lectura (se muestra su último JSON en caché sin consultar WHM).
- Con caché, cargar todas las cuentas pasa de varios segundos por servidor a una respuesta prácticamente inmediata.
- La validación por WHOIS de Nic.cl eliminó los falsos "DNS correctos" que obtenía con `dns_get_record()` en dominios recién migrados.

## Limitaciones conocidas antes de usarlo en producción

Si lo vas a usar en producción, ten en cuenta:

- **Verificación SSL desactivada.** Las llamadas cURL usan `CURLOPT_SSL_VERIFYPEER => 0`. Si tus servidores WHM tienen certificado válido, actívala (`true`) para evitar ataques de intermediario.
- **Contraseñas del panel en texto plano.** `config-ad.php` compara la clave tal cual. Lo recomendable es guardar hashes con `password_hash()` y validar con `password_verify()`.
- **La nueva contraseña de cPanel viaja en la URL** de la llamada `passwd`. Va cifrada por HTTPS, pero puede quedar en registros del servidor; conviene enviarla por POST.
- **Sin protección CSRF** en las acciones de suspender y eliminar, y sin `session_regenerate_id()` al iniciar sesión.

## Conclusión

Con un API Token por servidor y cuatro llamadas a la API de WHM puedes tener un panel que reemplaza la mayoría de las visitas manuales a WHM. La pieza que más valor aporta en Chile es la validación de DNS contra el WHOIS de Nic.cl, porque coincide con lo que el cliente ve y evita diagnósticos equivocados.
