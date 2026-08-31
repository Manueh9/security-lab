# 🛡️ security-lab — Aprendiendo seguridad web desde cero

Apuntes y prácticas de mi aprendizaje de **seguridad web ofensiva**, escritos mientras lo
aprendo. Vengo de desarrollo C#/.NET, así que enfoco cada vulnerabilidad desde el lado que
ya conozco: **por qué el código que la sufre está escrito así, y cómo se escribe para que
no pase**.

> Si estás empezando en seguridad web, este repo te sirve. No hay nada de "hacking
> mágico": hay vulnerabilidades explicadas de forma que un desarrollador entienda la causa
> y la defensa.

---

## ⚠️ Ética y alcance (léelo)

Todo lo que hay aquí se practica **solo** en entornos hechos para ello:

- **[PortSwigger Web Security Academy](https://portswigger.net/web-security)** — laboratorios oficiales, legales y gratuitos.
- **[OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)** — aplicación deliberadamente vulnerable, que levanto en mi propia máquina.

Nunca contra sistemas de terceros. Atacar un sistema sin permiso explícito es un delito,
y además no es de lo que va esto: esto va de aprender a **construir software más seguro**.

En este repo **no se copian flags ni las soluciones oficiales literales** de los labs. Cada
apunte explica el concepto y mi razonamiento **con mis palabras**.

---

## Cómo usar este repo

- **`apuntes/`** → la teoría, en orden. Cada vulnerabilidad: qué es, por qué ocurre a
  nivel de código, cómo se explota (a alto nivel) y — lo más importante para mí — cómo se
  defiende en .NET.
- **`labs/`** → notas de laboratorios concretos cuando aportan algo más que la teoría.

---

## Índice

### Apuntes

| # | Apunte | De qué va |
|---|---|---|
| 01 | [SQL Injection](apuntes/01-sql-injection.md) | Meter SQL por un input; por qué pasa y cómo lo evita un `SqlParameter` |

*(Se irá ampliando: autenticación, XSS, control de acceso, CSRF…)*

### Labs

Aún sin entradas propias. Los primeros labs (SQL injection Apprentice de PortSwigger) se
resumen dentro del apunte 01.

---

## Roadmap (orden previsto)

Sigo el temario de PortSwigger, de menos a más. Idea de recorrido:

1. **SQL injection** ← estoy aquí
2. Authentication (fallos de login, fuerza bruta, lógica)
3. Access control (IDOR, escalada de privilegios)
4. Cross-site scripting (XSS)
5. CSRF
6. Y práctica abierta sobre OWASP Juice Shop aplicando lo anterior

Un tema por sesión, sin prisa. Cuando tenga vocabulario suficiente, las sesiones pasan a
ser "rompo tal cosa en Juice Shop" en vez de labs guiados.

---

## Por qué existe este repo

Estudio seguridad como uno de tres tracks en rotación (junto a Cloud/DevOps e IA/ML). El
objetivo no es ser pentester: es que, como desarrollador, entienda las vulnerabilidades lo
bastante bien como para no escribirlas. Documentarlo para otros es parte del método: si no
sé explicarlo, es que no lo he entendido.

Si ves un error, abre un issue. Se agradece.

---

## Licencia

[MIT](LICENSE)
