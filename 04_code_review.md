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
| Cambios en módulos críticos (pagos, auth, manejo de datos sensibles) | TL obligatorio, además del peer | 2 |
| Cambios en pipeline, IaC o config de ambientes | TL + Infra/Cibersec | 2 |

**[ESTÁNDAR]** Nadie externo al pod revisa PRs del pod, salvo que Arquitectura lo solicite explícitamente como parte de una auditoría o escalación.

**[ESTÁNDAR]** Nadie aprueba su propio PR, ni siquiera el TL. Si el TL es el autor, revisa otro TL o el Lead TLs.

**[RECOMENDADO]** Usar un archivo `CODEOWNERS` en el repo para que los reviewers de las dos últimas filas se asignen solos, sin depender de que el autor se acuerde de etiquetarlos.

---

## 2. SLA de Revisión

**[ESTÁNDAR]** Todo PR se revisa en un plazo máximo de **24 horas hábiles** desde que se solicita la revisión. Si el reviewer no puede revisarlo en ese plazo, lo señala y se reasigna.

**[ESTÁNDAR]** Para `hotfix/*` el SLA es de **2 horas hábiles**. Si el TL no está disponible, escala al Lead TLs. Un hotfix no espera 24 horas.

**[ESTÁNDAR]** El SLA aplica igual a la re-revisión: una vez que el autor atiende los comentarios, el reviewer tiene el mismo plazo para volver a revisar.

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
- [ ] ¿Si el endpoint recibe un ID de recurso, se valida que el recurso pertenezca al usuario del token?

**Arquitectura:**
- [ ] ¿El cambio es consistente con la arquitectura del paquete de pase?
- [ ] ¿Si se agrega una dependencia nueva, está justificada?
- [ ] ¿No se introduce deuda técnica innecesaria?

**Datos y compatibilidad:**
- [ ] ¿Si hay migración de BD, es retrocompatible con la versión anterior de la app?
- [ ] ¿Si cambia un contrato de API, se versionó o se mantiene retrocompatibilidad para los consumidores?
- [ ] ¿Si se agregó una variable de entorno, está en `.env.example` y en el schema de Joi?

**[ESTÁNDAR]** Los puntos de Seguridad y los de Datos y compatibilidad son bloqueantes: si alguno falla, el PR no se integra hasta resolverlo. Los demás pueden salir como comentario de mejora a criterio del reviewer.

### 3.1 Cómo se comentan los hallazgos

**[RECOMENDADO]** Todo comentario de revisión indica si bloquea o no, con un prefijo:

| Prefijo | Significado |
|---------|-------------|
| `[bloqueante]` | Debe resolverse antes del merge |
| `[sugerencia]` | Mejora opcional, el autor decide |
| `[duda]` | Pregunta del reviewer, no implica cambio |
| `[nit]` | Detalle menor (estilo, naming), no bloquea |

**[ESTÁNDAR]** Los comentarios se hacen sobre el código, no sobre la persona. Se comenta "esta query no tiene índice y va a escanear la tabla completa", no "no sabes hacer queries".

---

## 4. Checks Automáticos

**[ESTÁNDAR]** Además de la revisión humana, los siguientes checks deben pasar antes de que un PR se pueda integrar. Los que bloquean impiden el merge si fallan:

| Check | Herramienta | Bloqueante |
|-------|-------------|------------|
| Lint | ESLint | ✅ Errores de lint |
| Tests | Jest/Vitest | ✅ Tests fallidos |
| Cobertura | Jest + SonarCloud | ✅ Por debajo del umbral |
| Seguridad (SAST) | SonarCloud | ✅ Vulnerabilidades críticas y altas |
| Seguridad (SCA) | Snyk | ✅ Vulnerabilidades críticas y altas |
| Secret scanning | Gitleaks o el del proveedor | ✅ Cualquier hallazgo |
| Título del PR (Conventional Commits) | Action de commitlint | ✅ Formato inválido |
| Build | Framework build | ✅ Build fallido |

**[ESTÁNDAR]** Ningún check bloqueante se salta con override manual. Si un check falla y el equipo considera que es un falso positivo, se documenta en el PR y lo autoriza el TL. Si es recurrente, se corrige la regla, no se ignora el check.

---

## 5. AI-Assisted Code Review

**[RECOMENDADO]** Usar herramientas de AI para asistir (no reemplazar) la revisión de código.

**Reglas:**
- La revisión de AI es complementaria. No sustituye la revisión humana del TL o peer.
- La AI puede detectar patrones, bugs potenciales, y sugerir mejoras. El reviewer humano decide qué aceptar.
- La configuración de la herramienta de AI review se estandariza (no cada dev con su propia config).
- **[ESTÁNDAR]** La aprobación de la AI no cuenta como la aprobación requerida en la Sección 1. El mínimo de aprobaciones siempre son humanas.
- **[ESTÁNDAR]** No se conectan herramientas de AI review a repos con restricción contractual de confidencialidad del cliente sin visto bueno previo. Estas herramientas leen el repositorio completo, no solo el diff, y eso es un supuesto distinto al del módulo 06 Sección 8.3, que regula lo que el dev pega en un prompt.

---

*Módulo 04 del Handbook de Estándares de Ingeniería — Ancient.*
