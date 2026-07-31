# 09 · Documentación

**Handbook de Estándares de Ingeniería — Ancient**

---

## 1. Qué se Documenta

**[ESTÁNDAR]** Todo proyecto debe tener como mínimo:

| Documento | Dónde vive | Cuándo se actualiza |
|-----------|-----------|---------------------|
| README.md | Raíz del repo | Con cada cambio significativo de setup o arquitectura |
| API docs (Swagger/OpenAPI) | Generado desde código (`@nestjs/swagger`) | Con cada cambio de API |
| Diagramas de flujo de procesos críticos | `docs/` en el repo | Cuando cambia el flujo |
| `.env.example` | Raíz del repo | Cuando se agrega/elimina una variable de entorno |
| `.nvmrc` | Raíz del repo | Cuando cambia la versión de Node |

---

## 2. README Template

**[ESTÁNDAR]** Todo repo usa el siguiente template como mínimo para estandarizar el contenido base.

```markdown
# [Nombre del Proyecto]

> [Descripción en una línea: qué hace este servicio/app y para quién]

## Stack

- **Runtime:** Node.js [versión] / Java [versión] / etc.
- **Framework:** NestJS [versión] / Angular [versión] / etc.
- **Base de datos:** PostgreSQL / MongoDB / etc.
- **Infraestructura:** AWS / GCP / Docker + Helm

## Requisitos previos

- Node.js [versión] (ver `.nvmrc`)
- [Otras dependencias: Docker, etc.]

## Setup local

```bash
# Clonar
git clone [url]

# Instalar dependencias
npm install

# Configurar variables de entorno
cp .env.example .env
# Editar .env con tus valores locales

# Correr en desarrollo
npm run start:dev

# Correr tests
npm run test
npm run test:cov
```

## Estructura del proyecto

```
src/
├── [módulo-1]/    # [Descripción breve]
├── [módulo-2]/    # [Descripción breve]
└── common/        # Utilidades compartidas
```

## API

- Swagger UI: `http://localhost:[port]/api`
- OpenAPI spec: `http://localhost:[port]/api-json`

## Pipeline

- **CI:** [GitHub Actions / GitLab CI]
- **Ambientes:** dev / qa / prod
- **Deploy:** [Automático a dev, manual a prod]

## Contacto

- **TL:** [Nombre]
- **PM:** [Nombre]
- **Arquitectura:** [Nombre de quien diseñó]
```

---

## 3. API Docs — Swagger/OpenAPI

**[ESTÁNDAR]** Todo backend que expone APIs REST usa `@nestjs/swagger` (NestJS) o `springdoc` (Java) para generar documentación automática desde el código.

**Reglas:**
- Todos los endpoints documentados con decoradores (`@ApiOperation`, `@ApiResponse`, `@ApiProperty` en DTOs).
- Swagger UI accesible en `/api` en ambientes de desarrollo y QA. Deshabilitado en producción (a menos que el cliente lo requiera).
- Para APIs que no son REST (WebSockets, GraphQL, gRPC), documentar contratos en `docs/` del repo.

**[RECOMENDADO]** Mantener una colección de Postman como complemento para testing manual y onboarding. Versionarla en el repo (`docs/postman/`).

---

*Módulo 09 del Handbook de Estándares de Ingeniería — Ancient.*
