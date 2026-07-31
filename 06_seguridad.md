# 06 · Seguridad

**Handbook de Estándares de Ingeniería — Ancient**

---

## 1. Referencia Base: OWASP Top 10

**[ESTÁNDAR]** El baseline de seguridad de Ancient es OWASP Top 10 como mínimo. Todo dev conoce los 10 riesgos y cómo mitigarlos en el stack que usa. Los proyectos con requisitos regulatorios adicionales (PCI-DSS, HIPAA, LFPDPPP) documentan los requisitos extra en el paquete de pase.

**[ESTÁNDAR]** Para proyectos que exponen APIs, que son la mayoría, aplica además el **OWASP API Security Top 10**. Los riesgos específicos de API (BOLA, mass assignment, falta de rate limiting) no están todos cubiertos en el Top 10 genérico.

**[ESTÁNDAR]** El conocimiento del baseline se valida en el onboarding: es un punto del checklist del módulo 11 que el TL firma.

---

## 2. Manejo de Secrets

### 2.1 Regla absoluta

**[ESTÁNDAR]** Ningún secret, credencial, API key, token, contraseña, o dato sensible se versiona en Git. Nunca. Sin excepciones. Sin "es solo para dev". Sin "lo quito después".

**[ESTÁNDAR]** Un secret que llegó a un repo se considera comprometido aunque el commit se haya borrado: se **rota**, no se limpia. Borrar el commit no sirve, queda en los forks, en los clones locales, en la caché de la plataforma y en los logs de CI.

**Procedimiento cuando pasa:**
1. Rotar la credencial expuesta de inmediato. Este paso va primero, antes de tocar el repo.
2. Avisar al TL y a Cibersec el mismo día.
3. Limpiar el historial si aplica y revisar los accesos hechos con esa credencial mientras estuvo expuesta.

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

# Cloud / IaC
*.tfvars
!*.tfvars.example
.terraform/
terraform.tfstate*
*-service-account*.json
.aws/
kubeconfig*
```

**[ESTÁNDAR]** Todo repositorio incluye un `.env.example` con las variables necesarias (sin valores reales) para que un dev nuevo pueda configurar su ambiente:

```env
# .env.example — NO incluir valores reales
DATABASE_URL=postgresql://<user>:<password>@localhost:5432/<dbname>
JWT_SECRET=<generar-con-openssl-rand-base64-32>
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

**[ESTÁNDAR]** El `.env.example` se mantiene sincronizado con el schema de Joi: si una variable se agrega al schema, se agrega al `.env.example` en el mismo PR. Es punto del checklist de revisión del módulo 04.

### 2.3 Gestión de secrets en producción

**[ESTÁNDAR]** Los secrets en ambientes desplegados (dev/qa/prod) se gestionan con un gestor de secretos:

| Nube | Herramienta |
|------|-------------|
| GCP | External Secrets Operator → GCP Secret Manager |
| AWS | AWS Secrets Manager |

**[ESTÁNDAR]** Reglas comunes a las dos nubes:
- Los secrets de `prod` no se comparten con `dev` ni `qa`. Un secret por ambiente, sin reuso.
- Ningún dev tiene acceso de lectura a los secrets de `prod`. La app los lee, las personas no.
- Rotación al menos anual, y de inmediato cuando alguien con acceso sale del proyecto o de la empresa.

**[ESTÁNDAR]** Para inyección de secrets en runtime, usar `@nestjs/config` con validación Joi. La app no arranca si falta una variable requerida:

```typescript
// config/env.validation.ts
import * as Joi from 'joi';

export const envValidationSchema = Joi.object({
  DATABASE_URL: Joi.string().required(),
  JWT_SECRET: Joi.string().min(32).required(),
  PORT: Joi.number().default(3000),
  // NODE_ENV mantiene los valores que esperan las librerías del ecosistema
  NODE_ENV: Joi.string().valid('development', 'production', 'test').required(),
  // APP_ENV identifica el ambiente de Ancient (ver módulo 07)
  APP_ENV: Joi.string().valid('dev', 'qa', 'prod').required(),
});
```

### 2.4 Variables de CI/CD

**[ESTÁNDAR]** Tokens y credenciales de CI (SONAR_TOKEN, GIT_PUSH_TOKEN, registry credentials) se configuran como variables enmascaradas/secretas en el proveedor de CI (GitHub Actions secrets, GitLab CI variables masked). Nunca en archivos del repo.

**[ESTÁNDAR]** Los secrets de CI no se exponen a pipelines disparados por PRs de forks, y los jobs que los usan no imprimen su valor, ni con `set -x` ni con `echo` de depuración. El enmascarado del proveedor no cubre un secret transformado: si lo pasas por `base64` o lo concatenas, sale en claro en el log.

---

## 3. Autenticación y Autorización

### 3.1 JWT como estándar

**[ESTÁNDAR]** El mecanismo de autenticación estándar de Ancient es JWT (JSON Web Tokens), implementado con `@nestjs/jwt` + `@nestjs/passport` + `passport-jwt`.

**Reglas de JWT:**
- **[ESTÁNDAR]** Tokens de acceso con expiración corta (15-30 minutos máximo).
- **[ESTÁNDAR]** Refresh tokens con expiración más larga (7-30 días), almacenados de forma segura.
- **[ESTÁNDAR]** Hashing de passwords con `bcrypt` (salt rounds ≥ 10). Para proyectos nuevos, ≥ 12.
- **[ESTÁNDAR]** Nunca almacenar tokens en localStorage si la app maneja datos sensibles — usar httpOnly cookies.
- **[ESTÁNDAR]** Algoritmo de firma explícito en la verificación (`algorithms: ['RS256']` o `['HS256']`). Nunca aceptar el algoritmo que venga en el header del token, y nunca permitir `alg: none`.
- **[ESTÁNDAR]** Validar `iss` y `aud` además de la firma y la expiración. Un token válido emitido para otro servicio no debe servir en el nuestro.
- **[ESTÁNDAR]** Los refresh tokens se guardan hasheados, no en claro, y se invalidan al usarse (rotación) y al cerrar sesión. Debe existir forma de revocarlos.
- **[ESTÁNDAR]** El payload del JWT no lleva datos sensibles. Un JWT va firmado, no cifrado: cualquiera que lo intercepte lo lee con un base64.
- **[RECOMENDADO]** Para proyectos con múltiples servicios, considerar OAuth 2.0 con servidor de autorización centralizado.

**[ESTÁNDAR]** Política de contraseñas para las apps que manejan credenciales propias:
- Longitud mínima de 12 caracteres, sin obligar rotación periódica.
- Bloqueo temporal por intentos fallidos.
- MFA obligatorio en paneles administrativos y en cualquier acceso con privilegios elevados.

### 3.2 NestJS Guards para autorización

**[ESTÁNDAR]** La autorización se implementa con Guards de NestJS (`@UseGuards`, `CanActivate`). Los Guards verifican permisos antes de que la request llegue al controller.

**[ESTÁNDAR]** Deny by default: el guard de autenticación se registra de forma global y los endpoints públicos se marcan explícitamente (`@Public()`). Nunca al revés. Si proteger es opt-in, el endpoint que se olvide marcar queda abierto y nadie se entera.

**[ESTÁNDAR]** Autenticar no es autorizar. Todo endpoint que reciba un identificador de recurso (`/customers/:id`, `/accounts/:id/movements`) valida que el recurso pertenezca al usuario del token, no solo que el token sea válido.

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

**[ESTÁNDAR]** Además de helmet, en todo backend expuesto:
- **CORS explícito** con lista blanca de orígenes por ambiente. Nunca `origin: '*'` en un servicio autenticado.
- **Rate limiting** (`@nestjs/throttler`), con límite más estricto en los endpoints de login y de recuperación de contraseña.
- **Límite de tamaño de payload** (`bodyParser` limit), para no aceptar un JSON de 50 MB que tumbe el proceso.
- La versión del framework no se anuncia (deshabilitar el header `x-powered-by`).

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

**[ESTÁNDAR]** La validación es de lista blanca: se define qué se acepta (tipo, formato, rango, longitud máxima), no qué se rechaza. Toda cadena que venga del exterior lleva longitud máxima declarada.

**[ESTÁNDAR]** Validar también lo que sale: ningún endpoint retorna la entidad de base de datos directamente. Se mapea a un DTO de respuesta con los campos explícitos, para no filtrar columnas que se agreguen después (hash de password, notas internas, banderas de riesgo).

---

## 6. Auditoría de Dependencias

### 6.1 Snyk

**[ESTÁNDAR]** Todo repositorio tiene Snyk configurado para escaneo automático de dependencias, integrado en el pipeline de CI y bloqueante para vulnerabilidades críticas y altas.

### 6.2 Dependabot

**[RECOMENDADO]** Dependabot habilitado a nivel de organización en GitHub para generar PRs automáticos de actualización de dependencias.

**[RECOMENDADO]** Configurarlo agrupado (`groups` en el `dependabot.yml`) y con cadencia semanal, separando las actualizaciones de seguridad (que se atienden de inmediato) de las de versión menor (que se juntan en un `chore/` por sprint). Sin agrupar, abre 30 PRs y el equipo deja de leerlos.

### 6.3 Repositorio interno de paquetes auditados

**[RECOMENDADO]** Un repositorio interno (`ancient-approved-packages`) con paquetes de terceros auditados y aprobados por Arquitectura. Los TLs pueden usar cualquier paquete de la lista sin aprobación adicional. Paquetes fuera de la lista requieren revisión, con un SLA de respuesta de 48 horas hábiles.

### 6.4 `npm audit` / `yarn audit`

**[ESTÁNDAR]** Correr `npm audit` como parte del pipeline de CI. Vulnerabilidades críticas y altas bloquean el deploy. Medias y bajas se reportan y se priorizan en el backlog de deuda técnica.

**[ESTÁNDAR]** El lockfile (`package-lock.json` o `yarn.lock`) se versiona siempre, y en CI se instala con `npm ci`, no con `npm install`, para que el build sea reproducible y nadie meta una versión distinta a la auditada.

---

## 7. SAST / DAST / SCA

**[ESTÁNDAR]**

| Tipo | Herramienta | Alcance |
|------|-------------|---------|
| SAST (análisis estático) | SonarCloud | Todos los repos con CI |
| SCA (composición de dependencias) | Snyk + Dependabot | Todos los repos |
| Secret scanning | Gitleaks o el del proveedor | Todos los repos, bloqueante en PR |
| DAST (análisis dinámico) | Según proyecto | Nivel Confidencial y Regulado (ver Sección 8) |
| SBOM (bill of materials) | Según proyecto | Nivel Regulado |

---

## 8. Datos Sensibles

### 8.1 Clasificación

**[ESTÁNDAR]** Todo proyecto clasifica los datos que maneja al inicio (en el paquete de pase):

| Nivel | Tipo de datos | Ejemplos | Requisitos |
|-------|--------------|----------|------------|
| Público | Información no sensible | Catálogos, contenido de marketing | Estándares base |
| Interno | Datos de operación de Ancient | Métricas de proyecto, documentación interna | Estándares base + control de acceso |
| Confidencial | Datos de cliente, PII | Nombre, correo, teléfono, dirección | Todo lo anterior + encriptación en reposo y tránsito |
| Regulado | Datos financieros (PCI), salud (HIPAA), datos personales (LFPDPPP) | CLABE, número de tarjeta, RFC, CURP, saldos, movimientos | Todo lo anterior + requisitos regulatorios específicos documentados |

### 8.2 Reglas por nivel

**[ESTÁNDAR]** Nivel Confidencial y Regulado:
- Encriptación en tránsito (TLS 1.2+, sin excepciones).
- Encriptación en reposo (AES-256 o equivalente para bases de datos y storage).
- Logs sanitizados (nunca loguear datos sensibles — ni parciales, ni en dev).
- Acceso restringido por principio de mínimo privilegio.
- Enmascarado en la respuesta y en la UI cuando el dato completo no es necesario (`**** **** **** 4321`).
- Bitácora de acceso (quién consultó qué dato y cuándo) para nivel Regulado.
- Backups cifrados y con la misma restricción de acceso que el dato original.

**[ESTÁNDAR]** La sanitización de logs no se deja a criterio del dev: el logger se configura con una lista de campos que siempre se redactan (`password`, `token`, `authorization`, `clabe`, `card`, `cvv`, `rfc`, `curp`), a nivel de configuración del logger y no de cada llamada.

### 8.3 Regla para prompts de AI

**[ESTÁNDAR]** No se pegan datos reales de clientes, credenciales, ni PII en prompts de herramientas de AI (Cursor, ChatGPT, Claude, etc.). Si se necesita usar datos para desarrollo/debugging con AI, se usan datos ficticios o anonimizados.

**[ESTÁNDAR]** La regla no aplica solo a lo que el dev escribe en el prompt: aplica también al contexto que la herramienta lee sola (archivos abiertos, indexado del repositorio, herramientas de AI review conectadas al repo). Por eso las fixtures y los seeds tampoco llevan datos reales, ver módulo 05 Sección 6.

---

*Módulo 06 del Handbook de Estándares de Ingeniería — Ancient.*
