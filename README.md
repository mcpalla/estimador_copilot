# Estimador Senior de BI LANTIK — Microsoft Copilot Studio (MVP 1.0)

Estandarización de estimaciones de proyectos BI mediante un asistente AI conversacional integrado en Microsoft Copilot Studio.

**Estado:** MVP 1.0 en producción. Enfocado en transferencia de conocimiento y coherencia de estimaciones en el equipo.

---

## Descripción oficial del agente

Copia y pega esto en Copilot Studio (sección **Información general** → **Descripción**):

```
Estimador corporativo de proyectos BI para LANTIK. Aplica el catálogo 
de tarifas v1.0 y los patrones de proyecto estándar del equipo 
para guiar diagnósticos estructurados de requisitos. Entrega estimaciones 
coherentes, trazables y documentadas, facilitando el onboarding de nuevos 
miembros del equipo y la estandarización de criterios en la organización.
```

---

## Visión del producto

### Problema que resuelve

El equipo está en expansión con nuevos miembros que necesitan acceso rápido al conocimiento corporativo de estimaciones. Hoy:

- Cada uno desarrolla su propio "catálogo mental" de criterios — distinto según de quién haya aprendido.
- Se pierden patrones de proyecto (ej. "toda actualización normativa de Hacienda cuesta X horas").
- Las nuevas tipologías de proyecto que descubrimos no se documentan.
- No hay coherencia si en el futuro queremos industrializar la pipeline.

### Solución (MVP 1.0)

Convertir el conocimiento disperso en **conocimiento formalizado y reutilizable**:

1. **Captura:** catálogo de tarifas, patrones de proyecto (PAT-001..PAT-011) y reglas de complejidad extraídos de casos pasados.
2. **Estandariza:** clasificar → diagnosticar → descomponer → sentenciar (4 pasos estructurados, no a ojo).
3. **Facilita:** nuevos miembros consultan el agente sin buscar casos antiguos; el resultado se documenta automáticamente.
4. **Escala:** descubrimos patrones nuevos → los agregamos; cambian tarifas → actualizamos centralmente; si industrializamos, ya están los criterios unificados.

**Resultado esperado:** independientemente de quién estime, la salida es consistente, trazable y aprendible para el equipo.

---

## Contenido

| Archivo | Para qué sirve |
|---|---|
| `instrucciones.txt` | Instrucciones del agente: rol, proceso de 4 pasos, formato de salida |
| `conf/tarifas_lantik.json` | Catálogo corporativo de tarifas v1.0 |
| `conf/catalogo_patrones.md` | Biblioteca de 11 patrones de proyecto (PAT-001..PAT-011) |
| `conf/glosario_y_reglas.md` | Definiciones de complejidad, roles y criterios |

---

## Configuración en Copilot Studio

### 1. Crear el agente

En [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com):

1. **Crear** → **Nuevo agente**
2. Nombre: `Estimador Senior de BI LANTIK`
3. Descripción: `Agente de estimación de proyectos BI (SQL Server, SSIS, Cognos, Power BI) basado en catálogo corporativo de tarifas.`
4. Idioma: **Español**

### 2. Configurar instrucciones

1. En tu agente, pestaña **Información general** → sección **Instrucciones**
2. Pega el contenido de `instrucciones.txt`
3. **Guardar**

### 3. Configurar fuente de conocimiento

**Opción A: Desde GitHub (recomendado para mantenimiento centralizado)**

1. Pestaña **Conocimiento** → **+ Agregar conocimiento** → **Sitios web específicos**
2. Introduce la URL: `https://github.com/mcpalla/estimador_copilot/tree/master/conf`
3. Activa **Usar solo fuentes especificadas**
4. Guarda y espera el reindexado (1-2 minutos)



**En cualquier caso:**
- Verifica que el conocimiento cargado está **activo**
- La opción de "información pública de la web" debe estar **desactivada**


### 4. Probar

La carpeta test/ contiene peticiones de ejemplo para validar el comportamiento del agente.

Estos casos también sirven para validar que el agente mantiene la coherencia después de actualizaciones en tarifas, patrones o instrucciones.



### 5. Publicar

Cuando el comportamiento sea correcto:
1. **Publicar**


---



## Roadmap

### Fase 2 (H3 2026): Exportación en YAML & API REST

El agente generará resultados también en **YAML con frontmatter**, compatible con scripts de automatización. Permite guardar expedientes de forma estructurada sin reescribir.

Crear un estimador en forma de script de Python cuya función es generar el MS Excel a partir de la información contenida en el output del estimador.

Publicar el estimador como **servicio web**: consumible desde portales PM, sistemas de integración y aplicaciones externas. Abre la puerta a automatización profunda.



---

## Soporte

**¿El agente da respuestas incorrectas?**
- Verifica que los archivos `conf/` de este repositorio Github estén actualizados.


**¿Las tarifas están desactualizadas?**
- Edita `conf/tarifas_lantik.json`
