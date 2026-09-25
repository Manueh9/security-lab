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
| 02 | [Authentication](apuntes/02-authentication.md) | Fallos de lógica en el login; bypass de 2FA |

*(Se irá ampliando: control de acceso, XSS, CSRF…)*

### Labs

| # | Lab | Nivel | Estado |
|---|-----|-------|--------|
| 02 | [2FA simple bypass](labs/02-2fa-simple-bypass/) | Apprentice | ✅ Completo |

*(Los labs de SQLi del apunte 01 se quedan documentados dentro de ese apunte; el primer lab
con carpeta propia es este 02, por eso la numeración de `labs/` empieza ahí.)*

---

## Roadmap (orden previsto)

Sigo el temario de PortSwigger **por nivel Apprentice, tema a tema**: cierro el Apprentice de
cada tema y paso al siguiente, dejando los Practitioner para una segunda vuelta.

1. **SQL injection** — Apprentice ✅ (2 labs)
2. **Authentication** ← estoy aquí — Apprentice en curso (2FA bypass ✅; quedan *password reset broken logic* y *username enumeration*, este último con Burp)
3. Access control (IDOR, escalada de privilegios)
4. Cross-site scripting (XSS)
5. CSRF
6. Y práctica abierta sobre OWASP Juice Shop aplicando lo anterior

> Nota: el recuento de labs Apprentice de cada tema lo verifico el día que arranco ese tema,
> porque PortSwigger reorganiza su catálogo cada cierto tiempo.

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