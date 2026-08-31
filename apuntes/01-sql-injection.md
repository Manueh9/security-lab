# 01 · SQL Injection

- **Plataforma:** [PortSwigger Web Security Academy — SQL injection](https://portswigger.net/web-security/sql-injection)
- **Nivel de esta sesión:** Apprentice (2 labs). Enlaces de los labs abajo

---

## 1. La idea en una frase

Una **inyección SQL (SQLi)** ocurre cuando una aplicación construye una consulta SQL
metiendo dentro, tal cual, texto que ha escrito el usuario. Si ese texto no se trata como
un simple **dato** sino que acaba formando parte de la **sentencia**, el usuario puede
reescribir lo que la base de datos ejecuta.

En una frase: **el atacante deja de rellenar el formulario y empieza a escribir la query.**

---

## 2. El concepto, desde el lado del desarrollador

Imagina un buscador de productos por categoría. En código vulnerable, la query se monta pegando strings:

```csharp
// ❌ VULNERABLE — no hagas esto nunca
string sql = "SELECT * FROM products WHERE category = '" + categoria + "' AND released = 1";
```

Si `categoria` vale `"Gifts"`, la query final es la esperada:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Pero el usuario controla `categoria`. ¿Y si en vez de `Gifts` escribe `Gifts'--`?

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

`--` es el inicio de un **comentario** en SQL: todo lo que va detrás se ignora. Acabas de
borrar el `AND released = 1`. La app quería enseñarte solo productos publicados; ahora te
enseña también los que no lo están.

Y se puede ir más lejos. Con `Gifts' OR 1=1--`:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--
```

`1=1` es siempre verdadero, así que el `WHERE` deja de filtrar: devuelve **toda** la tabla.

> **La causa raíz:** la base de datos no distingue "el dato que mandó el usuario" de "la
> orden que escribió el programador". Todo le llega como un único string de texto. La
> comilla `'` del atacante cierra la cadena antes de tiempo, y a partir de ahí lo que
> escribe se interpreta como SQL.

---

## 3. La analogía con EF Core

En .NET por ejemplo ya se escribe esto:

```csharp
// ✅ SEGURO — consulta parametrizada
var productos = db.Products
    .FromSqlInterpolated($"SELECT * FROM products WHERE category = {categoria}")
    .ToList();

// ✅ SEGURO — LINQ, ni siquiera tocas SQL a mano
var productos = db.Products.Where(p => p.Category == categoria).ToList();

// ✅ SEGURO — ADO.NET clásico con parámetro
cmd.CommandText = "SELECT * FROM products WHERE category = @cat";
cmd.Parameters.AddWithValue("@cat", categoria);
```

Con un **parámetro**, el valor viaja por un canal aparte de
la sentencia: la base de datos recibe la query con un hueco (`@cat`) y, por separado, el
valor que va en ese hueco. Nunca los concatena. Da igual que el usuario escriba
`' OR 1=1--`: eso se guarda como una categoría que literalmente se llama `' OR 1=1--` y
que, evidentemente, no existe. **El input jamás puede cambiar la estructura de la query.**

---

## 4. Los dos ataques del nivel Apprentice

PortSwigger tiene exactamente **dos labs Apprentice** de SQL injection. Estos son los
conceptos que entrenan.

### 4.1 · Recuperar datos ocultos (retrieval of hidden data)

La app filtra por una condición extra que a ti te oculta cosas (`AND released = 1`,
`AND public = 1`…). Comentando el resto de la query con `--`, o forzando la condición con
`OR 1=1`, se salta ese filtro:

```
category = Gifts'--
category = Gifts' OR 1=1--
```

**Qué demuestra:** que el filtro de seguridad estaba en el `WHERE`, y el `WHERE` lo
controla quien controla el input.

### 4.2 · Saltarse el login (login bypass)

El login vulnerable comprueba usuario y contraseña en la misma query:

```sql
SELECT * FROM users WHERE username = 'ENTRADA_USER' AND password = 'ENTRADA_PASS'
```

Si en el campo de usuario metes `administrator'--`, comentas la comprobación de la
contraseña entera:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

La base de datos busca al usuario `administrator`, ignora todo lo de la contraseña, lo
encuentra, y la app te da por autenticado. **Entras como admin sin saber su contraseña.**

> Ojo al detalle de diseño que esto revela: meter la validación de la contraseña dentro de
> la propia query SQL ya es mala idea de por sí. La comprobación de credenciales debería
> hacerse en la lógica de la aplicación, comparando un hash, no dependiendo de que una
> fila haga match.

---

## 5. Cómo se defiende

En orden de importancia:

1. **Consultas parametrizadas / prepared statements, siempre.** Es la defensa real y de
   fondo. En .NET: LINQ, `FromSqlInterpolated`, o `SqlParameter`. Nunca concatenar input
   dentro de la sentencia. Regla mental: *el input del usuario es un valor, jamás es
   código.*

2. **ORM bien usado.** EF Core parametriza por defecto. El riesgo aparece justo cuando te
   sales de él con SQL crudo mal construido (`FromSqlRaw` con concatenación, por ejemplo).

3. **Menor privilegio en la cuenta de BD.** El usuario con el que la app se conecta no
   debería poder borrar tablas ni leer el catálogo entero. Limita el daño si algo falla.

4. **Validación de entrada** (allowlists donde el conjunto de valores es conocido, p. ej.
   un `ORDER BY` que solo admite ciertos nombres de columna). Es defensa en profundidad,
   **no** sustituye a los parámetros.

> Lo que **no** basta: "escapar comillas a mano" o filtrar palabras como `OR`/`SELECT`.
> Es frágil y se salta de mil formas. La solución correcta es estructural: parámetros.

---

## 6. Ejemplo resuelto

Resolví los dos labs Apprentice de PortSwigger. Los explico con mis palabras.

### Lab 1 — Retrieval of hidden data

**Campo vulnerable:** el parámetro `category` de la URL (`/filter?category=...`),
que va directo al `WHERE` de la query.

**Payload:** `' OR 1=1--`  (en la URL, codificado: `%27+OR+1=1--`)

**Query resultante:**

```sql
SELECT * FROM products WHERE category = '' OR 1=1--' AND released = 1
```

**Por qué funciona:** `OR 1=1` es siempre cierto, así que el `WHERE` deja de filtrar
y devuelve todos los productos; el `--` comenta el `AND released = 1`, que era el
filtro que ocultaba los productos no publicados.

### Lab 2 — Login bypass

**Campo vulnerable:** el campo de usuario del formulario de login.

**Payload:** `administrator'--` (contraseña en blanco)

**Query resultante:**

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

**Por qué funciona:** la comilla cierra el nombre de usuario y `--` comenta la
comprobación de la contraseña entera. La BD encuentra al usuario `administrator` y
la app me autentica sin validar ninguna contraseña.

### Lo que me llevo

La causa de las dos es la misma: el input del usuario acaba formando parte de la
sentencia SQL en vez de tratarse como un dato. Con una consulta parametrizada
(`SqlParameter`, `FromSqlInterpolated`, LINQ) esto es imposible, porque el valor
nunca se re-interpreta como SQL.

> **Nota sobre `%27`:** en la URL escribí `%27+OR+1=1--`. Es el mismo payload
> `' OR 1=1--` pero URL-encoded: `%27` es la comilla `'` y `+` es un espacio.
> En una URL hay que codificar esos caracteres; en un campo de formulario (como
> el login del Lab 2) se escribe el carácter real y el navegador lo codifica solo.

---

## 7. Recursos
- [PortSwigger — SQL injection](https://portswigger.net/web-security/sql-injection) (teoría oficial, base de esta sesión)
- [PortSwigger — SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) (referencia de sintaxis por motor de BD)
- [OWASP — SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) (el lado defensivo, muy bueno)

- [Lab 1](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)
- [Lab 2](https://portswigger.net/web-security/sql-injection/lab-login-bypass)
---

⬅️ [Volver al índice del repo](../README.md) ·
➡️ Siguiente apunte: *(pendiente — Authentication)*
