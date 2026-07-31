# 01 · Principios Generales

**Handbook de Estándares de Ingeniería — Ancient**

---

## 1. Stack de Ancient

### 1.1 Stack primario

**[REFERENCIA]** Ancient es una empresa TypeScript-first. El stack primario es:

| Capa | Tecnología | Uso principal |
|------|-----------|---------------|
| Backend | Node.js + TypeScript, NestJS (adaptador Fastify) | Servicios y APIs |
| Frontend web | Vue 3 + single-spa (micro-frontends) | Aplicaciones de escala media/grande |
| Frontend admin | Angular | Paneles administrativos |
| Mobile | Flutter | Apps nativas (wallets, apps de cliente) |
| IaC | Terraform | Infraestructura como código (AWS y GCP) |
| Contenedorización | Docker + Helm sobre Kubernetes | Empaquetado y despliegue |
| Dominio financiero | Java + Spring | Servicios de dominio financiero: legacy y también desarrollo nuevo cuando el core bancario del cliente lo impone |

### 1.2 Preferencia de stack para proyectos nuevos

**[ESTÁNDAR]** Salvo restricción del cliente o justificación técnica documentada en el paquete de pase, los proyectos nuevos usan:

- **Backend:** NestJS con adaptador Fastify, TypeScript.
- **Frontend web:** Vue 3 con single-spa (arquitectura micro-frontend) para aplicaciones de escala media/grande. Angular para paneles administrativos simples.
- **Mobile:** Flutter.
- **IaC:** Terraform.
- **Contenedorización:** Docker + Helm sobre Kubernetes.

**[REFERENCIA]** Ancient no descarta ningún stack. Se puede usar cualquier tecnología si el proyecto lo requiere (restricción del cliente, integración con sistemas existentes, dominio técnico específico). La desviación del stack preferido se documenta en el paquete de pase con justificación. La justificación la aprueba Arquitectura antes del arranque del proyecto, no después. Si el pase ya salió y a medio proyecto se cambia el stack, eso escala como desviación (ver Gobernanza Técnica, Sección 7).

---

## 2. Arquitectura de Referencia

### 2.1 Backend — NestJS Modular por Dominio

**[ESTÁNDAR]** El patrón de arquitectura backend de Ancient es modular por bounded context. Cada dominio de negocio tiene su propia carpeta dentro de `src/` con la siguiente estructura:

```
src/
├── auth/                    # Módulo de autenticación
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── auth.module.ts
│   ├── dto/
│   ├── guards/
│   └── strategies/
├── customers/               # Dominio: clientes
│   ├── customers.controller.ts
│   ├── customers.service.ts
│   ├── customers.module.ts
│   ├── dto/
│   └── entities/
├── operations/              # Dominio: operaciones
│   ├── ...
├── config/                  # Configuración global
│   ├── env.validation.ts    # Joi schema
│   └── ...
├── common/                  # Utilidades compartidas
│   ├── filters/
│   ├── interceptors/
│   ├── pipes/
│   └── decorators/
└── main.ts
```

Además de `src/`, en la raíz del repo:

```
test/                        # Tests e2e (Nest los genera aquí, no en src/)
database/
├── migrations/              # Migraciones versionadas
└── seeds/                   # Datos semilla para dev/qa
docs/                        # Diagramas y contratos no-REST (ver módulo 09)
```

### 2.2 Backend — Monorepo NestJS (proyectos de plataforma)

**[RECOMENDADO]** Para proyectos con múltiples servicios que comparten lógica (3+ servicios con librerías comunes):

```
apps/
├── transaction/
├── payment-links/
├── notification/
└── web-backend/
libs/
├── core-banking/
├── oauth/
├── cache-provider/
├── common-core/
└── email-notification/
```

### 2.3 Frontend — Micro-frontends con single-spa

**[ESTÁNDAR]** La arquitectura frontend de Ancient es micro-frontends con single-spa + Vue 3:

- **Root:** Aplicación contenedora que orquesta los MFEs.
- **Shell MFEs:** Header, menu, sidebar (compartidos).
- **Domain MFEs:** Uno por dominio funcional (login, dashboard, customers, operations, reports, etc.).
- Cada MFE tiene su propio repo, pipeline, y deploy independiente.

Angular se reserva para paneles administrativos simples donde single-spa sería over-engineering. Un frontend de un solo dominio, sin necesidad de deploys independientes y con menos de 5 vistas, no requiere MFEs: se hace como SPA normal (Vue o Angular) y se documenta en el pase.

### 2.4 Patrones transversales NestJS

**[ESTÁNDAR]** Los siguientes patrones son obligatorios en todo backend NestJS:

| Patrón | Uso |
|--------|-----|
| Guards (`@UseGuards`, `CanActivate`) | Autenticación y autorización |
| ValidationPipe global + DTOs | Validación de inputs en toda la app |
| Interceptors | Logging, transformación de respuestas |
| Exception Filters (`@Catch`) | Manejo centralizado de errores |
| `@nestjs/config` + Joi validation | Configuración de entorno tipada y validada |
| Health checks (`@nestjs/terminus`) | Endpoints `/health` (liveness) y `/health/ready` (readiness) para K8s |

---

## 3. Política de Versiones

### 3.1 Versiones mínimas soportadas

**[ESTÁNDAR]** Todo proyecto usa versiones dentro del rango soportado:

| Tecnología | Versión mínima | Versión recomendada | Nota |
|------------|---------------|---------------------|------|
| Node.js | 22 LTS | 24 LTS | `.nvmrc` obligatorio en todo repo |
| TypeScript | 5.0 | 5.8+ | |
| NestJS | 10 | 11 | Nest 10 solo para repos existentes; proyecto nuevo arranca en 11 |
| Angular | 19 | 19+ | |
| Vue | 3.x | 3.x | |

*Última revisión de esta tabla: [DD/MM/AAAA]. Se revisa en cada versión del Handbook.*

### 3.2 `.nvmrc` obligatorio

**[ESTÁNDAR]** Todo repositorio incluye un archivo `.nvmrc` en la raíz que fija la versión de Node.js, de modo que todo el equipo use la misma versión y se eviten bugs no reproducibles.

```
# .nvmrc
22.14.0
```

**[ESTÁNDAR]** La versión del `.nvmrc` debe ser la misma que usa la imagen base del Dockerfile y la que se configura en el pipeline de CI. Si las tres no coinciden, es un hallazgo.

### 3.3 `engines` en `package.json`

**[RECOMENDADO]** Además de `.nvmrc`, el `package.json` incluye el campo `engines` para que npm/yarn alerte si la versión de Node no es compatible:

```json
{
  "engines": {
    "node": ">=22.0.0",
    "npm": ">=10.0.0"
  }
}
```

Para que `engines` bloquee de verdad y no solo tire un warning, agregar también `.npmrc` con `engine-strict=true`.

---

## 4. Contenedorización

**[ESTÁNDAR]** Todo servicio desplegable se contenedoriza con Docker.

**Reglas mínimas del Dockerfile:**

- Multi-stage build (etapa de build separada de la etapa de runtime). La imagen final no lleva devDependencies ni código fuente TS.
- La imagen corre con usuario no-root (`USER node` o equivalente).
- Imagen base con tag fijo y versionado (`node:22.14.0-alpine`), nunca `latest`.
- `.dockerignore` en el repo (mínimo `node_modules`, `.env`, `.git`, `coverage`, `dist`).

**[RECOMENDADO]** Para proyectos sobre Kubernetes: Helm charts para la gestión de configuración por ambiente (`helm/values-dev.yaml`, `values-qa.yaml`, `values.yaml`). Definir siempre `resources.requests` y `resources.limits` en los values; un pod sin límites se puede comer el nodo.

---

*Módulo 01 del Handbook de Estándares de Ingeniería — Ancient.*
