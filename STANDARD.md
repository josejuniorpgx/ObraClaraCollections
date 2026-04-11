# Bruno Collection Standard — ObraClara

Prompt de referencia para **crear** y **actualizar** colecciones Bruno alineadas a este proyecto.

> Cuando se pida crear o actualizar endpoints, seguir este documento al pie de la letra.
> No omitir secciones. No inventar convenciones nuevas.

---

## 0. Nomenclaturas

### Archivos de endpoint

| Elemento         | Convencion                        | Ejemplo                       |
|------------------|-----------------------------------|-------------------------------|
| Nombre de archivo| Titulo legible, espacios, `.yml`  | `Create.yml`, `List All.yml`, `Assign to Project.yml` |
| `info.name`      | Coincide con el archivo (sin .yml)| `Create`, `List All`, `Assign to Project` |

**NO** usar: snake_case (`projects_create.yml`), camelCase (`projectsCreate.yml`), ni prefijos redundantes (`projects_expenses_categories_partial_update.yml`).

### Carpetas de recurso

| Elemento         | Convencion                        | Ejemplo                       |
|------------------|-----------------------------------|-------------------------------|
| Carpeta raiz     | minusculas, sin espacios          | `projects/`, `expenses/`, `mediafiles/` |
| Sub-carpeta      | minusculas, guiones si es compuesto | `follow-ups/`, `participants/`, `categories/` |

### Archivos de configuracion

| Archivo              | Nombre fijo      | Ubicacion                |
|----------------------|------------------|--------------------------|
| Config de carpeta    | `folder.yml`     | Raiz de cada carpeta     |
| Config de coleccion  | `opencollection.yml` | Raiz de la coleccion |
| Config de workspace  | `workspace.yml`  | Raiz del proyecto        |
| Environments         | `<Nombre>.yml`   | `environments/` (PascalCase: `Local.yml`, `Staging.yml`) |

### Variables de environment

| Tipo              | Convencion                        | Ejemplo                       |
|-------------------|-----------------------------------|-------------------------------|
| URL base          | camelCase                         | `baseUrl`                     |
| Tokens            | snake_case, `secret: true`        | `token`, `refresh_token`      |
| IDs de recurso    | snake_case, sufijo `_id` o `_pk`  | `project_pk`, `stage_id`, `incident_id` |

> `project_pk` usa `_pk` porque asi lo espera la URL (`:project_pk`). Los demas usan `_id`.

### URLs de endpoint

| Elemento         | Convencion                        | Ejemplo                       |
|------------------|-----------------------------------|-------------------------------|
| Base             | `{{baseUrl}}/api/v1/`             | Siempre con variable          |
| Recurso          | plural, minusculas, guiones       | `projects/`, `media-files/`, `follow-ups/` |
| Path params      | snake_case con `:` prefijo        | `:project_pk`, `:id`, `:incident_pk`, `:token` |
| Trailing slash   | Siempre presente                  | `/api/v1/projects/` (NO `/api/v1/projects`) |

### Tags

| Elemento         | Convencion                        | Ejemplo                       |
|------------------|-----------------------------------|-------------------------------|
| Tag              | Nombre de la carpeta raiz del recurso | `projects`, `incidents`, `auth` |
| Sub-recursos     | Heredan el tag del padre          | `follow-ups/Create.yml` -> tag `incidents` |

### Nombres de endpoint por operacion

| Operacion                | Nombre del archivo  | Excepcion en auth              |
|--------------------------|--------------------|---------------------------------|
| Listar global            | `List All`         | —                               |
| Estadisticas global      | `Stats`            | —                               |
| Listar todos (filtrados) | `All`              | `expenses/categories/All.yml`   |
| Listar scoped            | `List`             | —                               |
| Listar filtro especial   | `List <Tipo>`      | `List Documents`                |
| Crear                    | `Create`           | `Login`, `Register`, etc.       |
| Agregar relacion         | `Add`              | `Add Participant` (en sub-carpeta: `Add`) |
| Subir archivo            | `Upload`           | —                               |
| Asignar relacion         | `Assign to <Cosa>` | `Assign to Project`             |
| Obtener detalle          | `Get`              | `Get Profile`, `Get User`, `Get by Token` |
| Actualizar completo      | `Update`           | `Update Profile`, `Update User` |
| Actualizar parcial       | `Patch`            | `Patch Profile`, `Patch User`   |
| Eliminar                 | `Delete`           | `Delete Account`                |
| Eliminar relacion        | `Remove`           | `Remove Participant`, `Remove Supplier` |
| Cancelar                 | `Cancel`           | —                               |
| Aceptar                  | `Accept`           | —                               |

> En sub-carpetas se usan nombres cortos (`Add`, `Remove`) porque la carpeta ya da contexto.
> En la carpeta raiz si el recurso no es el principal, se usa nombre completo (`Remove Supplier` en `projects/`).

---

### Documentacion (docs)

| Elemento         | Regla                                                        |
|------------------|--------------------------------------------------------------|
| `folder.yml`     | `docs` **obligatorio**. Describir recurso, permisos por rol, constraints. |
| Endpoint `.yml`  | `docs` **obligatorio**. Minimo una linea describiendo que hace el endpoint. |

Aunque la descripcion sea breve, **todo endpoint debe tener `docs`**. Nunca dejar un endpoint sin documentar.

Ejemplos validos:
```yaml
# Breve (minimo aceptable)
docs: List or assign suppliers to a project.

# Detallado
docs: |-
  Check the credentials and return the REST Token
  if the credentials are valid and authenticated.
  Calls Django Auth login method to register User ID
  in Django session framework

  Accept the following POST parameters: username, password
  Return the REST Framework Token Object's key.
```

---

## 1. Estructura de proyecto

```
workspace.yml
collections/
  <NombreAPI>/
    opencollection.yml          # Config de coleccion (auth, scripts, proxy)
    environments/
      Local.yml
      Staging.yml
    <recurso>/                  # Un folder por recurso principal
      folder.yml
      List All.yml              # Endpoints globales (sin project scope)
      Stats.yml
      List.yml                  # Endpoints scoped al proyecto
      Create.yml
      Get.yml
      Update.yml
      Patch.yml
      Delete.yml
      <sub-recurso>/            # Sub-carpeta para recursos anidados
        folder.yml
        List.yml
        Create.yml
        ...
```

### Sub-carpetas obligatorias

Cuando un recurso tiene sub-recursos anidados (ej: participants dentro de projects), se crea una sub-carpeta. Nunca mezclar endpoints de distintos recursos en el mismo nivel.

Ejemplos actuales:
- `projects/participants/` — participantes de un proyecto
- `incidents/follow-ups/` — seguimientos de un incidente
- `expenses/categories/` — categorias de gasto

---

## 2. Environments

Cada environment tiene las mismas variables. Los tokens van como `secret: true`.

```yaml
name: Local
variables:
  - name: baseUrl
    value: http://127.0.0.1:8000
  - secret: true
    name: token
  - secret: true
    name: refresh_token
  - name: project_pk
    value: ""
  - name: stage_id
    value: ""
  - name: participant_id
    value: ""
  - name: expense_id
    value: ""
  - name: incident_id
    value: ""
  - name: report_id
    value: ""
  - name: supplier_id
    value: ""
```

### Reglas

- `baseUrl` **sin** trailing slash (`http://host:8000`, NO `http://host:8000/`)
- `token` y `refresh_token` siempre `secret: true`
- Agregar una variable `<recurso>_id` por cada recurso que tenga Create
- Al agregar un nuevo recurso, agregar su variable de ID a **todos** los environments

---

## 3. opencollection.yml — Configuracion de coleccion

```yaml
opencollection: 1.0.0

info:
  name: ObraClaraAPI
config:
  proxy:
    inherit: true
    config:
      protocol: http
      hostname: ""
      port: ""
      auth:
        username: ""
        password: ""
      bypassProxy: ""

request:
  auth:
    type: bearer
    token: "{{token}}"
script:
  pre-request: |-
    const token = bru.getEnvVar("token");
    if (token) {
      try {
        const payload = JSON.parse(atob(token.split('.')[1]));
        const exp = payload.exp * 1000;
        const now = Date.now();
        const margin = 30 * 1000;
        if (exp - now < margin) {
          const refreshToken = bru.getEnvVar("refresh_token");
          if (refreshToken) {
            const baseUrl = bru.getEnvVar("baseUrl");
            const res = await fetch(baseUrl + "/api/v1/auth/token/refresh/", {
              method: "POST",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify({ refresh: refreshToken })
            });
            if (res.ok) {
              const data = await res.json();
              if (data.access) bru.setEnvVar("token", data.access);
              if (data.refresh) bru.setEnvVar("refresh_token", data.refresh);
            }
          }
        }
      } catch (e) {}
    }
bundled: false
extensions:
  bruno:
    ignore:
      - node_modules
      - .git
```

### Reglas

- Auth a nivel coleccion: `bearer` con `{{token}}`
- Pre-request script: auto-refresh JWT cuando faltan menos de 30s para expirar
- Los endpoints heredan esta auth por defecto

---

## 4. folder.yml — Configuracion de carpeta

```yaml
info:
  name: <nombre-del-recurso>
  type: folder
  seq: <numero>

request:
  auth: inherit

docs: |-
  Descripcion del recurso.
  Roles y permisos relevantes.
  Constraints importantes.
```

### Reglas

- `auth: inherit` siempre (hereda de la coleccion)
- `docs` obligatorio: describir que es el recurso, permisos por rol, constraints
- `seq` define el orden del folder en el sidebar (mantener orden logico)
- Sub-carpetas tambien llevan su `folder.yml` con docs

---

## 5. Nombres de endpoints

Nombres cortos y descriptivos. La carpeta da el contexto del recurso.

| Operacion             | Nombre           | Ejemplo en sidebar        |
|-----------------------|------------------|---------------------------|
| Listar (global)       | `List All`       | progress / List All       |
| Estadisticas          | `Stats`          | expenses / Stats          |
| Listar (por proyecto) | `List`           | stages / List             |
| Crear                 | `Create`         | reports / Create          |
| Obtener detalle       | `Get`            | suppliers / Get           |
| Actualizar completo   | `Update`         | stages / Update           |
| Actualizar parcial    | `Patch`          | projects / Patch          |
| Eliminar              | `Delete`         | incidents / Delete        |
| Acciones especiales   | Verbo descriptivo| `Accept`, `Cancel`, `Upload`, `Assign to Project` |

### Nombres de auth (excepcion)

Los endpoints de auth usan nombres descriptivos completos porque no siguen patron CRUD:
`Login`, `Register`, `Logout`, `Refresh Token`, `Verify Token`, `Change Password`, etc.

---

## 6. Secuencia (seq) — Orden CRUD

Dentro de cada carpeta, los endpoints se ordenan asi:

| seq | Tipo                    |
|-----|-------------------------|
| 1   | List All / All          |
| 2   | Stats                   |
| 3   | List                    |
| 4   | List Documents (u otro filtro) |
| 5   | Create / Add / Upload   |
| 6   | Get                     |
| 7   | Update                  |
| 8   | Patch                   |
| 9   | Delete / Remove / Cancel|
| 10  | Acciones especiales     |

---

## 7. Autenticacion por endpoint

| Tipo de endpoint            | Auth         |
|-----------------------------|--------------|
| Endpoint autenticado        | `auth: inherit` |
| Endpoint publico            | `auth: none`    |

### Endpoints publicos (auth: none)

Solo estos endpoints NO requieren autenticacion:
- Login, Register
- Reset Password, Confirm Reset
- Verify Email, Resend Verification
- Refresh Token, Verify Token
- Invitations: Get by Token

**Todos los demas** usan `auth: inherit`.

---

## 8. Estructura de un endpoint

### 8.1 GET List (paginado)

```yaml
info:
  name: List
  type: http
  seq: 3
  tags:
    - <nombre-del-recurso>

http:
  method: GET
  url: "{{baseUrl}}/api/v1/projects/:project_pk/<recurso>/"
  params:
    - name: project_pk
      value: ""
      type: path
    - name: ordering
      value: ""
      type: query
      description: "Que campo usar para ordenar los resultados."
      disabled: true
    - name: page
      value: ""
      type: query
      description: "Un numero de pagina dentro del conjunto de resultados paginado."
      disabled: true
    - name: page_size
      value: ""
      type: query
      description: "Numero de resultados a devolver por pagina."
      disabled: true
    - name: search
      value: ""
      type: query
      description: "Un termino de busqueda."
      disabled: true
  auth: inherit

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5

examples:
  - name: 200 Response
    request:
      url: "{{baseUrl}}/api/v1/projects/:project_pk/<recurso>/"
      method: GET
      params:
        - name: project_pk
          value: ""
          type: path
        - name: ordering
          value: ""
          type: query
          disabled: true
        - name: page
          value: ""
          type: query
          disabled: true
        - name: page_size
          value: ""
          type: query
          disabled: true
        - name: search
          value: ""
          type: query
          disabled: true
    response:
      status: 200
      statusText: OK
      headers:
        - name: Content-Type
          value: application/json
      body:
        type: json
        data: |-
          {
            "count": 0,
            "next": null,
            "previous": null,
            "results": []
          }

docs: Descripcion del endpoint.
script:
  tests: |-
    test("Status is 200", function() {
      expect(res.getStatus()).to.equal(200);
    });

    test("Response is JSON", function() {
      const contentType = res.getHeader("Content-Type");
      expect(contentType).to.contain("application/json");
    });
```

#### Reglas de query params

- **Todos** los List llevan como minimo: `ordering`, `page`, `page_size`, `search`
- Query params van con `disabled: true` (deshabilitados por defecto)
- Si el endpoint tiene filtros adicionales (ej: `status`, `priority`, `date_from`), se agregan despues de `search`
- Paginacion por defecto: `page_size=20`, `max_page_size=100`

### 8.2 GET Detail

```yaml
info:
  name: Get
  type: http
  seq: 6
  tags:
    - <nombre-del-recurso>

http:
  method: GET
  url: "{{baseUrl}}/api/v1/projects/:project_pk/<recurso>/:id/"
  params:
    - name: project_pk
      value: ""
      type: path
    - name: id
      value: ""
      type: path
      description: "UUID del recurso."
  auth: inherit

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5

examples:
  - name: 200 Response
    request:
      url: "{{baseUrl}}/api/v1/projects/:project_pk/<recurso>/:id/"
      method: GET
      params:
        - name: project_pk
          value: ""
          type: path
        - name: id
          value: ""
          type: path
    response:
      status: 200
      statusText: OK
      headers:
        - name: Content-Type
          value: application/json
      body:
        type: json
        data: |-
          {
            "id": "",
            "campo1": "",
            "campo2": ""
          }

docs: Descripcion del endpoint.
script:
  tests: |-
    test("Status is 200", function() {
      expect(res.getStatus()).to.equal(200);
    });

    test("Response is JSON", function() {
      const contentType = res.getHeader("Content-Type");
      expect(contentType).to.contain("application/json");
    });
```

### 8.3 POST Create

```yaml
info:
  name: Create
  type: http
  seq: 5
  tags:
    - <nombre-del-recurso>

http:
  method: POST
  url: "{{baseUrl}}/api/v1/projects/:project_pk/<recurso>/"
  params:
    - name: project_pk
      value: ""
      type: path
  body:
    type: json
  auth: inherit

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5

examples:
  - name: 201 Response
    request:
      url: "{{baseUrl}}/api/v1/projects/:project_pk/<recurso>/"
      method: POST
      params:
        - name: project_pk
          value: ""
          type: path
      body:
        type: json
        data: |-
          {
            "campo1": "",
            "campo2": ""
          }
    response:
      status: 201
      statusText: Created
      headers:
        - name: Content-Type
          value: application/json
      body:
        type: json
        data: |-
          {
            "id": "",
            "campo1": "",
            "campo2": ""
          }

docs: Descripcion del endpoint.
script:
  post-response: |-
    const body = res.getBody();
    if (body.id) {
      bru.setEnvVar("<recurso>_id", body.id);
    }
  tests: |-
    test("Status is 201", function() {
      expect(res.getStatus()).to.equal(201);
    });

    test("Response has id", function() {
      const body = res.getBody();
      expect(body.id).to.be.a("string");
    });
```

#### Reglas de body

- El body del request principal solo declara `type: json` (sin `data`)
- La estructura de campos va en el **example** (tanto en request como en response)
- El example del request muestra los campos de entrada
- El example del response muestra los campos de salida (puede incluir campos extra como `id`, `created`, etc.)

#### Reglas de post-response

- Todo Create endpoint guarda el `id` del recurso creado en la variable de environment correspondiente
- Formato: `bru.setEnvVar("<recurso>_id", body.id)`

### 8.4 PUT Update

Identico a POST Create pero con:
- `method: PUT`
- `seq: 7`
- `name: Update`
- Example: `200 Response` (no 201)
- Path param `id` adicional
- **Sin** script post-response (no guarda ID)
- Test: solo status 200

### 8.5 PATCH Partial Update

Identico a PUT Update pero con:
- `method: PATCH`
- `seq: 8`
- `name: Patch`

### 8.6 DELETE

```yaml
info:
  name: Delete
  type: http
  seq: 9
  tags:
    - <nombre-del-recurso>

http:
  method: DELETE
  url: "{{baseUrl}}/api/v1/projects/:project_pk/<recurso>/:id/"
  params:
    - name: project_pk
      value: ""
      type: path
    - name: id
      value: ""
      type: path
      description: "UUID del recurso."
  auth: inherit

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5

examples:
  - name: 204 Response
    description: No response body
    request:
      url: "{{baseUrl}}/api/v1/projects/:project_pk/<recurso>/:id/"
      method: DELETE
      params:
        - name: project_pk
          value: ""
          type: path
        - name: id
          value: ""
          type: path
    response:
      status: 204
      statusText: No Content
      body:
        type: text
        data: ""

docs: Descripcion del endpoint.
script:
  tests: |-
    test("Status is 204", function() {
      expect(res.getStatus()).to.equal(204);
    });
```

---

## 9. Scripts y Tests

### 9.1 Post-response scripts

Se usan para guardar datos del response en variables de environment.

| Endpoint         | Variable guardada  | Dato        |
|------------------|--------------------|-------------|
| Login            | `token`, `refresh_token` | `body.access`, `body.refresh` |
| Refresh Token    | `token`, `refresh_token` | `body.access`, `body.refresh` |
| Create <recurso> | `<recurso>_id`     | `body.id`   |

Formato del script para Login/Refresh Token (usar `runtime.scripts`):
```yaml
runtime:
  scripts:
    - type: after-response
      code: |-
        const body = res.getBody();
          if (body.access) {
            bru.setEnvVar("token", body.access);
          }
          if (body.refresh) {
            bru.setEnvVar("refresh_token", body.refresh);
          }
```

Formato del script para Create endpoints:
```yaml
script:
  post-response: |-
    const body = res.getBody();
    if (body.id) {
      bru.setEnvVar("<recurso>_id", body.id);
    }
```

### 9.2 Tests (assertions)

Cada endpoint lleva assertions bajo `script.tests`.

| Metodo | Status esperado | Assertions                                |
|--------|-----------------|-------------------------------------------|
| GET    | 200             | Status 200 + Content-Type JSON            |
| POST (create) | 201      | Status 201 + response tiene `id` (string) |
| POST (auth)   | 200      | Status 200                                |
| PUT    | 200             | Status 200                                |
| PATCH  | 200             | Status 200                                |
| DELETE | 204             | Status 204                                |

---

## 10. Tags

Cada endpoint lleva un tag que coincide con el **nombre de la carpeta padre** del recurso principal.

```yaml
tags:
  - projects      # para endpoints en projects/
  - incidents     # para endpoints en incidents/
  - auth          # para endpoints en auth/
```

Los endpoints en sub-carpetas usan el tag del recurso padre:
- `incidents/follow-ups/Create.yml` -> tag: `incidents`
- `expenses/categories/List.yml` -> tag: `expenses`
- `projects/participants/Add.yml` -> tag: `projects`

---

## 11. Settings (identicos en todos los endpoints)

```yaml
settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5
```

No modificar. Estos valores son constantes.

---

## 12. Checklist para crear un nuevo recurso

1. Crear carpeta `<recurso>/` con `folder.yml` (name, seq, auth: inherit, docs)
2. Si tiene sub-recursos, crear sub-carpeta con su `folder.yml`
3. Crear endpoints siguiendo las plantillas de la seccion 8
4. Nombres segun seccion 5, seq segun seccion 6
5. Auth: `inherit` o `none` segun seccion 7
6. Body: estructura en examples, no en request principal (seccion 8.3)
7. Query params en todos los List: ordering, page, page_size, search (seccion 8.1)
8. Post-response script en Create para guardar ID (seccion 9.1)
9. Tests/assertions en todos los endpoints (seccion 9.2)
10. Agregar variable `<recurso>_id` a **todos** los environments
11. Tags: nombre de la carpeta padre (seccion 10)
12. Docs: descripcion, permisos, constraints relevantes

## 13. Checklist para actualizar un endpoint existente

1. Verificar que el nombre sigue la convencion (seccion 5)
2. Verificar auth: `inherit` o `none` segun corresponda (seccion 7)
3. Verificar que el body solo tiene `type: json` sin data (seccion 8.3)
4. Verificar que los examples tienen la estructura actualizada (request + response)
5. Verificar que tiene tests/assertions (seccion 9.2)
6. Si es Create: verificar post-response script y variable en environments
7. Si es List: verificar query params minimos (ordering, page, page_size, search)
8. Verificar seq en orden CRUD (seccion 6)
9. Actualizar docs si cambio la funcionalidad
