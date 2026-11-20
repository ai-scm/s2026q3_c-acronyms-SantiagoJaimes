# s2026q3_c-acronyms-SantiagoJaimes

Micro-assignment de siglas del semillero, bloque (5): **MLOps, CI/CD y CORS**.
Autor: Santiago Jaimes

---

## 1. MLOps - Machine Learning Operationsgit

**MLOps** es el conjunto de prácticas, procesos y herramientas que une Machine Learning, DevOps e ingeniería de datos para llevar modelos de ML a producción y mantenerlos ahí de forma confiable, reproducible y automatizada durante todo su ciclo de vida: datos, entrenamiento, validación, despliegue, monitoreo y reentrenamiento.

En una frase: es aplicar a los modelos de ML la disciplina que CI/CD le dio al software, con la diferencia de que en ML no solo cambia el código, también cambian los datos y el modelo.

---

## 2. CI/CD - Continuous Integration / Continuous Delivery o Deployment

**CI/CD** es automatizar el camino que recorre un cambio de código desde el `git push` hasta producción, para que llegue rápido, seguido y con bajo riesgo.

| Sigla | Significa | Qué garantiza | Aprobación humana antes de producción |
|---|---|---|---|
| CI | Continuous Integration | Cada cambio se integra a la rama principal con frecuencia y dispara build y tests automáticos. Si algo se rompe, se sabe en minutos. | No aplica |
| CD | Continuous Delivery | Todo lo que pasa el pipeline queda siempre listo para desplegar; salir a producción es un clic. | Sí |
| CD | Continuous Deployment | Todo lo que pasa el pipeline se despliega solo a producción. | No |

La única diferencia entre *Delivery* y *Deployment* es si una persona aprueba el paso a producción.

```
   git push / pull request
             |
             v
   +----------------------- CI ------------------------+
   |   build  --->  lint  --->  tests                  |
   +---------------------------------------------------+
             |                              |
           pasa                           falla
             |                              |
             |                              v
             |                     el pipeline se detiene
             |                     y avisa al desarrollador
             v
   +----------------------- CD ------------------------+
   |   artefacto versionado (imagen Docker :sha)       |
   |     ---> deploy a staging ---> pruebas de humo    |
   |     ---> ¿aprobación humana?                      |
   +---------------------------------------------------+
             |                              |
            si                              no
   (Continuous Delivery)         (Continuous Deployment)
             |                              |
             +-------> PRODUCCION <---------+
                           |
                           v
                 monitoreo + rollback
```

Herramientas: GitHub Actions, GitLab CI, Jenkins. En AWS: CodePipeline (orquesta), CodeBuild (build y tests), CodeDeploy (despliegue) y ECR (registro de imágenes).

---

## 3. CORS - Cross-Origin Resource Sharing

**CORS** es un mecanismo del **navegador**, basado en headers HTTP, con el que un **servidor declara qué otros orígenes** pueden leer sus respuestas desde JavaScript. Sirve para relajar de forma controlada la *Same-Origin Policy*.

**Same-Origin Policy:** por seguridad, el navegador no deja que el JavaScript de una página lea respuestas de otro origen.
**Origen = esquema + host + puerto.** Si cambia cualquiera de los tres, es otro origen:

| Página en `https://app.com` pide a | ¿Mismo origen? | Por qué |
|---|---|---|
| `https://app.com/api/users` | Sí | solo cambia la ruta |
| `http://app.com/api` | No | cambia el esquema |
| `https://api.app.com` | No | cambia el host |
| `https://app.com:8443` | No | cambia el puerto |

**Dos tipos de petición:**

- **Simple** (`GET`, `HEAD` o `POST` de formulario): el navegador la envía directamente y luego revisa si la respuesta trae `Access-Control-Allow-Origin` con su origen.
- **Con preflight** (`PUT`, `DELETE`, JSON, header `Authorization`, etc.): el navegador pregunta primero con un `OPTIONS`.

```
   Navegador (origen A)                            API (origen B)
          |                                               |
          |---- OPTIONS /recurso ------------------------>|   preflight
          |     Origin: A                                 |
          |     Access-Control-Request-Method: POST       |
          |                                               |
          |<--- 204 --------------------------------------|
          |     Access-Control-Allow-Origin: A            |
          |     Access-Control-Allow-Methods: GET, POST   |
          |                                               |
          |---- POST /recurso --------------------------->|   petición real
          |<--- 200 + Access-Control-Allow-Origin: A -----|
          |                                               |

   Si el 204 no trae el Allow-Origin correcto, el navegador
   bloquea y el POST nunca se envía.
```

| Header | Para qué |
|---|---|
| `Origin` | Lo pone el navegador; dice desde dónde viene la petición |
| `Access-Control-Allow-Origin` | El clave: origen permitido (exacto) o `*` |
| `Access-Control-Allow-Methods` / `-Headers` | Métodos y headers permitidos (en el preflight) |
| `Access-Control-Allow-Credentials` | Permite cookies o auth; no funciona junto con `*` |
| `Access-Control-Max-Age` | Cuánto tiempo se cachea el preflight |


---

## 4. Cómo se conectan

```
   +------------------------------ MLOps ------------------------------+
   |  misma idea de CI/CD, pero además versiona y automatiza           |
   |  los datos, el entrenamiento y los modelos                        |
   |                                                                   |
   |   +-------------+        +------------------------------+         |
   |   |   Codigo    |------->|            CI/CD             |         |
   |   +-------------+        |   integrar -> probar ->      |         |
   |   +-------------+        |   desplegar, en automatico   |         |
   |   |  Datos y    |------->|                              |         |
   |   |  modelo     |        +------------------------------+         |
   |   +-------------+                       |                         |
   +-----------------------------------------|-------------------------+
                                             |  despliega
                                             v
                              +------------------------------+
                              |     API en produccion        |
                              |     (por ejemplo en AWS)     |
                              +------------------------------+
                                   ^                   |
                 peticion fetch()  |                   |  respuesta con headers CORS
                 desde otro origen |                   |  (Access-Control-Allow-Origin)
                                   |                   v
                              +------------------------------+
                              |   Frontend en el navegador   |
                              |   (otro dominio o puerto)    |
                              +------------------------------+
```

- **CI/CD** es *cómo* llega el software a producción: cada cambio se integra, se prueba y se despliega de forma automática.
- **MLOps** toma esa misma idea y la extiende a sistemas de ML, donde además del código cambian los datos y el modelo (se habla de entrenamiento continuo además de CI y CD).
- **CORS** aparece en cuanto ese software desplegado, normalmente una API, es consumido por un frontend que vive en otro dominio o puerto. Es el navegador quien decide si deja leer la respuesta, según los headers que envíe la API.
- **El punto de unión práctico:** la configuración CORS es parte del código de la API, así que el pipeline de CI/CD puede probarla. Si alguien rompe los headers, los tests fallan y el error no llega a producción. Eso es lo que hace la demo de este repo.
