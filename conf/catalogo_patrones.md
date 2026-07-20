# Catálogo de Patrones de Proyecto — LANTIK BI (Knowledge Base v1.0)

Documento de clasificación para el Agente Estimador LANTIK. Consolidado desde la Pattern Library corporativa (PAT-001 a PAT-011).

**Cómo usar este catálogo:** toda petición entrante debe clasificarse primero contra estos patrones ("Classify First"), indicando también qué patrones se descartan y por qué. Cada patrón aporta las preguntas de cualificación y las sospechas del consultor para el diagnóstico.

**Regla de autoridad:** los "órdenes de magnitud" de este catálogo sirven SOLO como comprobación de coherencia del resultado final. **Las horas de una estimación salen siempre del catálogo de tarifas (`tarifas_lantik.json`), nunca de este documento.** Si el total calculado queda muy fuera del orden de magnitud del patrón, revisar el diagnóstico antes de sentenciar.

**Si ningún patrón encaja:** no forzar la petición en una categoría inadecuada. Declararlo, proponer la creación de un patrón nuevo y bloquear o estimar solo con supuestos explícitos.

---

## Guía rápida de clasificación

| Señal en la petición | Patrón |
|---|---|
| Origen/aplicación nueva que no existe en el DW | PAT-001 |
| Añadir campos o cambiar tipos en una fuente ya integrada | PAT-002 |
| Nuevo análisis: crear/ampliar tablas Fact y Dim (modelo estrella) | PAT-003 |
| Nuevo cuadro de mando o informe (los datos ya existen) | PAT-004 |
| Nueva métrica, ratio o KPI sobre datos existentes | PAT-005 |
| Algo funciona pero va lento (carga nocturna o informe) | PAT-006 |
| Algo falla o descuadra (job caído, dato erróneo) | PAT-007 |
| Cambio de plataforma, versión o herramienta | PAT-008 |
| Catálogo de referencia / tabla maestra / equivalencias | PAT-009 |
| Módulo funcional completo nuevo, de origen a informe | PAT-010 |
| Nueva versión normativa de un modelo que ya existe (Hacienda) | PAT-011 |

---

## PAT-001 — Integración de una nueva fuente de datos

**Qué es:** incorporar un origen que hoy no se explota: desde la conectividad hasta el Normalizado y la automatización del flujo nocturno.

**Cuándo SÍ:** aplicación nueva o sistema externo (tabla Microfocus, BD SQL externa, volcado CSV/Excel, API); hay que crear la estructura de Staging desde cero.
**Cuándo NO:** los datos ya se extraen y solo se añaden columnas (→ PAT-002); análisis puntual de una sola vez (no es proyecto BI corporativo).

**Preguntas clave:** tecnología del origen; forma de conexión física (driver, linked server, ruta de red, credenciales); volumen e historial; ¿hay PK o campo de fecha que permita carga incremental?; ¿restricciones LOPD/enmascaramiento?; ¿arquitectura de proyecto grande (3 BDs) o pequeño (1 BD con esquemas)?

**Sospechas del consultor:** calidad de datos deficiente en el origen (duplicados, nulos, fechas rotas); el "Excel/CSV estable" cambiará de estructura a los tres meses; riesgo de bloquear el productivo del cliente al extraer; el desarrollo debe pasar por el CSV del Generador, no picarse a mano.

**Capas/fases:** SSIS → Staging → Normalizado (FN2 y FN3).
**Unidades de tarifa habituales:** `tabla_estandar` / `fichero_estandar` / `incremental_cdc` / `extraccion_complicada` (FN2); flujos SSIS y DataStore/DataMart según complejidad (FN3).
**Orden de magnitud (solo contraste):** 60–120 h; extremo bajo si la aplicación ya tiene proyecto SSIS parametrizado que duplicar.
**Relacionados:** PAT-002 (evoluciones futuras), PAT-010 (donde actúa como eslabón inicial).

---

## PAT-002 — Evolución de una fuente ya integrada

**Qué es:** modificar o ampliar una estructura ya operativa en el DW: añadir columnas de un origen ya conectado, replicar cambios de tipo de datos, corregir la lógica de un SP existente.

**Cuándo SÍ:** añadir columnas a una tabla que ya se extrae; cambio de tipo en el origen que hay que replicar; modificar lógica de cálculo de un campo.
**Cuándo NO:** tabla nueva completa (→ PAT-001); cambio solo visual de informe (→ PAT-004); cambio de versión normativa estructurado con múltiples tablas nuevas (→ PAT-011).

**Preguntas clave:** nombres y tipos exactos de los campos nuevos; ¿afecta a la PK?; ¿el histórico ya almacenado debe rellenarse o queda NULL?; ¿el campo llega a reporting o es solo técnico?

**Sospechas del consultor:** "es solo añadir una columna" nunca lo es — el mapeo SSIS se desconfigura y hay que remapear y recompilar; no se hace ALTER TABLE a mano: se actualiza el CSV del Generador y se regenera; si cambia un tipo de datos, algún informe de Cognos/Power BI romperá casi seguro.

**Capas/fases:** SSIS → Staging → Normalizado (FN2/FN3), y FN4 si el dato se expone al usuario.
**Unidades habituales:** flujo SSIS según nivel (FN3), `maestro_*` si aplica, `campo_cognos` y `capa_fisica_cognos` (FN4).
**Orden de magnitud (solo contraste):** 12–40 h (cerca de 12 si el campo se queda en Normalizado; se acumulan PAT-003/PAT-004 si viaja hasta el informe).
**Relacionados:** PAT-001 (antecesor), PAT-003 (si el campo se explota analíticamente).

---

## PAT-003 — Creación o evolución de un DataMart / Modelo Dimensional

**Qué es:** diseñar, construir o ampliar un modelo en estrella (tablas de Hechos `FAC_` y Dimensiones `DIM_`) a partir de la capa Normalizada.

**Cuándo SÍ:** nueva área de análisis que requiere Fact+Dim nuevas; añadir una dimensión de análisis a un hecho existente; añadir atributos a una dimensión.
**Cuándo NO:** los datos aún no están en el Normalizado (→ PAT-001/PAT-002 primero); solo se pide un informe sobre datos que ya existen en la estrella (→ PAT-004).

**Preguntas clave:** granularidad exacta de la Fact (¿factura, línea, acumulado diario?); métricas a sumarizar; ejes de análisis (Dim); comportamiento ante cambios de atributos (SCD: ¿el histórico se muda o se conserva donde ocurrió?); ¿alimenta Power BI directo o requiere cubo SSAS? (los cubos solo existen en áreas concretas, no en Hacienda); ID de proceso de negocio (arrastra la nomenclatura `Datastore_NNN` → `FACT_NNN`).

**Sospechas del consultor:** el usuario mezclará granularidades ("presupuesto anual" y "ticket diario" en la misma tabla) — habrá que separar hechos; sin claves subrogadas enteras el rendimiento colapsa; varias fechas de análisis no son varias tablas de tiempo, son role-playing sobre la DimTiempo corporativa.

**Capas/fases:** Normalizado → Dimensional (FN3). El código de carga lo produce el Generador; solo la lógica fuera de estándar justifica T-SQL manual (registrar como excepción).
**Unidades habituales:** `datastore_datamart_*` según criterios (nº tablas, cruces, SCD), `maestro_ds`, `vista_para_informes` (FN3).
**Orden de magnitud (solo contraste):** ampliación de dimensión 16–40 h; DataMart nuevo completo 40–80 h.
**Relacionados:** PAT-002 (proveedor), PAT-004 (consumidor), PAT-005 (KPIs que viven aquí).

---

## PAT-004 — Nuevo dashboard o informe

**Qué es:** capa de presentación: cuadros de mando, reportes operativos o informes sobre datos que ya existen en el Dimensional.

**Cuándo SÍ:** nuevo cuadro de mando, reporte o informe; migrar un informe a otra herramienta con la misma lógica.
**Cuándo NO:** requiere cálculos que el modelo no soporta (→ PAT-005 primero); los datos no existen en ninguna tabla dimensional (→ PAT-003 primero).

**Preguntas clave:** herramienta de destino — la elección no es libre: **Cognos** = análisis libre/autoservicio; **Power BI** = cuadros de mando cerrados y maquetados; **SSRS** = solo mantenimiento de lo heredado (un desarrollo nuevo en SSRS es excepción a justificar y registrar como riesgo). Audiencia y concurrencia; ¿el modelo dimensional tiene el 100% de los datos?; ¿seguridad por filas (RLS)?; frecuencia de actualización.

**Sospechas del consultor:** ~40% de las veces falta alguna dimensión aunque el usuario crea que está todo; la seguridad por perfiles aparecerá al final aunque no se pida al inicio; pedirán comparativas temporales (mes vs mismo mes año anterior) — validar la DimTiempo; siempre faltan en la petición las horas de manuales y formación.

**Capas/fases:** Reporting (FN4). Las herramientas atacan exclusivamente la capa Dimensional.
**Unidades habituales:** `pbi_menu` / `pbi_pestana_simple` / `pbi_pestana_estandar` / `pbi_pestana_compleja`; en Cognos `capa_fisica_cognos` + `campo_cognos` + `vista_indexada_cognos` (FN4).
**Orden de magnitud (solo contraste):** 24–60 h solo visualización.
**Relacionados:** PAT-003, PAT-005.

---

## PAT-005 — Nuevo KPI o indicador de negocio

**Qué es:** definir e implementar una métrica o ratio garantizando la "versión única de la verdad": la regla de cálculo se centraliza en BD o modelo semántico, nunca en un informe suelto.

**Cuándo SÍ:** nueva fórmula sobre datos existentes; cambio en la definición de un indicador.
**Cuándo NO:** los datos base no entran aún al DW (→ PAT-001); requerimiento puramente visual (→ PAT-004).

**Preguntas clave:** fórmula exacta y componentes; comportamiento ante división por cero y nulos; ¿es aditivo en el tiempo o es un saldo (non-additive)?; ¿contra qué reporte oficial se cuadra?

**Sospechas del consultor:** el cálculo debe bajar al modelo o a la BD — si se escribe en el informe, otro usuario lo recalculará distinto y descuadrará; "Ventas Netas" es ambiguo hasta definir qué se resta (devoluciones, descuentos, impuestos); los ratios deben sumar primero y dividir después o se rompen al agrupar.

**Capas/fases:** Normalizado/Fact si es precalculado (vía Generador: marca `F` en el CSV), capa semántica (DAX / Framework Manager) si es dinámico. FN3 o FN4 según dónde viva.
**Unidades habituales:** normalmente dentro de las tarifas DS/DM o de pestaña Power BI; si excede el estándar, declarar juicio experto justificado.
**Orden de magnitud (solo contraste):** 8–24 h.
**Relacionados:** PAT-003, PAT-004.

---

## PAT-006 — Optimización técnica

**Qué es:** mejorar rendimiento de procesos o informes que funcionan pero van lentos (job nocturno que se sale de la ventana, informe que tarda >5–10 s).

**Cuándo SÍ:** retraso en carga nocturna; informe lento al filtrar; consumo excesivo de CPU/memoria.
**Cuándo NO:** se piden campos/KPIs/reglas nuevas (→ PAT-002/003/005); el proceso falla con error o dato roto (→ PAT-007).

**Preguntas clave:** componente exacto que va lento; cuánto tardaba, cuánto tarda y objetivo aceptable; ¿es constante o solo en picos (cierres de mes)?; ¿se ha analizado el plan de ejecución?; ¿hay concurrencia lectura/escritura?

**Sospechas del consultor:** el 50% se resuelve con mantenimiento de índices y estadísticas; conversiones implícitas (VARCHAR vs NVARCHAR en JOINs) invalidan índices; cursores/bucles fila a fila donde debería haber operaciones de conjunto; lógicas manuales añadidas sobre tablas del Generador sin sus índices.

**Capas/fases:** SQL Server principalmente (FN3); SSIS o reporting según el cuello de botella.
**Unidades habituales:** no hay tarifa volumétrica directa — normalmente juicio experto justificado (horas fijas) + `vista_para_informes` / `vista_indexada_cognos` si la solución es de indexación.
**Orden de magnitud (solo contraste):** 16–80 h (un SP concreto 16–32 h; un pipeline completo hasta 80 h).
**Relacionados:** PAT-007 (si la lentitud provoca caídas).

---

## PAT-007 — Corrección funcional (incidencia)

**Qué es:** aislar y corregir un error operativo: job caído, paquete SSIS roto, descuadre numérico en un informe.

**Cuándo SÍ:** job nocturno detenido con error; SSIS falla por formato o fichero ausente; descuadre reportado por el usuario.
**Cuándo NO:** va lento pero termina bien (→ PAT-006); se pide funcionalidad nueva (→ PAT-002/003/005).

**Preguntas clave:** código de error / mensaje / SP registrado en logs (`audit`); ¿bloquea otros informes por precondiciones?; ejemplo concreto del dato erróneo (esperado vs real); ¿sistemático o caso puntual?; ¿hubo despliegue reciente en el origen?

**Sospechas del consultor:** métricas duplicadas de repente = registro duplicado en un maestro multiplicando el JOIN; KPI vacío o proceso caído = NULL en aritmética; orígenes antiguos o Excel = fechas imposibles; descuadre intermitente = alguien ejecutó a mano saltándose precondiciones. Nota operativa: las alertas por correo de fallos de carga no llegan (direcciones antiguas) — la detección depende de monitorizar la tabla de logs.

**Capas/fases:** cualquiera; el diagnóstico va hacia atrás desde el síntoma (FN según capa raíz).
**Unidades habituales:** juicio experto (horas fijas justificadas); tipo_trabajo = incidencia.
**Orden de magnitud (solo contraste):** 4–16 h (bug estándar 4–8 h; con reproceso de histórico hasta 16 h).
**Relacionados:** PAT-002 (si la corrección exige cambio estructural), PAT-006.

---

## PAT-008 — Migración tecnológica

**Qué es:** trasladar componentes del DW de una plataforma/versión/herramienta a otra replicando exactamente la lógica previa (ej. Cognos → Power BI, SQL Server 2016 → 2022, cubos OLAP → tabular).

**Cuándo SÍ:** cambio de herramienta de visualización corporativa; actualización del motor de BD; sustitución de componente de arquitectura.
**Cuándo NO:** actualizar un informe dentro de la misma herramienta (→ PAT-004); cambio de lógica por reglas de negocio (→ PAT-002/003/005).

**Preguntas clave:** tecnología/versión origen y destino exactas; inventario real de objetos (paquetes, SPs, tablas, informes activos); ¿hay funciones obsoletas no soportadas en destino?; ¿estrategia big-bang o paralelo?; ¿existe batería de pruebas o informe "patrón de oro" para certificar?

**Sospechas del consultor:** del inventario declarado, ~30% ya no se usa y ~20% son duplicados — auditar uso antes de estimar; la traducción literal Cognos→DAX no funciona (lógica multidimensional vs tabular); los cambios de versión de SQL alteran planes de ejecución y comportamiento ante nulos.

**Capas/fases:** variable según alcance.
**Unidades habituales:** mayormente juicio experto + inventario × unidades correspondientes; requiere fase de auditoría previa.
**Orden de magnitud (solo contraste):** migración de versión SQL 40–120 h de regresión; migración completa de plataforma BI 200–500+ h. Con inventario incierto, tiende a BLOQUEO hasta auditar.
**Relacionados:** PAT-004, PAT-006.

---

## PAT-009 — Creación o modificación de datos maestros

**Qué es:** tablas maestras y estructuras de referencia compartidas (clientes, productos, equivalencias) que sirven de ejes comunes a múltiples hechos.

**Cuándo SÍ:** nuevo catálogo de referencia o equivalencias; añadir jerarquía a un maestro (familias/subfamilias); tabla de equivalencias mantenida a mano.
**Cuándo NO:** datos transaccionales o movimientos (→ PAT-003); tabla técnica interna de un único SP que no se expone como dimensión.

**Preguntas clave:** ¿qué fuente dicta la verdad del maestro (ERP, Microfocus, archivo manual)?; ¿quién tiene autoridad de altas/bajas?; ¿frecuencia de cambio?; ¿jerarquías?; ¿qué pasa con el histórico si un elemento se borra o renombra?

**Sospechas del consultor:** el origen recicla códigos — claves subrogadas obligatorias; aparecerán códigos huérfanos en las transacciones — prever registro por defecto `-1`; el Excel manual traerá espacios, mayúsculas cambiadas y filas vacías — limpieza estricta en Normalizado. Los maestros cargan SIEMPRE al principio del job nocturno.

**Capas/fases:** Normalizado y Dimensional (FN3).
**Unidades habituales:** `maestro_estandar` (4 h) / `maestro_con_integridades` (6 h) / `maestro_ds` (20 h).
**Orden de magnitud (solo contraste):** 20–40 h (20 si viene limpio de ERP; 40 con Excel manual y jerarquías).
**Relacionados:** PAT-003 (el maestro acaba siendo una Dim), PAT-010.

---

## PAT-010 — Nuevo modelo informativo end-to-end (macro-patrón)

**Qué es:** contenedor de proyectos de gran envergadura — un bloque funcional completo que no existe (Catastro, Censos, Ficha Fiscal…) recorriendo todo el pipeline: extracción, maestros, tablón, estrella, KPIs e informes.

**Cuándo SÍ:** módulo o requerimiento institucional completo nuevo que exige todas las capas.
**Cuándo NO:** ampliar un área ya construida (→ PAT-002 + PAT-004); requerimiento puramente técnico (→ PAT-006/PAT-008).

**Cómo se estima:** se descompone en instancias de patrones atómicos, en orden: N × PAT-001 (fuentes) → PAT-009 (maestros) → PAT-003 (DataMart) → PAT-005 (KPIs) → PAT-004 (informes). La estimación final es la suma agregada de los patrones activados.

**Preguntas clave:** marco normativo u objetivo institucional; interlocutores de ambas puntas (origen e informes); inventario preliminar de orígenes y volumen histórico; ¿fecha límite regulatoria?; criterios de éxito e informes de contraste; ¿arquitectura grande (3 BDs) o pequeña (1 BD con esquemas)?; ID de proceso de negocio.

**Sospechas del consultor:** la conectividad (permisos, firewalls) tarda semanas — lanzarla el día uno; el usuario no sabe lo que quiere hasta que lo ve — maquetas en paralelo al backend; decenas de procesos nuevos amenazan la ventana nocturna — incrementalidad estricta desde el diseño.

**Capas/fases:** todas (FN1–FN5).
**Orden de magnitud (solo contraste):** 120–250+ h.
**Relacionados:** contenedor de todos los demás.

---

## PAT-011 — Actualización de versión normativa recurrente

**Qué es:** incorporar una nueva versión normativa (Modelo 170, 196, campañas fiscales) a un modelo que YA existe en todas las capas del DW. Cambio estructurado, versionado y con especificación cerrada.

**Diferencia crítica con PAT-002:** PAT-002 es un cambio puntual (un campo, un tipo); PAT-011 es un cambio de versión completo — típicamente 3+ tablas nuevas y decenas de campos, impulsado por normativa formal.

**Cuándo SÍ:** el modelo existe en Staging + Normalizado + Dimensional + Cognos; versión normativa formal con especificación cerrada; múltiples elementos nuevos; flujo estándar corporativo (Generador C#, SSIS, T-SQL).
**Cuándo NO:** el modelo no existe (→ PAT-001); cambio puntual (→ PAT-002); solo visual (→ PAT-004); **especificación abierta o incierta → BLOQUEAR hasta documento normativo oficial** (no se estima bajo incertidumbre normativa).

**Preguntas clave:** ¿documento oficial de la versión (nombre, versión, fecha de vigor)?; nº de tablas nuevas **contadas** en la especificación; nº de campos y su tipo (simples vs con cálculos/jerarquías); ¿se modifican tablas existentes (renombrados, PKs)?; ¿cambia la lógica de validación?; ¿origen `DBMF*INFORMATIVA` accesible?; ¿dimensiones nuevas o solo ampliación?; ¿plazo regulatorio y margen de pruebas?

**Sospechas del consultor:** la "versión final" cambiará el mes siguiente — registrar versión y fecha como línea base; los campos "simples" pueden esconder jerarquías (preguntar); ¿el Generador soporta esta iteración? — validar, no asumir; la migración histórica "no pedida" aparecerá al final — dejarla como exclusión y riesgo explícito; en Cognos, ampliar campos es barato pero cambiar jerarquías/asuntos es caro — el discriminante es "¿cambia la estructura o solo la cantidad?".

**Capas/fases:** FN1 análisis (referencia 4–8 h con especificación cerrada) + FN2 Staging (tablas nuevas × `tabla_estandar`) + FN3 ETL (proceso DS/DM unificado según complejidad) + FN4 Cognos (`capa_fisica_cognos` + campos × `campo_cognos`) + FN5 transversal 10%.
**Orden de magnitud (solo contraste):** simple (3–5 tablas, 50–100 campos): 100–150 h; media (5–10 tablas, 100–200 campos): 150–250 h; compleja (10+ tablas, 200+ campos, cambios de lógica): 250–400+ h. Modificadores como riesgo/supuesto: ampliación del Generador +20–40 h; cambio de lógica de validación/SCD +15–30 h; migración histórica +30–100 h; cambio de arquitectura Cognos +20–50 h.
**Instancia real de referencia:** Modelo 170 (8 tablas, 140 campos, lógica simple, Generador validado) ≈ 137 h.
**Relacionados:** PAT-001 (caso base original), PAT-002 (hermano menor), PAT-003/PAT-004 (si amplía dimensional/reporting), PAT-010 (orquestador si incluye gestión de cambio).
