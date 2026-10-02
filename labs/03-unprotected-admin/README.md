# Lab 03 · Unprotected admin functionality

> Teoría: [apuntes/03-access-control.md](../../apuntes/03-access-control.md)
> Anterior: [lab 02 · 2FA simple bypass](../02-2fa-simple-bypass/)

**PortSwigger · Access control · Apprentice · solo navegador.**
Enlace: <https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality>

## Alcance ético
Resuelto solo en el entorno de PortSwigger.

## El escenario
La tienda tiene un panel de administración sin enlace visible en la interfaz; el objetivo es
encontrarlo y usarlo para eliminar al usuario `carlos`.

## Mi razonamiento
Sin ningún enlace al panel en el menú ni en el HTML, el sitio sólo intentaba ocultarlo. Revisé
`/robots.txt`, que en PortSwigger suele listar justo las rutas que el sitio preferiría no indexar,
y ahí aparecía `/administrator-panel` en el `Disallow`. Entré directamente a esa ruta sin haber
iniciado sesión como admin y la app me dejó entrar igual: no había ninguna comprobación de rol en
el servidor, solo la ausencia de un enlace. Desde el panel pude borrar a `carlos` sin más.

## La lección defensiva
El servidor tenía que comprobar, en el propio endpoint del panel, que el usuario autenticado tiene
rol de administrador antes de devolver nada — y rechazar con 401/403 si no lo tiene. Ocultar la
ruta (no enlazarla, meterla en `robots.txt`) no sustituye esa comprobación; en cuanto alguien
adivina o descubre la URL, el "control" desaparece.