# 10 · AI en Desarrollo

**Handbook de Estándares de Ingeniería — Ancient**

---

## 1. Herramientas Aprobadas

**[ESTÁNDAR]** La herramienta aprobada para AI-assisted development es **Cursor**.

**[REFERENCIA]** El equipo de AI (Giovanni) desarrolla herramientas internas adicionales con sus propias reglas de uso.

---

## 2. Reglas de Uso

### 2.1 Código generado por AI

**[ESTÁNDAR]** Todo código generado por AI se trata como código escrito por un dev junior: se revisa línea por línea antes de commitear. El hecho de que la AI lo generó no exime al dev de la responsabilidad sobre la calidad y corrección del código.

**En concreto:**
- El dev entiende qué hace el código generado (no es un copy-paste ciego).
- El código sigue los estándares del módulo 02 (naming, estructura, patrones).
- Los tests generados por AI se verifican manualmente (ver módulo 05, anti-patrones de tests fantasma).
- Las dependencias sugeridas por AI se validan contra la lista de paquetes aprobados.

### 2.2 Privacidad y datos

**[ESTÁNDAR]** No se introducen en prompts de AI:
- Datos reales de clientes (PII, datos financieros, datos de salud).
- Credenciales, API keys, tokens, passwords.
- Código propietario del cliente que tenga restricciones contractuales de confidencialidad.
- Información interna sensible de Ancient (financiera, estratégica).

**[RECOMENDADO]** Para debugging con AI, usar datos ficticios o anonimizados. Si se necesita contexto real para reproducir un bug, anonimizar antes de pegar en el prompt.

### 2.3 Branches generadas por AI

**[ESTÁNDAR]** Las branches generadas automáticamente por Cursor (`cursor/*`) se renombran para seguir la convención del módulo 03 (`feature/TICKET-123-descripcion`) antes de crear el PR.

---

## 3. Métricas de Adopción

**[REFERENCIA]** Arquitectura mide el uso de AI como parte de las métricas de ingeniería (ver Gobernanza Técnica, Sección 4). Esto mide:
- Adopción de las herramientas de AI del equipo de Giovanni.
- Patrones de uso (en qué se usa AI: generación de código, tests, refactoring, docs).
- Calidad del output (código generado por AI que no sigue estándares = hallazgo técnico).

El propósito es mejorar la adopción y la calidad, no vigilar a las personas.

---

*Módulo 10 del Handbook de Estándares de Ingeniería — Ancient.*
