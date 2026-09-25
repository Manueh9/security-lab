# 02 · Authentication

- **Plataforma:** [PortSwigger Web Security Academy — Authentication](https://portswigger.net/web-security/authentication)
- **Nivel de esta sesión:** Apprentice. El tema tiene **3 labs Apprentice**; en esta sesión resolví el de **2FA simple bypass**. Enlaces abajo.

---

## 1. La idea en una frase

Un **fallo de autenticación** aquí no va de meter caracteres raros como en SQLi: va de que
**la lógica del login está mal razonada**. La aplicación comprueba las cosas en mal orden, o
da por bueno un paso que el atacante puede saltarse.

En una frase: **cada puerta valida bien por separado, pero el sistema entero deja un hueco
porque una comprobación falta donde tenía que estar.**

---

## 2. El concepto, desde el lado del desarrollador

Un login con doble factor (2FA) tiene dos fases: primero la contraseña, luego un código de un
solo uso. La intención es que aunque te roben la contraseña, sin el código no entren.

El problema aparece cuando el segundo factor está **en la interfaz pero no en el servidor**.
La aplicación te *enseña* la pantalla del código, pero la página de la cuenta no comprueba, en
cada petición, que hayas superado de verdad ese paso.

Visto en pseudocódigo de un controlador, el flujo vulnerable es este:

```csharp
// Fase 1: la contraseña es correcta -> marco la sesión como "logueada"
sesion.Usuario = usuario;          // ❌ pero esto ya da acceso...

// Fase 2: le muestro la pantalla del código
return View("IntroduceCodigo2FA");
```

```csharp
// La página de la cuenta:
public IActionResult MyAccount()
{
    if (sesion.Usuario == null) return Redirect("/login");  // ❌ solo mira si hay sesión
    return View(datosDe(sesion.Usuario));                   //    NO mira si pasó el 2FA
}
```

La sesión queda **a medio autenticar** —contraseña sí, segundo factor no— y `MyAccount()`
solo pregunta *"¿hay sesión?"*, no *"¿completó los dos pasos?"*. Si pides directamente la
página de la cuenta sin pasar por la pantalla del código, te la sirve igual. El código 2FA se
vuelve decorativo.

> **La causa raíz:** el estado de autenticación lo está decidiendo **la pantalla que le toca
> ver al usuario**, no el servidor en cada petición. El atacante no tiene que romper el
> código: solo tiene que no pasar por la pantalla que se lo pide.

---

## 3. La analogía con .NET

En una aplicación .NET bien hecha, una sesión a medio autenticar **no es una sesión válida**
hasta que supera el segundo factor. Eso se modela con un estado explícito, y el recurso
protegido lo exige:

```csharp
// La sesión guarda EN QUÉ FASE está, no solo "quién es"
sesion.Estado = EstadoAuth.PendienteDe2FA;   // tras validar la contraseña

// Solo al validar el código correcto:
sesion.Estado = EstadoAuth.Completo;

// Y el recurso protegido comprueba la fase, no solo la identidad:
public IActionResult MyAccount()
{
    if (sesion.Estado != EstadoAuth.Completo)   // ✅ exige los DOS pasos
        return Redirect("/login");
    return View(datosDe(sesion.Usuario));
}
```

Es la misma idea que `[Authorize]` frente a `[Authorize(Roles="Admin")]`: no basta con estar
identificado, hay que cumplir **la condición completa** que protege ese recurso. Aquí la
condición es "2FA superado", y tiene que vivir en el servidor, en cada petición, no en el
front.

> Regla mental que deja este tema: **el estado de autenticación lo decide el servidor en cada
> petición, no la pantalla que le toca ver al usuario.**

---

## 4. Los ataques del nivel Apprentice

El tema **Authentication** tiene **3 labs Apprentice**. En esta sesión hice el primero; anoto
los tres para saber qué queda.

### 4.1 · 2FA simple bypass  ✅ (el de esta sesión)

La aplicación te lleva a la pantalla del código tras la contraseña, pero la página de la
cuenta **no verifica** que el segundo factor se haya completado. Se accede a la cuenta de la
víctima pidiendo directamente su página de cuenta, sin pasar el código.

**Qué demuestra:** que la comprobación del 2FA faltaba en el servidor, en el recurso
protegido. La pantalla del código era un cartel, no una puerta cerrada.

### 4.2 · Password reset broken logic  🔜 (solo navegador)

Un restablecimiento de contraseña mal razonado que permite cambiar la contraseña de otro
usuario. Pendiente; no necesita herramienta nueva.

### 4.3 · Username enumeration via different responses  🔜 (necesita Burp Intruder)

Distinguir usuarios válidos de inválidos porque el servidor responde de forma sutilmente
distinta. Requiere **Burp Intruder** (fuerza bruta controlada), así que queda para cuando
instale Burp Suite Community.

> Con eso, el tema Authentication no se cierra del todo hasta instalar Burp. Los dos primeros
> son solo navegador; el tercero, no.

---

## 5. Cómo se defiende

En orden de importancia:

1. **Verificar el segundo factor en el servidor, en cada petición a un recurso protegido.**
   No basta con mostrar la pantalla del código: hay que impedir que una sesión a medio
   autenticar llegue a ningún sitio que no sea esa pantalla.

2. **Modelar el estado de la sesión de forma explícita** (`PendienteDe2FA` frente a
   `Completo`), en vez de tratar "tiene sesión" como sinónimo de "está autenticado". Los dos
   pasos son parte de *una* autenticación, no dos autenticaciones sueltas.

3. **Que el recurso protegido exija la condición completa, no solo la identidad.** El
   equivalente a no confundir `[Authorize]` con la comprobación real que ese recurso necesita.

4. **No fiarse del flujo del cliente.** Que la interfaz lleve al usuario por el orden correcto
   no garantiza nada: el atacante no usa tu interfaz, hace peticiones directas.

> Lo que **no** basta: poner la pantalla del código y confiar en que el usuario pase por ella.
> El control de que se ha pasado tiene que estar en el backend, no en la navegación.

---

## 6. Ejemplo resuelto

Resolví el lab **2FA simple bypass**. Lo cuento con mis palabras, y con honestidad sobre cómo
salió, porque salió de forma reveladora.

### Lab — 2FA simple bypass

<details>

<summary>Solución</summary>

**Lo que hice (y el detalle importante: lo resolví casi sin darme cuenta).**

1. Entré en el login con las credenciales de la víctima, `carlos:montoya` —en este lab las
   credenciales de Carlos las dan; lo que no tienes es acceso a su email para el código.
2. La aplicación validó la contraseña y me llevó a la **pantalla del código de segundo
   factor**. Mi sesión estaba ya a medio autenticar.
3. **No introduje ningún código** —no tengo el email de Carlos, así que no podía—. En vez de
   eso acabé en la página de la cuenta, cuya URL sigue el patrón:

   ```
   /my-account?id=carlos
   ```

   (el host es el subdominio efímero del lab, del tipo `https://LAB-ID.web-security-academy.net`).

4. El servidor **me sirvió la cuenta de Carlos**, sin haber pasado el segundo factor.

**Por qué funciona:** la página `/my-account` solo comprobaba que hubiera una sesión iniciada,
no que esa sesión hubiera **completado** el 2FA. Al pedir directamente el recurso protegido,
me lo dio, porque la verificación del segundo factor no existía en el servidor para esa página
—solo estaba la pantalla que *pedía* el código.

**El detalle que más me enseñó:** lo hice sin sentir que estuviera "atacando" nada. No hizo
falta ninguna herramienta ni ningún truco; bastó con llegar a una URL sin haber pasado por la
pantalla que se suponía que me frenaba. Esa es justo la sensación del bug del lado del
defensor: *"pero si yo puse la pantalla del código…"*. La pusiste en la interfaz; faltaba en
el servidor.

</details>

### Conclusión

El fallo es de **lógica**, no de entrada malformada: cada pieza —contraseña, pantalla del
código, página de cuenta— funcionaba, pero faltaba una comprobación en el sitio que contaba.
La defensa es que el servidor decida el estado de autenticación en cada petición y no deje que
una sesión a medio autenticar alcance un recurso protegido. En .NET: un estado de sesión
explícito (`PendienteDe2FA` / `Completo`) que el recurso exige, no un simple "¿hay usuario en
sesión?".

---

## 7. Recursos

- [PortSwigger — Authentication vulnerabilities](https://portswigger.net/web-security/authentication) (teoría oficial, base de esta sesión)
- [PortSwigger — Multi-factor authentication](https://portswigger.net/web-security/authentication/multi-factor) (la parte de 2FA)
- [OWASP — Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) (el lado defensivo)

- [Lab — 2FA simple bypass](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass)

---

⬅️ [Volver al índice del repo](../README.md) ·
⬅️ Apunte anterior: [01 · SQL Injection](01-sql-injection.md) ·
➡️ Siguiente apunte: *(pendiente — Access control)*