# 05 · Testing

**Handbook de Estándares de Ingeniería — Ancient**

Este módulo define el estándar de testing que todo proyecto debe cumplir: existe un piso de cobertura que no se negocia, y los tests son sustantivos, no cosméticos.

---

## 1. Tipos de Tests

### 1.1 Obligatorios

**[ESTÁNDAR]** Todo proyecto tiene como mínimo:

| Tipo | Qué valida | Cuándo es obligatorio |
|------|-----------|----------------------|
| **Unit tests** | Lógica de negocio aislada (services, utils, pipes, guards) | Siempre, en todo proyecto |
| **Integration tests** | Interacción entre módulos (controller → service → DB/API) | Proyectos de complejidad media y alta |

### 1.2 Recomendados

**[RECOMENDADO]** Según el tipo de proyecto:

| Tipo | Qué valida | Cuándo se recomienda |
|------|-----------|---------------------|
| **E2E tests** | Flujos completos desde la perspectiva del usuario | Proyectos con frontend, flujos críticos de negocio |
| **Contract tests** | Contratos de API entre servicios | Arquitecturas de microservicios |
| **Performance tests** | Rendimiento bajo carga | Proyectos con requisitos de rendimiento explícitos |

---

## 2. Cobertura

### 2.1 Umbral mínimo

**[ESTÁNDAR]**

| Contexto | Cobertura mínima | Justificación |
|----------|-----------------|---------------|
| Proyectos nuevos | **80%** | Es alcanzable desde día 1, especialmente con AI-assisted testing |
| Código nuevo | **80%** | Todo código nuevo entra con su cobertura, independientemente del repo |
| Módulos críticos (pagos, auth, datos sensibles) | **90%** | El riesgo de un bug en estos módulos justifica el esfuerzo extra |

### 2.2 Configuración de coverageThreshold

**[ESTÁNDAR]** Todo proyecto define `coverageThreshold` en su configuración de Jest (o equivalente):

```json
// jest.config.ts o package.json
{
  "jest": {
    "coverageThreshold": {
      "global": {
        "branches": 70,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    },
    "collectCoverageFrom": [
      "src/**/*.ts",
      "!src/**/*.spec.ts",
      "!src/**/*.module.ts",
      "!src/**/main.ts",
      "!src/**/*.dto.ts",
      "!src/**/*.entity.ts"
    ]
  }
}
```

### 2.3 Cobertura en CI

**[ESTÁNDAR]** El pipeline de CI:
1. Corre tests con cobertura (`--coverage`).
2. Verifica que la cobertura cumple el umbral (`coverageThreshold` falla el build si no se cumple).
3. Reporta la cobertura a SonarQube.
4. Bloquea el merge a `develop`/`main` si la cobertura está por debajo del umbral.

---

## 3. Frameworks por Stack

**[RECOMENDADO]** Frameworks de testing alineados con el stack de Ancient:

| Stack | Unit/Integration | E2E | Notas |
|-------|-----------------|-----|-------|
| NestJS | **Jest** | Supertest (HTTP) | Nest trae Jest pre-configurado. Usar testing module de Nest para integration tests |
| Vue 3 / MFE | **Vitest** | Cypress o Playwright | Vitest es más rápido que Jest para Vue. Cypress para e2e |
| Angular | **Vitest** | Cypress o Playwright | |
| Java/Spring | **JUnit 5** + Mockito | — | Estándar de Spring |
| Flutter | **flutter_test** | integration_test | Incluido en Flutter SDK |
| Python | **pytest** | — | Con fixtures y parametrize |

---

## 4. Convenciones de Tests

### 4.1 Naming

**[ESTÁNDAR]** Archivos de test: `*.spec.ts` (NestJS/TS), `*.test.ts` (alternativa), `*_test.dart` (Flutter), `test_*.py` (Python).

**[ESTÁNDAR]** Estructura de test (Jest/Vitest):

```typescript
describe('PaymentService', () => {
  describe('processPayment', () => {
    it('should process a valid payment successfully', () => { /* ... */ });
    it('should throw InsufficientFundsError when balance is too low', () => { /* ... */ });
    it('should retry on transient network error', () => { /* ... */ });
  });
});
```

**Regla de naming:** `it('should <acción esperada> when <condición>')`. El nombre del test debe ser suficiente para entender qué se rompió sin leer el código.

### 4.2 Cuándo se escriben

**[ESTÁNDAR]** Los tests se escriben junto con el código, en el mismo PR. No se acepta un PR de feature sin sus tests correspondientes. El pipeline verifica que la cobertura no baje.

**[RECOMENDADO]** Para bugs: escribir primero el test que reproduce el bug (test rojo), luego el fix (test verde). Esto garantiza que el bug no reaparezca.

### 4.3 Uso de AI para testing

**[RECOMENDADO]** Cursor y otras herramientas de AI son especialmente útiles para generar tests. Se recomienda usarlas para:
- Generar el boilerplate de tests (describe/it structure).
- Generar test cases para edge cases que el dev podría no considerar.
- Generar mocks y fixtures.

**[ESTÁNDAR]** Todo test generado por AI se revisa como si fuera código de un dev junior. No se acepta un test generado por AI sin revisión que confirme: (a) que prueba algo real, (b) que las assertions son correctas, (c) que no es un test fantasma.

---

## 5. Anti-Patrones Prohibidos

**[ESTÁNDAR]** Los siguientes tipos de tests están prohibidos porque inflan cobertura sin aportar valor. Su presencia en una auditoría es un hallazgo:

| Anti-patrón | Descripción | Ejemplo |
|-------------|------------|---------|
| **Test fantasma** | Test que existe pero no verifica nada (sin assertions o con assertions triviales) | `it('should work', () => { expect(true).toBe(true); })` |
| **Test que solo verifica existencia** | Confirma que una función existe, no que funcione | `expect(typeof service.process).toBe('function')` |
| **Snapshot sin revisión** | Snapshot test que se auto-aprueba con `--updateSnapshot` sin revisar el cambio | Snapshots actualizados automáticamente en CI |
| **Test acoplado a implementación** | Verifica cómo se implementó algo, no qué produce | `expect(spy).toHaveBeenCalledTimes(3)` sin verificar el resultado |
| **Test que pasa siempre** | Lógica del test hace que siempre pase independientemente del código bajo prueba | Test con mock que retorna exactamente lo que el assertion espera |

---

*Módulo 05 del Handbook de Estándares de Ingeniería — Ancient.*
