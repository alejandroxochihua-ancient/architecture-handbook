# 04 · Revisión de Código (Pull Requests)

**Handbook de Estándares de Ingeniería — Ancient**

---

## 1. Quién Revisa

**[ESTÁNDAR]**

| Integración | Reviewer requerido | Mínimo de aprobaciones |
|-------------|-------------------|----------------------|
| `feature/*` o `fix/*` → `develop` | Otro dev del pod (peer review) **o** TL | 1 |
| `release/*` → `main` (promoción a producción) | TL del proyecto | 1 (TL obligatorio) |
| `hotfix/*` → `main` | TL del proyecto | 1 (fast-track: review inmediato) |

**[ESTÁNDAR]** Nadie externo al pod revisa PRs del pod, salvo que Arquitectura lo solicite explícitamente como parte de una auditoría o escalación.

---

## 2. SLA de Revisión

**[ESTÁNDAR]** Todo PR se revisa en un plazo máximo de **24 horas hábiles** desde que se solicita la revisión. Si el reviewer no puede revisarlo en ese plazo, lo señala y se reasigna.

¿Por qué importa? Los PRs que esperan 3-5 días generan: context-switching cuando el autor tiene que volver al código después de días, conflictos de merge que crecen con el tiempo, y cuello de botella que frena al equipo entero.

---

## 3. Checklist de Revisión

**[ESTÁNDAR]** Todo reviewer verifica al menos los siguientes puntos:

**Funcionalidad:**
- [ ] ¿El código hace lo que el PR dice que hace?
- [ ] ¿Se manejan los edge cases razonables?
- [ ] ¿Los errores se manejan correctamente (no se tragan, no exponen info interna)?

**Calidad:**
- [ ] ¿Hay tests nuevos o actualizados que cubran el cambio?
- [ ] ¿El código sigue las convenciones del módulo 02 (naming, estructura, patrones)?
- [ ] ¿No hay código muerto, imports sin usar, o TODOs sin ticket asociado?

**Seguridad:**
- [ ] ¿No se introducen secrets hardcodeados?
- [ ] ¿Los inputs del usuario se validan?
- [ ] ¿Los datos sensibles no se loguean?

**Arquitectura:**
- [ ] ¿El cambio es consistente con la arquitectura del paquete de pase?
- [ ] ¿Si se agrega una dependencia nueva, está justificada?
- [ ] ¿No se introduce deuda técnica innecesaria?

---

## 4. Checks Automáticos

**[ESTÁNDAR]** Además de la revisión humana, los siguientes checks deben pasar antes de que un PR se pueda integrar. Los que bloquean impiden el merge si fallan:

| Check | Herramienta | Bloqueante |
|-------|-------------|------------|
| Lint | ESLint | ✅ Errores de lint |
| Tests | Jest/Vitest | ✅ Tests fallidos |
| Cobertura | Jest + Sonar | ✅ Por debajo del umbral |
| Seguridad (SAST) | SonarQube | ✅ Vulnerabilidades críticas |
| Seguridad (SCA) | Snyk | ✅ Vulnerabilidades críticas |
| Build | Framework build | ✅ Build fallido |

---

## 5. AI-Assisted Code Review

**[RECOMENDADO]** Usar herramientas de AI para asistir (no reemplazar) la revisión de código.

**Reglas:**
- La revisión de AI es complementaria. No sustituye la revisión humana del TL o peer.
- La AI puede detectar patrones, bugs potenciales, y sugerir mejoras. El reviewer humano decide qué aceptar.
- La configuración de la herramienta de AI review se estandariza (no cada dev con su propia config).

---

*Módulo 04 del Handbook de Estándares de Ingeniería — Ancient.*
