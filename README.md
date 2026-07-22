# Agente Estimador LANTIK — Microsoft Copilot Studio (MVP 1.0)

Estandarización de estimaciones de proyectos BI en la organización mediante un asistente AI conversacional integrado en Microsoft Copilot Studio.

**Estado:** MVP 1.0 en producción desde julio 2026. Enfocado en estandarizar cómo el equipo estima. No es automatización, es **captura y reutilización de conocimiento**.

---

## Visión del Producto

### Problema que resuelve (MVP 1.0)

Cada consultor senior de BI estima diferente. La experiencia está dispersa, no documentada y no reutilizable. Resultado:
- Estimaciones inconsistentes para el mismo tipo de proyecto.
- Consultores nuevos tardan en calibrarse.
- El conocimiento corporativo se pierde.

### Solución (MVP 1.0)

Un agente conversacional que:
1. **Clasifica** automáticamente la petición (PAT-001..PAT-011: 11 patrones de proyecto).
2. **Diagnostica** mediante preguntas estructuradas (no adivina).
3. **Estima** aplicando un catálogo corporativo de tarifas v1.0 (Sin inventar horas).
4. **Entrega** el resultado en tabla Markdown copiable a Excel en segundos.

**Resultado:** todas las estimaciones siguen el mismo proceso → mejor calidad, mejor consistencia, documentación automática de supuestos.


---

## Contenido del repositorio

| Archivo | Para qué sirve | Dónde va en Copilot Studio |
|---|---|---|
| `instrucciones.txt` | El "cerebro" del agente: rol, proceso de 4 pasos y formato de salida | Sección **Instrucciones** del agente |
| `conf/tarifas_lantik.json` | Catálogo corporativo de tarifas v1.0 (horas por unidad de trabajo) | Sección **Conocimiento** |
| `conf/catalogo_patrones.md` | Biblioteca de patrones de proyecto (PAT-001..PAT-011) para clasificar peticiones | Sección **Conocimiento** |
| `conf/glosario_y_reglas.md` | Definiciones de complejidad ETL, Generador C#, roles y reglas | Sección **Conocimiento** |
| `instrucciones_standalone.txt` | Variante del prompt con las tarifas embebidas, para licencias sin archivos de conocimiento | Sección **Instrucciones** (alternativa) |
| `README.md` | Este manual | — (no se sube a Copilot Studio) |

---

## Inicio rápido (5 pasos)

### 1. Crear el agente en Copilot Studio

1. Entra en [https://copilotstudio.microsoft.com](https://copilotstudio.microsoft.com) con tu cuenta corporativa de Microsoft 365.
2. Selecciona el **entorno** de tu organización (arriba a la derecha).
3. Pulsa **Crear** (Create) → **Nuevo agente** (New agent).
4. Si te ofrece describir el agente conversando, pulsa **Omitir para configurar** (Skip to configure).
5. Rellena:
   - **Nombre:** `Estimador Senior de BI LANTIK`
   - **Descripción:** `Agente de estimación de proyectos BI (SQL Server, SSIS, Cognos, Power BI) basado en el catálogo de tarifas corporativas de LANTIK.`
   - **Idioma:** Español
6. Pulsa **Crear** (Create).

### 2. Configurar las instrucciones

1. Abre `instrucciones.txt` de este repositorio.
2. Selecciona todo (`Ctrl + A`) y cópialo (`Ctrl + C`).
3. En Copilot Studio, pestaña **Información general** → sección **Instrucciones** → **Editar**.
4. Borra y pega (`Ctrl + V`) el contenido completo.
5. Pulsa **Guardar** (Save).

> **Importante:** el proceso de 4 pasos y la regla FN5=10% están en este texto. Pegarlo íntegro es crítico.

### 3. Subir los documentos de conocimiento

1. Pestaña **Conocimiento** (Knowledge) → **+ Agregar conocimiento** (Add knowledge) → **Archivos** (Files).
2. Sube desde la carpeta `conf/`:
   - `tarifas_lantik.json`
   - `catalogo_patrones.md`
   - `glosario_y_reglas.md`
3. Espera a que aparezca **Listo** (Ready) — puede tardar unos minutos.
4. Verifica en **Información general** que el conocimiento cargado está activo y la opción de "información pública de la web" está **desactivada**.

### 4. Probar

1. Panel **Probar** (Test) → copia y pega esta petición:
   ```
   Necesito estimar la incorporación de 8 tablas nuevas de un modelo de Hacienda 
   que ya existe en el DW, con 140 campos nuevos a publicar en Cognos.
   ```
2. Verifica que el agente:
   - NO da horas de inmediato (espera a terminar el diagnóstico).
   - Hace preguntas de clasificación y cualificación.
   - Termina con una tabla Markdown y supuestos explícitos.

Resultado esperado: ~137 horas (corresponde a una actualización normativa de Modelo 170 — caso de prueba verificado).

### 5. Publicar

1. Cuando el comportamiento sea correcto, pulsa **Publicar** (Publish).
2. Pestaña **Canales** (Channels) → activa **Microsoft Teams** para que el equipo acceda desde Teams.

---

## Mantenimiento

| Cambio | Qué hacer |
|---|---|
| **Tarifas cambian** | Edita `conf/tarifas_lantik.json`, elimina el antiguo en Copilot Studio y sube el nuevo. No toques las instrucciones. |
| **Patrones o complejidades cambian** | Actualiza `conf/catalogo_patrones.md` o `conf/glosario_y_reglas.md` en el repo y resube en Copilot Studio. |
| **Proceso de estimación cambia** | Actualiza `instrucciones.txt`, repégalo en Copilot Studio y republica. |

---

## Roadmap

El MVP 1.0 resuelve la **estandarización en el equipo**. Las siguientes fases ampliarán el alcance:

### Fase 2 (H3 2026): Salida en YAML para integración con pipeline

**Objetivo:** que el agente genere el resultado no solo en Markdown-para-Excel, sino también en **YAML con frontmatter** compatible con el script Python existente (`tools/generar_excel.py` del repo principal).

**Beneficio:** automatizar la creación de expedientes de estimación en el DW. El YAML contendría:
```yaml
---
id: EST-AAAA-NNN
titulo: "[Título del proyecto]"
patron: PAT-XXX
tarifas_version: "1.0-corporativo"
bloques:
  - cod: FN1
    descripcion: "Análisis"
    tipo: analisis
    horas_fijas: 6.0
  - cod: FN2
    ...
---
# Narrative section in Markdown...
```

Desde el chat o Teams, el usuario podría hacer: _"Quiero guardar esta estimación"_ y el agente generaría el YAML listo para pasar al script de generación de Excel/Expediente.

### Fase 2: Publicación como servicio web con API REST

**Objetivo:** exponer el agente de Copilot Studio + el generador de Excel como un **servicio API** consumible desde aplicaciones externas (portales, sistemas PM, integraciones).

**Arquitectura:**
```
Cliente (Portal PM)
    ↓ HTTP POST {peticion}
    ↓
API REST (wrapper Python)
    ↓
Agente Copilot Studio (clasificación + diagnóstico)
    ↓
Script Python (generar_excel.py)
    ↓
Respuesta JSON {expediente, tabla, xlsx_base64}
    ↓
Cliente (descarga/visualiza)
```

**Beneficio:** integración profunda en flujos de PM corporativos. Posibilidad de:
- Portal de estimaciones con histórico y analítica.
- Webhooks para auditoría de cambios de tarifas.
- Versionado automático de expedientes.



---
