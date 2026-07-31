# 06 · Seguridad

**Handbook de Estándares de Ingeniería — Ancient**

---

## 1. Referencia Base: OWASP Top 10

**[ESTÁNDAR]** El baseline de seguridad de Ancient es OWASP Top 10 como mínimo. Todo dev conoce los 10 riesgos y cómo mitigarlos en el stack que usa. Los proyectos con requisitos regulatorios adicionales (PCI-DSS, HIPAA, LFPDPPP) documentan los requisitos extra en el paquete de pase.

---

## 2. Manejo de Secrets

### 2.1 Regla absoluta

**[ESTÁNDAR]** Ningún secret, credencial, API key, token, contraseña, o dato sensible se versiona en Git. Nunca. Sin excepciones. Sin "es solo para dev". Sin "lo quito después".

### 2.2 `.gitignore` obligatorio

**[ESTÁNDAR]** Todo repositorio incluye en `.gitignore`:

```gitignore
# Environment files
.env
.env.*
!.env.example

# Credentials
*.pem
*.key
*.p12
*.pfx
```

**[ESTÁNDAR]** Todo repositorio incluye un `.env.example` con las variables necesarias (sin valores reales) para que un dev nuevo pueda configurar su ambiente:

```env
# .env.example — NO incluir valores reales
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
JWT_SECRET=your-secret-here
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

### 2.3 Gestión de secrets en producción

**[ESTÁNDAR]** Los secrets en ambientes desplegados (dev/qa/prod) se gestionan con un gestor de secretos:

| Nube | Herramienta |
|------|-------------|
| GCP | External Secrets Operator → GCP Secret Manager |
| AWS | AWS Secrets Manager |

**[ESTÁNDAR]** Para inyección de secrets en runtime, usar `@nestjs/config` con validación Joi. La app no arranca si falta una variable requerida:

```typescript
// config/env.validation.ts
import * as Joi from 'joi';

export const envValidationSchema = Joi.object({
  DATABASE_URL: Joi.string().required(),
  JWT_SECRET: Joi.string().min(32).required(),
  PORT: Joi.number().default(3000),
  NODE_ENV: Joi.string().valid('development', 'staging', 'production').required(),
});
```

### 2.4 Variables de CI/CD

**[ESTÁNDAR]** Tokens y credenciales de CI (SONAR_TOKEN, GIT_PUSH_TOKEN, registry credentials) se configuran como variables enmascaradas/secretas en el proveedor de CI (GitHub Actions secrets, GitLab CI variables masked). Nunca en archivos del repo.

---

## 3. Autenticación y Autorización

### 3.1 JWT como estándar

**[ESTÁNDAR]** El mecanismo de autenticación estándar de Ancient es JWT (JSON Web Tokens), implementado con `@nestjs/jwt` + `@nestjs/passport` + `passport-jwt`.

**Reglas de JWT:**
- **[ESTÁNDAR]** Tokens de acceso con expiración corta (15-30 minutos máximo).
- **[ESTÁNDAR]** Refresh tokens con expiración más larga (7-30 días), almacenados de forma segura.
- **[ESTÁNDAR]** Hashing de passwords con `bcrypt` (salt rounds ≥ 10).
- **[ESTÁNDAR]** Nunca almacenar tokens en localStorage si la app maneja datos sensibles — usar httpOnly cookies.
- **[RECOMENDADO]** Para proyectos con múltiples servicios, considerar OAuth 2.0 con servidor de autorización centralizado.

### 3.2 NestJS Guards para autorización

**[ESTÁNDAR]** La autorización se implementa con Guards de NestJS (`@UseGuards`, `CanActivate`). Los Guards verifican permisos antes de que la request llegue al controller.

---

## 4. Headers de Seguridad — Helmet

**[ESTÁNDAR]** Todo backend NestJS expuesto a internet usa `helmet`:

```typescript
// main.ts
import helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(helmet());
  // ...
}
```

Helmet configura automáticamente: X-Content-Type-Options, X-Frame-Options, X-XSS-Protection, Strict-Transport-Security, y otros headers defensivos.

---

## 5. Validación de Inputs

**[ESTÁNDAR]** Toda input del usuario o de sistemas externos se valida antes de procesarse. En NestJS:

- `ValidationPipe` global habilitado.
- DTOs con decoradores de `class-validator` para cada endpoint.
- `transform: true` y `whitelist: true` para eliminar propiedades no declaradas.

```typescript
// main.ts
app.useGlobalPipes(new ValidationPipe({
  transform: true,
  whitelist: true,
  forbidNonWhitelisted: true,
}));
```

---

## 6. Auditoría de Dependencias

### 6.1 Snyk

**[ESTÁNDAR]** Todo repositorio tiene Snyk configurado para escaneo automático de dependencias, integrado en el pipeline de CI y bloqueante para vulnerabilidades críticas.

### 6.2 Dependabot

**[RECOMENDADO]** Dependabot habilitado a nivel de organización en GitHub para generar PRs automáticos de actualización de dependencias.

### 6.3 Repositorio interno de paquetes auditados

**[RECOMENDADO]** Un repositorio interno (`ancient-approved-packages`) con paquetes de terceros auditados y aprobados por Arquitectura. Los TLs pueden usar cualquier paquete de la lista sin aprobación adicional. Paquetes fuera de la lista requieren revisión.

### 6.4 `npm audit` / `yarn audit`

**[ESTÁNDAR]** Correr `npm audit` como parte del pipeline de CI. Vulnerabilidades críticas y altas bloquean el deploy. Medias y bajas se reportan y se priorizan en el backlog de deuda técnica.

---

## 7. SAST / DAST / SCA

**[ESTÁNDAR]**

| Tipo | Herramienta | Alcance |
|------|-------------|---------|
| SAST (análisis estático) | SonarQube/SonarCloud | Todos los repos con CI |
| SCA (composición de dependencias) | Snyk + Dependabot | Todos los repos |
| DAST (análisis dinámico) | Según proyecto | Proyectos con datos sensibles |
| SBOM (bill of materials) | Según proyecto | Proyectos regulados |

---

## 8. Datos Sensibles

### 8.1 Clasificación

**[ESTÁNDAR]** Todo proyecto clasifica los datos que maneja al inicio (en el paquete de pase):

| Nivel | Tipo de datos | Requisitos |
|-------|--------------|------------|
| Público | Información no sensible | Estándares base |
| Interno | Datos de operación de Ancient | Estándares base + control de acceso |
| Confidencial | Datos de cliente, PII | Todo lo anterior + encriptación en reposo y tránsito |
| Regulado | Datos financieros (PCI), salud (HIPAA), datos personales (LFPDPPP) | Todo lo anterior + requisitos regulatorios específicos documentados |

### 8.2 Reglas por nivel

**[ESTÁNDAR]** Nivel Confidencial y Regulado:
- Encriptación en tránsito (TLS 1.2+, sin excepciones).
- Encriptación en reposo (AES-256 o equivalente para bases de datos y storage).
- Logs sanitizados (nunca loguear datos sensibles — ni parciales, ni en dev).
- Acceso restringido por principio de mínimo privilegio.

### 8.3 Regla para prompts de AI

**[ESTÁNDAR]** No se pegan datos reales de clientes, credenciales, ni PII en prompts de herramientas de AI (Cursor, ChatGPT, Claude, etc.). Si se necesita usar datos para desarrollo/debugging con AI, se usan datos ficticios o anonimizados.

---

*Módulo 06 del Handbook de Estándares de Ingeniería — Ancient.*
