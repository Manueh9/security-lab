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
| 02 | [Authentication](apuntes/02-authentication.md) | Autenticación ≠ autorización; por qué una pantalla de 2FA no protege por sí sola |
| 03 | [Access control](apuntes/03-access-control.md) | El permiso se comprueba en cada acción; función de admin sin proteger y con URL impredecible |
| 05 | [Rol decidido por el cliente](apuntes/05-user-role-request-parameter.md) | El control existe pero se fía de una cookie, que el cliente cambia |


### Labs

| # | Lab | Plataforma / nivel | Estado |
|---|---|---|---|
| 02 | [2FA simple bypass](labs/02-2fa-simple-bypass/) | PortSwigger · Apprentice | ✅ |
| 03 | [Unprotected admin functionality](labs/03-unprotected-admin/) | PortSwigger · Apprentice | ✅ |
| 04 | [Admin con URL impredecible](labs/04-admin-url-impredecible/) | PortSwigger · Apprentice | ✅ |
| 05 | [User role controlled by request parameter](labs/05-user-role-request-parameter/) | PortSwigger · Apprentice | ✅ |

Los labs de SQL injection (Apprentice) se resumen dentro del apunte 01.

---

## Roadmap (orden previsto)

Sigo el temario de PortSwigger, de menos a más. Idea de recorrido:

1. **SQL injection** ✅
2. **Authentication** ✅ — falta el lab de enumeración de usuarios, que necesita Burp Suite
3. **Access control** ← estoy aquí: 3 de los 5 labs Apprentice del itinerario oficial. Siguen los de IDOR
4. Cross-site scripting (XSS)
5. CSRF
6. Y práctica abierta sobre OWASP Juice Shop aplicando lo anterior

---

## Otros repos de esta serie

Este repo es uno de tres, montados con la misma estructura de "asignatura":

| Repo | De qué va |
|---|---|
| [`cloud-devops-lab`](https://github.com/Manueh9/cloud-devops-lab) | Docker, Kubernetes, Terraform, CI/CD, Azure, observabilidad |
| [`security-lab`](https://github.com/Manueh9/security-lab) | Este: Seguridad web: PortSwigger Web Security Academy y OWASP Juice Shop |
| [`ml-lab`](https://github.com/Manueh9/ml-lab) | IA/ML con Python: datos, modelos y llevarlos a producción |

---

## Licencia

[MIT](LICENSE)