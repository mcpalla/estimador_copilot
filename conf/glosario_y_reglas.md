# Glosario y Reglas de Estimación — LANTIK BI (Knowledge Base v1.0)

Documento de soporte para el Agente Estimador LANTIK en Microsoft Copilot Studio.
Fuente: catálogo corporativo de tarifas v1.0 (2026-07-15) y documentación de arquitectura del equipo BI.

---

## 1. Arquitectura de referencia

Todo proyecto BI de LANTIK atraviesa estas capas, en este orden:

```
Sistemas Origen → SSIS (Extracción) → Staging → Normalizado (DataStore/"Tablón")
→ Modelo Dimensional (FAC_/DIM_) → Reporting (Cognos, Power BI, SSRS, Excel)
```

Las fases de estimación se corresponden con las capas:

| Fase | Capa | Contenido |
|---|---|---|
| FN1 | — | Análisis de requisitos y alineación funcional-técnica |
| FN2 | Staging | Extracción de tablas/ficheros de origen (SSIS) |
| FN3 | ETL | DataStore (Normalizado) + Modelo Dimensional, flujos SSIS, maestros, vistas |
| FN4 | Reporting | Cognos (capas física/lógica/presentación) y Power BI |
| FN5 | Transversal | Pruebas, ajustes post-pruebas, seguimiento y coordinación |

**Regla FN5:** siempre el **10% de la base técnica (FN2 + FN3 + FN4)**, redondeado a 1 decimal.
FN1 (análisis) **nunca** entra en esa base.

---

## 2. Niveles de complejidad en ETL (DataStore/DataMart)

Ésta es la decisión más sensible de la estimación. **No se elige por intuición: se elige contando tablas de origen, cruces y reglas de negocio.** Si el usuario no puede contar, se pregunta; si no puede responder, se registra como supuesto.

### Muy Fácil — 40 h (`datastore_datamart_muy_facil`)
- **1–2 tablas de origen**, sin cruces complejos.
- Es esencialmente una **copia con limpieza mínima**: la SELECT de negocio es directa.
- Sin dimensiones lentamente cambiantes (SCD), sin lógica transaccional.
- La tarifa es un **bloque unificado**: incluye tanto el DataStore (Normalizado) como el DataMart (Dimensional). No se estiman por separado.
- Ejemplo real: la actualización normativa del Modelo 170 (reescritura de los scripts de Normalizado + Dimensional sobre estructura existente) se tarificó como 1 unidad Muy Fácil = 40 h.

### Fácil — 60 h (`datastore_datamart_facil`)
- **2–3 tablas de origen**, integración básica.
- JOINs simples y cálculos directos.
- Sin SCD.

### Medio — 80 h (`datastore_datamart_medio`)
- **4–6 tablas de origen**, integración múltiple.
- Cruces con dimensiones y lógica de cálculo de complejidad media.
- **SCD Tipo 1** (sobrescritura, sin histórico de versiones).

### Complicado — 120 h (`datastore_datamart_complicado`)
- **7 o más tablas de origen**, múltiples fuentes.
- Reglas transaccionales, gestión de histórico con **SCD Tipo 2**.
- Validaciones complejas y auditoría reforzada.

### Cómo evitar el error típico
El error más frecuente es asumir "Medio" (80 h) por defecto ante cualquier DataStore.
Preguntas de control antes de asignar nivel:
1. ¿Cuántas tablas de origen alimentan realmente el tablón? (contarlas, no estimarlas)
2. ¿Hay cruces con dimensiones o solo limpieza de campos?
3. ¿Se necesita histórico (SCD Tipo 2) o vale la última foto (Tipo 1 / ninguno)?
4. ¿La estructura ya existe y solo se amplía? (si ya existe, tiende a Muy Fácil/Fácil)

Si las respuestas apuntan a 1–2 tablas sin cruces → **Muy Fácil (40 h)**, aunque el proyecto "suene" grande. Si hay 4–6 tablas con reglas de negocio y SCD Tipo 1 → **Medio (80 h)**. La diferencia entre ambos es del doble de esfuerzo: elegir mal aquí es el mayor riesgo de la estimación.

### Complejidad en flujos SSIS
| Nivel | Horas | Criterio |
|---|---|---|
| Fácil | 4 | Clonar un flujo existente de la misma aplicación con cambios mínimos (parámetros) |
| Medio | 12 | Crear/modificar un flujo con lógica estándar: diseño, componentes, mapeos, validación en Staging |
| Difícil | 24 | Lógica compleja: múltiples transformaciones, expresiones, condicionales; requiere SSIS avanzado |

### Maestros
| Unidad | Horas | Criterio |
|---|---|---|
| Maestro estándar | 4 | Catálogo de referencia simple, sin lógica transaccional |
| Maestro con integridades | 6 | Claves subrogadas, restricciones UNIQUE, auditoría |
| Maestro DS | 20 | Data Store integrado con reglas de negocio: joins, cálculos, control de histórico |

---

## 3. El Generador automático C# (por qué las extracciones estándar son 5 h)

### Qué es
La herramienta central de industrialización del equipo BI de LANTIK: un **generador de código escrito en C#** que fabrica tablas y procedimientos almacenados a partir de un **CSV de metadatos**. Las estimaciones estándar **asumen siempre su uso**; el desarrollo manual de T-SQL estructural es la excepción, nunca la norma.

### Qué produce automáticamente
A partir del CSV, el generador emite un archivo `.sql` completo con:

1. **Capa Normalizado:** el `CREATE TABLE` del DataStore ("Tablón"), índices, descripciones de campos (`MS_Description`, autodocumentación obligatoria) y cumplimiento de la normativa LANTIK (nomenclatura, seguridad, NOT NULL automático).
2. **Esqueleto del procedimiento almacenado:** control de errores, transacciones, logs, marcas de tiempo y lógica de carga incremental.
3. **SP de fusión incremental de Staging:** el `MERGE` que consolida las tablas temporales de extracción sobre la tabla definitiva de Staging.
4. **Capa Dimensional completa:** la transformación del Tablón al modelo en estrella (hechos `FAC_` y dimensiones `DIM_`). Este código casi nunca se toca a mano.
5. **Actualización de puntos de control de auditoría** (marcas de tiempo de ejecución).

### Qué queda para el humano
El generador cubre **~80% del trabajo repetitivo**. El consultor solo escribe **la sentencia SELECT de carga del Normalizado** (JOINs, limpieza, reglas de negocio) y la pega en el esqueleto generado, además de diseñar y rellenar el CSV de metadatos.

### Impacto en las tarifas
- **Extracción de tabla estándar = 5 h**: el esfuerzo real se limita a registrar la tabla en el CSV de metadatos, regenerar, y validar la carga. La creación de tablas, índices, MERGE de Staging, logs y auditoría la produce el generador sin esfuerzo manual. Sin el generador, esa misma extracción exigiría programar a mano toda la estructura y el procedimiento (varias veces ese esfuerzo).
- Por la misma razón, en las unidades de DataStore/DataMart el esfuerzo estimado se concentra en **diseñar el CSV + escribir la SELECT de negocio**, no en programar T-SQL estructural.

### Regla de disciplina (afecta a riesgos)
**Nunca se modifica a mano** (`ALTER TABLE`, edición directa del SP) una estructura gestionada por el generador: el cambio se hace en el CSV y se regenera. Si un requisito obliga a lógica **fuera del estándar del generador**, eso justifica desarrollo manual y debe registrarse en la estimación como **riesgo/excepción explícita** (nunca absorberse en silencio en la tarifa estándar).

---

## 4. Otras unidades de referencia

### Extracción (FN2)
| Unidad | Horas | Cuándo aplica |
|---|---|---|
| Tabla estándar | 5 | Tabla SQL Server sin reglas complejas de conexión (vía generador) |
| Fichero estándar | 4 | Fichero plano (CSV/Excel) con estructura estable |
| Extracción complicada | 21 | Múltiples orígenes, transformaciones previas, validaciones |
| Incremental CDC | 7 | Change Data Capture; preferible a full load con volúmenes grandes |

### Cognos (FN4)
| Unidad | Horas | Cuándo aplica |
|---|---|---|
| Capa física | 4 | Actualizar relaciones/entidades/claves en Framework Manager (por tabla) |
| Campo publicado | 0.25 | Exponer un campo en capas lógica/presentación (renombrado, formato, jerarquía) |
| Vista indexada Cognos | 5 | Vista indizada para optimizar consultas de reporting/agregaciones |

Ejemplo: publicar 140 campos nuevos = 140 × 0.25 = 35 h; con 1 capa física (4 h), FN4 = 39 h.

### Power BI (FN4)
| Unidad | Horas | Cuándo aplica |
|---|---|---|
| Menú/navegación | 8 | Botones y slicers de navegación |
| Pestaña simple | 24 | 1–2 visuales básicos, sin interactividad compleja |
| Pestaña estándar | 40 | 3–5 visuales, filtros, drill-down, validación y rendimiento |
| Pestaña compleja | 64 | 6+ visuales, DAX complejo, interactividad avanzada |

### Vistas para informes (FN3)
| Unidad | Horas | Cuándo aplica |
|---|---|---|
| Vista indexada SQL | 10 | Optimización de queries de reporting: índices, testing de rendimiento, documentación |

---

## 5. Roles y reparto de horas

| Rol | Significado |
|---|---|
| JP | Jefatura de Proyecto (gestión, coordinación, seguimiento) |
| AN | Análisis de Negocio (requisitos, validación funcional) |
| DT | Diseño Técnico (arquitectura, SQL, procedimientos) |
| PR | Programación (SSIS, desarrollo, testing técnico) |

Reparto por defecto:
- **Fases técnicas (FN2–FN4) y transversal (FN5):** JP 10% · AN 30% · DT 35% · PR 25%
- **Fase de análisis (FN1):** JP 10% · AN 50% · DT 30% · PR 10%

Las horas por rol se redondean a 1 decimal.

---

## 6. Principios de estimación (recordatorio)

1. Un proyecto es una iniciativa de negocio ("Nuevo DataMart de TPVs"), no una tarea técnica ("crear un SP").
2. Nunca se estima sin diagnóstico previo: objetivo → patrón → capas → tareas → dependencias → complejidad → esfuerzo.
3. Toda tarifa sale del catálogo corporativo; lo que no está en el catálogo se declara como conocimiento faltante.
4. Toda duda sin resolver se convierte en supuesto con impacto explícito ("si resulta falso: +X h") o bloquea la estimación.
5. Nunca se entrega un número seco: siempre acompañado de supuestos, exclusiones y condiciones de validez.
