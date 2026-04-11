# ObraClara — Bruno API Collection

Coleccion de endpoints para la API de [ObraClara](https://github.com/YuniFed), gestionada con [Bruno](https://www.usebruno.com/).

## Requisitos

- [Bruno](https://www.usebruno.com/downloads) (v1.x o superior)

## Configuracion

1. Abrir Bruno
2. **Open Collection** -> seleccionar la carpeta `collections/ObraClaraAPI/`
3. Ir a **Environments** y configurar las variables del environment que quieras usar (Local, Staging)
4. Configurar `baseUrl` (ej: `http://127.0.0.1:8000`)
5. Ejecutar **auth/Login** con tus credenciales — el token se guarda automaticamente

## Estructura

```
collections/ObraClaraAPI/
  environments/         # Local, Staging
  auth/                 # Login, Register, tokens, password, perfil
  projects/             # CRUD de proyectos
    participants/       # Participantes del proyecto
  stages/               # Etapas de obra
  progress/             # Avance de obra
  expenses/             # Gastos
    categories/         # Categorias de gasto
  incidents/            # Incidentes
    follow-ups/         # Seguimientos de incidente
  invitations/          # Invitaciones por token
  suppliers/            # Proveedores
  mediafiles/           # Archivos (imagenes, documentos)
  reports/              # Reportes con snapshots
```

## Features

- **Token automatico**: Login y Refresh Token guardan los JWT en el environment via post-response scripts
- **Auto-refresh**: Pre-request script a nivel coleccion que detecta JWT expirado y hace refresh antes de cada request
- **Auth heredada**: Todos los endpoints heredan Bearer token de la coleccion. Los publicos (login, register, etc.) usan `auth: none`
- **Variables de ID**: Los Create endpoints guardan el ID del recurso creado en el environment para usar en requests siguientes
- **Assertions**: Cada endpoint tiene tests de status code y validaciones basicas
- **Examples**: El body de los requests esta documentado en examples (request + response esperado)
- **Paginacion**: Todos los List endpoints tienen query params: `ordering`, `page`, `page_size`, `search`

## Variables de environment

Todas las variables son `secret: true` para evitar conflictos de merge y proteger datos sensibles. Los valores se configuran desde Bruno y no se commitean.

| Variable         | Descripcion                          |
|------------------|--------------------------------------|
| `baseUrl`        | URL base de la API                   |
| `token`          | JWT access token (se llena con Login)|
| `refresh_token`  | JWT refresh token (se llena con Login)|
| `project_pk`     | ID del ultimo proyecto creado        |
| `stage_id`       | ID de la ultima etapa creada         |
| `participant_id` | ID del ultimo participante agregado  |
| `expense_id`     | ID del ultimo gasto creado           |
| `incident_id`    | ID del ultimo incidente creado       |
| `report_id`      | ID del ultimo reporte creado         |
| `supplier_id`    | ID del ultimo proveedor creado       |

## Flujo de uso tipico

```
1. Login              -> guarda token y refresh_token
2. Create Project      -> guarda project_pk
3. Create Stage        -> guarda stage_id (usa project_pk)
4. Create Progress     -> guarda progress_id (usa project_pk)
5. Create Expense      -> guarda expense_id (usa project_pk)
6. ...los demas endpoints ya tienen los IDs disponibles
```

## Estandar

Ver [STANDARD.md](STANDARD.md) para las convenciones de nomenclatura, estructura, auth, scripts, tests y como crear/actualizar endpoints.
