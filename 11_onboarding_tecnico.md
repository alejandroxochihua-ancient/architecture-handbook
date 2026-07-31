# 11 · Onboarding Técnico

**Handbook de Estándares de Ingeniería — Ancient**

---

## 1. Propósito

Este módulo define el proceso que sigue un dev nuevo cuando ingresa a Ancient, antes de ser asignado a un pod de proyecto. El objetivo es que llegue a su primer sprint con el conocimiento base necesario para contribuir sin generar fricción por desconocimiento de las reglas del juego.

---

## 2. Responsables

| Etapa | Responsable |
|-------|-------------|
| Onboarding general (cultura, herramientas, accesos) | Talent / Operación |
| Onboarding técnico (este módulo) | TL asignado, con supervisión de Santiago (Lead TLs) |
| Validación de seniority pre-contratación | Arquitectura (entrevista técnica) |

---

## 3. Proceso

### Día 1-2: Setup

- [ ] Acceso a GitHub/GitLab (organización `ancient-global`).
- [ ] Acceso al proveedor de CI/CD del proyecto asignado.
- [ ] Acceso a SonarCloud.
- [ ] Configuración del entorno local: clonar repo, instalar dependencias, correr en local.
- [ ] Configurar IDE (Cursor/VSCode) con las extensiones recomendadas: ESLint, Prettier, GitLens.
- [ ] Verificar que `.nvmrc` funciona y está usando la versión correcta de Node.

### Día 2-3: Lectura del Handbook

- [ ] Leer módulos 01 (Principios Generales), 02 (Convenciones), 03 (Gestión de Cambios), 04 (Code Review), 05 (Testing), 06 (Seguridad) de este Handbook.
- [ ] Revisar la configuración de ESLint y Prettier del proyecto asignado.
- [ ] Hacer un commit de práctica siguiendo Conventional Commits (en una rama `feature/*` de práctica, nunca directo a `main` ni `develop`).
- [ ] Abrir un PR de práctica para entender el flujo de revisión del pod.

### Día 3-5: Contexto del Proyecto

- [ ] Leer el README del proyecto asignado.
- [ ] Explorar la estructura de carpetas y entender los módulos principales.
- [ ] Revisar la documentación de API (Swagger) del proyecto.
- [ ] Entender el pipeline de CI/CD: qué pasos corre, qué bloquea, qué informa.
- [ ] Identificar las integraciones principales del proyecto (APIs externas, bases de datos, servicios).

### Día 5: Primer Task

- [ ] El TL asigna un task pequeño y bien definido (bug fix simple, mejora menor, agregar test).
- [ ] El dev lo resuelve siguiendo todo el flujo: branch → código + tests → PR → review → merge.
- [ ] El TL revisa el PR como revisaría cualquier otro, señalando desviaciones de estándares si las hay.

---

## 4. Checklist de Onboarding

**[ESTÁNDAR]** El TL firma este checklist al final de la primera semana del dev. Si no puede firmar algún punto, se atiende antes de la segunda semana.

```
CHECKLIST DE ONBOARDING TÉCNICO — [Nombre del Dev]
═══════════════════════════════════════════════════
Fecha de ingreso:    [DD/MM/AAAA]
TL responsable:      [Nombre]
Proyecto asignado:   [Nombre]

SETUP
☐ Tiene acceso a todos los repos y herramientas necesarias
☐ Puede correr el proyecto localmente
☐ IDE configurado con ESLint + Prettier

HANDBOOK
☐ Leyó y entiende los módulos 01-06 del Handbook
☐ Conoce las convenciones de Git (Conventional Commits, branching)
☐ Conoce el flujo de PRs (quién revisa, qué se revisa, SLA)
☐ Conoce las reglas de testing (cobertura mínima, anti-patrones)
☐ Conoce las reglas de seguridad (secrets, .env, datos sensibles)

PROYECTO
☐ Entiende la estructura y módulos principales del proyecto
☐ Puede navegar la documentación de API
☐ Entiende el pipeline de CI/CD del proyecto
☐ Conoce al TL, PM, y otros miembros del pod

PRIMER TASK
☐ Completó un task usando el flujo completo (branch → PR → merge)
☐ El PR fue revisado y el feedback fue atendido

TL: ____________________  Fecha: ___________
Dev: ____________________  Fecha: ___________
```

---

## 5. Onboarding de TLs

Los TLs tienen un proceso adicional porque son responsables de la calidad técnica del pod:

- [ ] Leer **todo** el Handbook (módulos 01-11).
- [ ] Leer el documento de **Gobernanza Técnica** del Área de Arquitectura.
- [ ] Leer el documento de **Requerimientos ARQ → OPS**.
- [ ] Entender el proceso de escalación (qué puede resolver solo, qué escala).
- [ ] Entender el proceso de auditoría (qué se audita, cuándo, cómo se reporta).
- [ ] Entender el mecanismo de reporte por sprint (qué métricas, en qué formato, con qué cadencia).
- [ ] Sesión 1:1 con Santiago (Lead TLs) para contexto operativo.
- [ ] Si es asignado a un proyecto con arquitectura compleja: sesión de capacitación con Arquitectura (ver Checklist de Capacitación en el doc de Requerimientos ARQ→OPS).

---

*Módulo 11 del Handbook de Estándares de Ingeniería — Ancient.*
