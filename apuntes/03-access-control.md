# 03 · Access control: el permiso se comprueba, no se presupone

> Anterior: [02 · Authentication](02-authentication.md) · Siguiente: *(pendiente)*
> Lab que lo acompaña: [labs/03-unprotected-admin](../labs/03-unprotected-admin/)

Access control es la pregunta que viene después del login: siendo ya un usuario reconocido,
**¿puedes hacer esta acción concreta?** Cuando se hace mal, es *broken access control*, el nº 1 del
OWASP Top 10 (A01:2021).

## 1. Autenticación vs autorización

El login comprueba **quién eres, una vez**. El control de acceso comprueba **qué puedes hacer, en
cada acción**. Superar el login no autoriza nada por sí solo.

## 2. Funcionalidad sin proteger

Una acción potente (típicamente de administración) que se protege **solo** no enlazándola desde la
interfaz, en vez de comprobar el permiso de quien la pide. Ocultar no es controlar: si la ruta se
puede encontrar o adivinar, no había ningún control.

Un detalle del caso: `robots.txt` existe para pedir a los buscadores que **no indexen** ciertas
rutas. No es un control de acceso — es, de hecho, un **mapa** de lo que el sitio preferiría tener
escondido. Confiar en él para "ocultar" el panel de admin es seguridad por ocultación: ninguna.

> Analogía (.NET): un endpoint sin `[Authorize]` en el que se confía porque no sale en el menú.
> La ausencia de un enlace no es un control de acceso.

## 3. La defensa

- **Denegar por defecto**: cada endpoint sensible exige un permiso explícito; lo que no se concede,
  se rechaza.
- El control se aplica **en el servidor**, en la propia acción, no en que la interfaz oculte el botón.
- En ASP.NET Core: `[Authorize(Roles = "Admin")]` o una política de autorización; nunca fiarlo a que
  la ruta sea poco adivinable.

## 4. Cómo lo evitaría en .NET

- Proteger el endpoint con `[Authorize(Roles = "Admin")]` (o una policy) desde el primer commit,
  no como añadido posterior.
- No depender de que la vista oculte el enlace: el atributo va en el controller/action, que es
  donde se decide de verdad quién puede entrar.
- Verificarlo con un test de integración que llame a la ruta de admin sin el rol adecuado y
  compruebe que devuelve 401/403, no 200.

## 5. El lab

[labs/03-unprotected-admin](../labs/03-unprotected-admin/) — PortSwigger, Apprentice, navegador.