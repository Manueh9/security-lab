# 05 · Access control: el rol decidido por un parámetro de la petición

> Anterior: [04 · Admin con URL impredecible](<<NOMBRE REAL DEL APUNTE 04>>)  ·  Siguiente: *(pendiente)*
> Lab que lo acompaña: [labs/05-user-role-request-parameter](../labs/05-user-role-request-parameter/)

Los dos casos anteriores fallaban porque **no había control**: la función de admin solo estaba
escondida. Este es más fino: el control **sí existe** —el servidor comprueba si eres admin— pero lo
decide a partir de un dato que viaja en la petición, y todo lo que viaja en la petición lo controla
quien la envía.

## El principio

Hay datos del **lado del servidor** (la sesión, la fila del usuario en la base de datos) y datos del
**lado del cliente** (cookies, campos ocultos, parámetros de URL o de cuerpo, cabeceras). Los
segundos pueden **entrar** en una petición, pero **nunca** pueden ser la base de una decisión de
permiso: el que está al otro lado los cambia a lo que quiera.

El fallo de hoy es confiar en un dato del cliente para decidir un rol.

## La lección defensiva

El rol se decide en el servidor, a partir de la identidad autenticada, mirando la fuente de verdad
del servidor —no un dato que venga en la petición. En .NET: `User` + la fila del usuario en la base
de datos, nunca un campo que mande el cliente.

`<<PEGA AQUÍ, si quieres, cómo lo cerrarías tú en un endpoint de .NET.>>`

## Pendiente a propósito

- Los dos labs Apprentice que siguen en el itinerario, la familia "User ID controlled by request
  parameter", que llevan a **IDOR** (referencias directas a objetos inseguras).
- Un reto de OWASP Juice Shop de control de acceso, para ver el mismo concepto en una aplicación
  entera y no en un lab de un solo fallo.