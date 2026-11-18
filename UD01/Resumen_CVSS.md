# Resumen: Cómo leer un vector CVSS

Resumen detallado de la guía *CVSS_guia_lectura.pdf*, pensada para interpretar las vulnerabilidades que aparecen en informes de Nessus y otras herramientas de seguridad.

---

## 1. ¿Qué es CVSS?

CVSS (Common Vulnerability Scoring System) es el estándar con el que Nessus, y casi cualquier herramienta de seguridad, puntúa la **severidad** de una vulnerabilidad en una escala del **0 al 10**.

La puntuación se representa mediante un **vector**, por ejemplo:

```
CVSS:3.0/AV:N/AC:H/PR:N/UI:R/S:U/C:H/I:H/A:H
```

Este vector no es un código opaco: es una "receta" en la que cada letra es una métrica con un valor concreto. Quien sabe leerlo puede reconstruir por qué la nota es la que es.

### Escala de severidad

| Puntuación | Severidad |
|---|---|
| 9.0 – 10.0 | Crítica |
| 7.0 – 8.9 | Alta |
| 4.0 – 6.9 | Media |
| 0.1 – 3.9 | Baja |
| 0.0 | Informativa / ninguna |

---

## 2. Las dos preguntas de todo vector

La nota final combina dos preguntas. Un impacto devastador no basta para que una vulnerabilidad sea crítica si es muy difícil de explotar.

| Pregunta | Tipo de métricas | Métricas |
|---|---|---|
| ¿Qué tan **fácil** es explotarla? | Explotabilidad | AV, AC, PR, UI, S |
| ¿Qué tan **grave** es el daño? | Impacto (tríada C-I-A) | C, I, A |

- **AV** – Attack Vector
- **AC** – Attack Complexity
- **PR** – Privileges Required
- **UI** – User Interaction
- **S** – Scope
- **C** – Confidentiality
- **I** – Integrity
- **A** – Availability

---

## 3. Métricas de explotabilidad

### 3.1 AV – Attack Vector (¿desde dónde se puede atacar?)

| Sigla | Valor | Significado |
|---|---|---|
| N | Network | Remoto, por red. Es el caso más peligroso: no requiere cercanía ni acceso previo |
| A | Adjacent | Solo desde la misma red local o segmento (misma wifi o VLAN) |
| L | Local | Requiere acceso local al equipo (sesión ya iniciada, terminal) |
| P | Physical | Requiere tocar físicamente el dispositivo |

**Regla general:** cuanto más lejos se pueda atacar sin necesitar cercanía física, más peligrosa es la vulnerabilidad.

### 3.2 AC – Attack Complexity (¿qué tan difícil es?)

| Sigla | Valor | Significado |
|---|---|---|
| L | Low | Se explota de forma fiable, sin condiciones especiales |
| H | High | Requiere condiciones fuera del control del atacante (interceptar tráfico, ganar una carrera de tiempos...) |

### 3.3 PR – Privileges Required (¿qué necesita tener ya el atacante?)

| Sigla | Valor | Significado |
|---|---|---|
| N | None | Ninguna cuenta ni credencial previa |
| L | Low | Necesita privilegios básicos de usuario normal |
| H | High | Necesita privilegios administrativos previos |

### 3.4 UI – User Interaction (¿necesita que alguien más haga algo?)

| Sigla | Valor | Significado |
|---|---|---|
| N | None | El atacante actúa solo, sin depender de nadie |
| R | Required | Necesita que una víctima haga algo (clic, abrir un archivo, tener sesión activa) |

### 3.5 S – Scope (¿hasta dónde llega el daño?)

| Sigla | Valor | Significado |
|---|---|---|
| U | Unchanged | El impacto se queda dentro del propio componente vulnerable |
| C | Changed | El impacto "escapa" a recursos fuera del control de seguridad de ese componente |

---

## 4. Métricas de impacto: C, I, A

Las tres se leen igual; solo cambia sobre qué propiedad de la tríada preguntan.

| Sigla | Valor | Significado |
|---|---|---|
| N | None | Sin impacto en esa propiedad |
| L | Low | Impacto parcial: se compromete algo, no todo |
| H | High | Impacto total: se compromete por completo |

- **C – Confidentiality:** ¿cuánta información puede leer el atacante?
- **I – Integrity:** ¿cuánto puede modificar el atacante?
- **A – Availability:** ¿cuánto puede interrumpir el servicio el atacante?

---

## 5. Métricas temporales (opcionales)

Aparecen en un **segundo vector** y no cambian el problema de fondo: matizan cómo de madura está la amenaza.

| Métrica | Pregunta | Valores posibles |
|---|---|---|
| E | ¿Existe ya una forma fácil de explotarla? | Unproven · PoC · Functional · High |
| RL | ¿Hay ya solución? | Official Fix · Temporary Fix · Workaround · Unavailable |
| RC | ¿Cuánto nos fiamos del informe? | Unknown · Reasonable · Confirmed |

**En Nessus** se ve como un segundo vector junto al base, por ejemplo `CVSS:3.0/E:U/RL:O/RC:C`. No hace falta memorizarlo, solo reconocerlo.

---

## 6. Ejemplo resuelto: Samba Badlock

**CVE-2016-2118**
**Vector:** `AV:N/AC:H/PR:N/UI:R/S:U/C:H/I:H/A:H`
**Nota:** 7.5 (Alta)

| Métrica | Valor | Lectura |
|---|---|---|
| AV | N | Se ataca por red |
| AC | H | Es difícil: necesita interceptar tráfico (MITM) |
| PR | N | No necesita ninguna cuenta previa |
| UI | R | Necesita una sesión cliente-servidor en curso |
| S | U | El daño se queda dentro de Samba |
| C / I / A | H / H / H | Impacto total si se consigue explotar |

**Por qué no es crítica pese a tener C-I-A al máximo:** las dos barreras de entrada (AC:H y UI:R) bajan la nota. CVSS mide el daño **ponderado por lo difícil que es causarlo**, no el daño en bruto.

---

## 7. Método de lectura: las tres preguntas

Para cualquier vulnerabilidad nueva, el orden de lectura recomendado es:

1. **¿Cómo se llega hasta ella?** → AV + AC
2. **¿Qué necesita tener o hacer el atacante de antemano?** → PR + UI
3. **¿Qué se rompe si tiene éxito y hasta dónde llega?** → S + C/I/A

---

## 8. CVSS v2.0 vs v3.x

Si Nessus muestra ambas notas a la vez, no es un error. Diferencias principales de la v2.0:

- No separa PR ni UI: los junta en una sola métrica, **Au** (autenticación).
- No tiene **Scope**.
- Solo tiene los valores **None / Partial / Complete** en el impacto, sin el nivel intermedio "Low".

---

## 9. Idea final

Con este método ya se puede leer cualquier vector CVSS. La guía propone aplicarlo a las vulnerabilidades del propio informe de Nessus.

---

## Chuleta rápida

| Métrica | Valores (de más a menos peligroso, en general) |
|---|---|
| AV | N → A → L → P |
| AC | L → H |
| PR | N → L → H |
| UI | N → R |
| S | C → U |
| C / I / A | H → L → N |
