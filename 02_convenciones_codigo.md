# 02 · Convenciones de Código

**Handbook de Estándares de Ingeniería — Ancient**

---

## 1. Principio General

**[ESTÁNDAR]** Ancient es agnóstico de stack para estas convenciones — las reglas aplican independientemente del lenguaje. Donde el framework (NestJS, Angular, etc.) impone una convención, esa convención es el estándar.

---

## 2. Naming Conventions

### 2.1 Reglas generales (agnósticas de stack)

**[ESTÁNDAR]**

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| Variables, funciones, métodos | camelCase | `getUserBalance`, `paymentAmount` |
| Clases, interfaces, types, enums | PascalCase | `PaymentService`, `UserDto`, `TransactionStatus` |
| Constantes globales | UPPER_SNAKE_CASE | `MAX_RETRY_ATTEMPTS`, `DEFAULT_TIMEOUT_MS` |
| Archivos TypeScript | kebab-case o dot-notation por rol | `payment.service.ts`, `user.controller.ts` |
| Carpetas | kebab-case | `payment-links/`, `core-banking/` |
| Variables de entorno | UPPER_SNAKE_CASE | `DATABASE_URL`, `JWT_SECRET` |
| Branches de Git | kebab-case con prefijo | `feature/PE-123-customer-validation` |

### 2.2 NestJS — Convenciones por rol

**[ESTÁNDAR]** NestJS impone sufijos por rol de archivo:

| Rol | Sufijo | Ejemplo |
|-----|--------|---------|
| Módulo | `.module.ts` | `customers.module.ts` |
| Controller | `.controller.ts` | `customers.controller.ts` |
| Service | `.service.ts` | `customers.service.ts` |
| DTO | `.dto.ts` | `create-customer.dto.ts` |
| Entity | `.entity.ts` | `customer.entity.ts` |
| Guard | `.guard.ts` | `jwt-auth.guard.ts` |
| Interceptor | `.interceptor.ts` | `logging.interceptor.ts` |
| Filter | `.filter.ts` | `http-exception.filter.ts` |
| Pipe | `.pipe.ts` | `validation.pipe.ts` |
| Test | `.spec.ts` | `customers.service.spec.ts` |

---

## 3. Estructura de Carpetas

### 3.1 Backend NestJS — Modular por dominio

**[ESTÁNDAR]** Documentado en detalle en el módulo 01 (Principios Generales), Sección 2.1. Resumen: una carpeta por bounded context dentro de `src/`, con sub-carpetas por rol (dto/, entities/, guards/).

### 3.2 Frontend Vue/MFE

**[RECOMENDADO]** Estructura estándar de micro-frontend single-spa:

```
src/
├── components/       # Componentes reutilizables
├── views/            # Vistas/páginas
├── composables/      # Hooks de Vue 3
├── services/         # Llamadas a APIs
├── stores/           # Estado (Pinia)
├── types/            # Interfaces/tipos
├── utils/            # Utilidades
└── main.ts           # Bootstrap
```

---

## 4. Linting

### 4.1 ESLint + Prettier

**[ESTÁNDAR]** Todo proyecto usa ESLint + Prettier. La configuración es compartida, no reinventada por proyecto.

### 4.2 Configuración compartida

**[ESTÁNDAR]** Todos los repos consumen la configuración organizacional `@ancient/eslint-config`, publicada como paquete interno. Esto garantiza que las reglas de lint sean idénticas en todos los proyectos y que un cambio de regla se propague automáticamente.

### 4.3 Lint en CI

**[ESTÁNDAR]** ESLint corre en el pipeline de CI. Los errores de lint bloquean el merge.

### 4.4 Format on save

**[RECOMENDADO]** Configurar Prettier como formatter por defecto en el IDE (VSCode/Cursor) con format-on-save habilitado. Incluir `.vscode/settings.json` en el repo:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

---

## 5. Manejo de Errores

### 5.1 NestJS — Exception Filters

**[ESTÁNDAR]** El patrón de manejo de errores en NestJS es Exception Filters (`@Catch`) con ValidationPipe global.

**Reglas:**
- **[ESTÁNDAR]** Un exception filter global que capture todas las excepciones no manejadas, las loguee, y retorne una respuesta consistente al cliente.
- **[ESTÁNDAR]** Usar `HttpException` y sus derivados (`BadRequestException`, `UnauthorizedException`, etc.) para errores de negocio. No retornar status 500 para errores de validación.
- **[ESTÁNDAR]** Nunca exponer stack traces, mensajes internos, o detalles de implementación en las respuestas de error a producción.

### 5.2 Errores genéricos prohibidos

**[ESTÁNDAR]** No usar `catch(e) { /* vacío */ }` (tragarse errores silenciosamente). Todo error se loguea o se propaga. Si un error se maneja y se decide no propagarlo, se documenta por qué en un comentario.

---

## 6. Configuración y Variables de Entorno

### 6.1 Patrón estándar: `@nestjs/config` + Joi

**[ESTÁNDAR]** Ya documentado en el módulo 06 (Seguridad), Sección 2.3. Resumen: validación de variables de entorno al arranque con Joi schema. La app no arranca si falta una variable requerida.

### 6.2 Jerarquía de configuración

**[RECOMENDADO]**
1. Variables de entorno del sistema (mayor prioridad).
2. `.env` local (solo desarrollo, no versionado).
3. Valores por defecto en el schema de Joi (menor prioridad).

---

## 7. Patrones Prohibidos

**[ESTÁNDAR]** Los siguientes patrones están prohibidos en código nuevo. Su presencia en una auditoría es un hallazgo:

| Patrón | Por qué está prohibido | Alternativa |
|--------|----------------------|-------------|
| `.env` versionado en Git | Exposición de credenciales | `.env` en `.gitignore` + `.env.example` |
| God Objects (clases con >500 líneas o >10 dependencias inyectadas) | Imposibles de testear y mantener | Separar responsabilidades en servicios más pequeños |
| `any` como tipo en TypeScript (salvo excepciones justificadas) | Anula el sistema de tipos | Tipos específicos, generics, `unknown` si es necesario |
| Callbacks anidados (callback hell) | Ilegible, imposible de debuggear | `async/await` |
| Console.log en producción | No estructurado, no filtrable, potencialmente expone datos | Logger estructurado (Winston, Pino, NestJS Logger) |
| Código muerto (funciones, imports, variables sin usar) | Acumula confusión y ruido | Eliminar. ESLint detecta esto automáticamente |
| Secrets hardcodeados | Exposición de credenciales | Ver módulo 06, Sección 2 |
| Config sin validar | La app arranca con config incorrecta y falla en runtime | `@nestjs/config` + Joi |

---

*Módulo 02 del Handbook de Estándares de Ingeniería — Ancient.*
