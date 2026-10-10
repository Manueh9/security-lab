# Lab 04 · Funcionalidad de administración con URL impredecible

> Anterior: [lab 03 · Unprotected admin functionality](../03-unprotected-admin/)
> Teoría: [apuntes/03-access-control.md](../../apuntes/03-access-control.md)

**PortSwigger Web Security Academy · Access control · Apprentice · solo navegador.**
Enlace del tema: <https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url\>

## Alcance ético

Resuelto **solo** en el entorno de PortSwigger. Aquí no hay flag ni solución oficial copiada: está
mi razonamiento, con mis palabras.

## El escenario

Un panel de administración que **no comprueba quién lo usa**. A diferencia del
[lab 03](../03-unprotected-admin/), la ruta no es adivinable: lleva un identificador que nadie
escribiría por casualidad. La aplicación trata ese nombre impredecible como si fuera la protección.
Objetivo: eliminar al usuario `carlos`.

## Dos formas de buscar la puerta: una ruidosa que falla, una silenciosa que funciona

### Intento 1 — fuerza bruta de rutas (dirb). NO la encuentra.

```
$ dirb https://<instancia>.web-security-academy.net/ /opt/homebrew/share/dirb/wordlists/common.txt

GENERATED WORDS: 4612
+ .../analytics   (CODE:200|SIZE:0)
+ .../favicon.ico (CODE:200|SIZE:15406)
+ .../filter      (CODE:200|SIZE:11343)
+ .../login       (CODE:200|SIZE:3753)
+ .../logout      (CODE:302|SIZE:0)
+ .../my-account  (CODE:302|SIZE:0)
+ .../product     (CODE:400|SIZE:30)

DOWNLOADED: 4612 - FOUND: 8
```

4.612 palabras probadas y **la ruta de admin no aparece**. Ni siquiera sale `/admin`. Y esto no es
un fallo de la herramienta: es el lab funcionando. `dirb` solo encuentra lo que está en su lista de
palabras, y una ruta impredecible, por definición, no está en ninguna lista. La ofuscación **sí**
detuvo la fuerza bruta.

### Intento 2 — leer lo que el servidor ya me mandó (F12). La encuentra en segundos.

<<PEGA AQUÍ con tus palabras: dónde miraste con las DevTools y cómo llegaste al JavaScript.>>

El servidor envía al navegador el código que construye el enlace al panel, aunque luego decida no
pintarlo. La ruta estaba ahí, en lo que descarga cualquiera.

<details>
<summary>La ruta y el fragmento que la delataba (spoiler)</summary>

Ruta encontrada: `/admin-null`

El fragmento de JavaScript era este:

```javascript
var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-null');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
```

<<PEGA AQUÍ en una frase: cómo la construía y por qué a ti te salía el -null.>>

</details>

## Por qué salía `-null`

La ruta se construye concatenando una variable que, para un usuario que no es administrador, viene
**vacía**. En lugar de un identificador secreto, me llegó literalmente `admin-null`. El intento de
esconder la ruta no solo era inútil —estaba en el código que descargué— sino que además estaba mal
hecho: me entregó la plantilla de la dirección con el hueco a la vista.

## La lección: `dirb` falla y F12 no, y eso lo explica todo

```
   dirb / fuerza bruta          lee la LISTA de rutas conocidas
                                -> encuentra /login, /my-account, /filter...
                                -> NO encuentra /admin-null  (no está en la lista)

   leer lo que el navegador     lee lo que el SERVIDOR te mandó
   ya descargó (F12)            -> la ruta estaba en el JavaScript
```

Una URL impredecible derrota al método ruidoso y cae ante el de treinta segundos. **Esconder no
previene el descubrimiento: solo cambia por qué puerta entra el que busca.** Todo lo que el servidor
manda al cliente —HTML, JavaScript, comentarios— es público por definición.

## La lección defensiva (cómo se hace bien en una app real)

El fallo de fondo es el mismo que en el lab 03: **no hay comprobación de permiso en la acción**. Ni
la ruta rara ni el `if` del cliente son un control de acceso. En ASP.NET Core, tres capas:

**1. Denegar por defecto** — que olvidarse signifique "cerrado", no "abierto":

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

**2. El permiso, en la acción** — comprobado en el servidor, en cada petición:

```csharp
[Authorize(Roles = "Admin")]
[HttpPost("/admin-xyz/delete")]
public IActionResult DeleteUser(string username) { ... }
```

Da igual cómo se encuentre la ruta —por F12, por un log filtrado o probando a mano—: si no eres
admin, 403.

**3. La URL impredecible, como mucho, defensa en profundidad** — una capa extra sobre un control
real, nunca el control. Si al quitar el nombre raro la app queda indefensa, es que el nombre raro
*era* la defensa, y eso es el bug.

<<PEGA AQUÍ con tus palabras: cómo lo blindarías tú, y por qué un if en el cliente (JavaScript) no
sirve y uno en una vista Razor del servidor es mejor pero tampoco basta sin [Authorize].>>

## Pendiente a propósito

- **`User role controlled by request parameter`**: el siguiente del tema. Ahí el rol viaja en una
  cookie (`Admin=false`) y se puede cambiar desde las DevTools. Es el salto de "no se comprueba" a
  "se comprueba, pero se cree lo que dice el cliente".
- **Los labs de IDOR**: el mismo principio aplicado a datos de otro usuario en vez de a
  funcionalidad de administración.
- **Un escaneo de rutas más serio** (gobuster/ffuf con una wordlist mayor) tampoco habría
  encontrado `/admin-null`: no es cuestión de lista más grande, es que la ruta no es adivinable. La
  vía siempre era leer el cliente.
