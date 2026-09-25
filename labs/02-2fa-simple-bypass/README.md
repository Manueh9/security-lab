# Lab 02 · 2FA simple bypass (PortSwigger, Apprentice)

**Primera entrada de la carpeta `labs/` de este repositorio.**

## Objetivo del lab

Acceder a la cuenta de Carlos (`carlos`) sin disponer de su código de segundo factor,
aprovechando que la aplicación no verifica en el servidor que el 2FA se haya completado.

## Credenciales del lab

- Mi cuenta: `wiener:peter`
- Cuenta objetivo: `carlos:montoya` (las credenciales se dan; lo que no tienes es su email
  para el código 2FA, y ese es el reto).

## Cómo lo resolví

<details>

<summary>Solución (mis pasos reales)</summary>

Lo resolví de forma más directa que el guion habitual —de hecho, casi sin darme cuenta de que
lo estaba resolviendo—, y lo dejo documentado tal como pasó porque es lo más honesto.

1. Entré en el login con `carlos:montoya`.
2. La aplicación validó la contraseña y me llevó a la **pantalla del código de segundo
   factor**. En ese punto mi sesión estaba a medio autenticar: contraseña sí, 2FA no.
3. **No introduje ningún código** (no tengo el email de Carlos). Acabé en la página de la
   cuenta, cuya URL sigue el patrón:

   ```
   /my-account?id=carlos
   ```

   El host es el subdominio efímero del lab (`https://LAB-ID.web-security-academy.net`); no lo
   pego entero porque cada instancia del lab es de un solo uso.
4. El servidor me sirvió la cuenta de Carlos, **sin haber pasado el segundo factor**. Lab
   resuelto.

</details>

## Por qué funciona

La página `/my-account` solo comprobaba que existiera una sesión iniciada, **no** que esa
sesión hubiera completado el 2FA. Al pedir directamente ese recurso, el servidor lo entregó,
porque la verificación del segundo factor no estaba donde tenía que estar —en el backend, en
cada petición al recurso protegido— sino solo en la pantalla que pedía el código.

Es un fallo de **lógica**: la contraseña, la pantalla del código y la página de la cuenta
funcionaban por separado, pero faltaba la comprobación que las ataba. La sesión a medio
autenticar nunca debió poder llegar a la cuenta.

## Lo que más me enseñó

No sentí que estuviera atacando nada: sin herramientas ni trucos, bastó con llegar a una URL
sin pasar por la pantalla que se suponía que me frenaba. Esa es exactamente la sensación del
bug desde el lado del que lo sufre: la protección estaba en la interfaz, no en el servidor.

## La defensa

- Verificar el segundo factor **en el servidor, en cada petición** a un recurso protegido.
- Modelar la sesión con un estado explícito (`PendienteDe2FA` / `Completo`) y no tratar "tiene
  sesión" como "está autenticado".
- Que el recurso protegido exija la condición completa, no solo la identidad.

## Qué queda pendiente a propósito

- Los otros dos labs Apprentice de Authentication: **"Password reset broken logic"** (solo
  navegador) y **"Username enumeration via different responses"** (necesita Burp Intruder).
- Instalar **Burp Suite Community**, que abre toda la familia de labs de fuerza bruta y
  enumeración —y hace falta para cerrar del todo este tema—.


> Teoría: [apuntes/02-authentication.md](../../apuntes/02-authentication.md)
>
> Lab oficial: https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass
