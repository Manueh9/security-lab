# Lab 05 · User role controlled by request parameter

> Anterior: [lab 04 · Admin con URL impredecible](../04-admin-url-impredecible/)
> Teoría: [apuntes/05-user-role-request-parameter.md](../../apuntes/05-user-role-request-parameter.md)
> Plataforma: PortSwigger Web Security Academy · Access control · Apprentice · solo navegador
> Enlace: <https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter>

## Alcance ético

Resuelto **solo** en el entorno de PortSwigger. Aquí no hay flag ni solución oficial copiada: está
mi razonamiento, con mis palabras.

## El escenario

Entrar al panel de administración con una cuenta de usuario normal (`wiener`) y usarlo para borrar
al usuario `carlos`.

## Qué lo hace distinto del 03 y del 04

En los anteriores no había control que saltarse: la función estaba escondida. Aquí **sí** hay
control —el servidor mira si eres admin— pero lo decide leyendo un dato que viaja en la petición.

## Mi procedimiento

<details>
<summary>Cómo lo resolví (spoiler)</summary>

1. Inicié sesión como `wiener` con las DevTools abiertas.
2. En las cookies del navegador vi que el servidor me había dejado una que indicaba si era
   administrador, con valor `false`.
3. Cambié ese valor a `true` desde las DevTools y recargué.
4. El panel de administración me dejó entrar y desde ahí borré a `carlos`.

</details>

## La lección defensiva

Una cookie la controla el cliente: el servidor no puede fiarse de lo que diga para decidir un
permiso. La cookie solo debería llevar un identificador de sesión, y el rol se tiene que buscar en
el servidor, en su base de datos. En ASP.NET Core, usar su autenticación con cookies (que las cifra
y firma) y `[Authorize(Roles = "Admin")]`, nunca leer un rol a mano de `Request.Cookies`.

## Qué queda pendiente a propósito

- La familia "User ID controlled by request parameter" → IDOR.
- OWASP Juice Shop, un reto de control de acceso, cuando toque la primera sesión de Juice Shop.