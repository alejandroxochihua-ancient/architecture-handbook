# 08 · Calidad — SonarQube

**Handbook de Estándares de Ingeniería — Ancient**

---

## 1. Plataforma

**[ESTÁNDAR]** Ancient usa **SonarCloud** como plataforma única de análisis de calidad, bajo la organización `ancient-global`. No requiere infraestructura propia y se integra de forma nativa con GitHub/GitLab.

---

## 2. Quality Gate Estándar

**[ESTÁNDAR]** El quality gate de Ancient para SonarCloud bloquea el merge si no se cumple:

| Métrica | Condición | Bloqueante |
|---------|-----------|------------|
| Bugs nuevos | 0 | ✅ |
| Vulnerabilidades nuevas | 0 | ✅ |
| Security Hotspots revisados | 100% | ⚠️ Informativo |
| Cobertura en código nuevo | ≥ 80% | ✅ |
| Duplicación en código nuevo | ≤ 3% | ⚠️ Informativo |
| Maintainability Rating | A | ⚠️ Informativo |
| Reliability Rating | A | ✅ |
| Security Rating | A | ✅ |

Si el quality gate falla, el PR no se puede integrar.

---

## 3. Configuración por Proyecto

**[ESTÁNDAR]** Todo proyecto configurado en SonarCloud incluye `sonar-project.properties` en la raíz del repo:

```properties
sonar.organization=ancient-global
sonar.projectKey=ancient-global_[nombre-del-repo]

# Fuentes
sonar.sources=src
sonar.tests=src
sonar.test.inclusions=**/*.spec.ts,**/*.test.ts

# Exclusiones
sonar.exclusions=node_modules/**,dist/**,coverage/**,**/*.dto.ts,**/*.entity.ts,**/*.module.ts

# Cobertura
sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.typescript.lcov.reportPaths=coverage/lcov.info
```

---

## 4. Reglas Desactivadas

**[ESTÁNDAR]** Si una regla de Sonar se desactiva para un proyecto, se documenta la razón en el `sonar-project.properties` o en un archivo `SONAR_EXCEPTIONS.md` en el repo. Reglas desactivadas sin justificación son un hallazgo de auditoría.

---

*Módulo 08 del Handbook de Estándares de Ingeniería — Ancient.*
