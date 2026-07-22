# Agente Estimador LANTIK — Manual rápido (MVP 1.0)

Guía para consultores de negocio: cómo montar el agente en **Microsoft Copilot Studio** y cómo usarlo para obtener estimaciones listas para Excel en segundos. No se necesita ningún conocimiento técnico ni instalación de software.

## Qué contiene esta carpeta

| Archivo | Para qué sirve | Dónde va en Copilot Studio |
|---|---|---|
| `instrucciones.txt` | El "cerebro" del agente: rol, proceso de 4 pasos y formato de salida | Sección **Instrucciones** del agente |
| `conf\tarifas_lantik.json` | Catálogo corporativo de tarifas v1.0 (horas por unidad de trabajo) | Sección **Conocimiento** |
| `conf\catalogo_patrones.md` | Biblioteca de patrones de proyecto (PAT-001..PAT-011) para clasificar peticiones | Sección **Conocimiento** |
| `conf\glosario_y_reglas.md` | Definiciones de complejidad ETL, Generador C#, roles y reglas | Sección **Conocimiento** |
| `instrucciones_standalone.txt` | Variante del prompt con las tarifas embebidas, para licencias sin archivos de conocimiento | Sección **Instrucciones** (alternativa) |
| `README.md` | Este manual | — (no se sube) |

---

## 1. Crear el agente en Copilot Studio

1. Entra en [https://copilotstudio.microsoft.com](https://copilotstudio.microsoft.com) con tu cuenta corporativa de Microsoft 365.
2. Selecciona el **entorno** de tu organización (arriba a la derecha) — si tienes dudas, usa el que te indique IT.
3. Pulsa **Crear** (Create) en el menú lateral y elige **Nuevo agente** (New agent).
4. Si te ofrece describir el agente conversando, pulsa **Omitir para configurar** (Skip to configure) para pasar directamente a la configuración manual.
5. Dale estos datos básicos:
   - **Nombre:** `Estimador Senior de BI LANTIK`
   - **Descripción:** `Agente de estimación de proyectos BI (SQL Server, SSIS, Cognos, Power BI) basado en el catálogo de tarifas corporativas de LANTIK.`
   - **Idioma:** Español
6. Pulsa **Crear** (Create). Ya tienes el agente vacío.

## 2. Pegar las instrucciones del agente

1. Abre el archivo `instrucciones.txt` de esta carpeta (doble clic; se abre con el Bloc de notas).
2. Selecciona **todo** el contenido (`Ctrl + A`) y cópialo (`Ctrl + C`).
3. En Copilot Studio, dentro de tu agente, ve a la pestaña **Información general** (Overview) y localiza el cuadro **Instrucciones** (Instructions).
4. Pulsa **Editar**, borra lo que hubiera, y pega (`Ctrl + V`) el contenido completo.
5. Pulsa **Guardar** (Save).

> Importante: pega el texto **íntegro y sin modificar**. El proceso de 4 pasos y la regla del 10% de la fase FN5 están escritos ahí; si se recorta, el agente dejará de calcular bien.

## 3. Subir los documentos de conocimiento

1. En tu agente, ve a la pestaña **Conocimiento** (Knowledge).
2. Pulsa **+ Agregar conocimiento** (Add knowledge) y elige **Archivos** (Files).
3. Arrastra o selecciona estos tres archivos de la carpeta `conf`:
   - `tarifas_lantik.json`
   - `catalogo_patrones.md`
   - `glosario_y_reglas.md`

4. Confirma y espera a que el estado de ambos documentos pase a **Listo** (Ready) — puede tardar unos minutos mientras se indexan.
5. Comprueba en **Información general** que la opción de usar conocimiento cargado está activada, y desactiva la opción de **usar conocimiento general de la IA / información pública de la web** si aparece: el agente solo debe responder con las tarifas de LANTIK.

## 4. Probar y publicar

1. Usa el panel **Probar** (Test) a la derecha y escribe una petición real, por ejemplo:
   > "Necesito estimar la incorporación de 8 tablas nuevas de un modelo de Hacienda que ya existe en el DW, con 140 campos nuevos a publicar en Cognos."
2. Verifica que el agente **no da horas de inmediato**: primero confirma el objetivo (Paso 1) y hace preguntas de diagnóstico (Paso 2). Responde a sus preguntas.
3. Al final debe entregar la **Sentencia Final**: un resumen narrativo + la tabla de estimación con las columnas `Cod. | Fase/Concepto | Horas | JP (h) | AN (h) | DT (h) | PR (h)`, incluyendo la fila FN5 (10% sobre FN2+FN3+FN4) y la fila TOTAL.
4. Cuando el comportamiento sea correcto, pulsa **Publicar** (Publish).
5. En la pestaña **Canales** (Channels), activa **Microsoft Teams** para que todo el equipo pueda chatear con el agente desde Teams.



## 5. Mantenimiento del conocimiento

- **Cambian las tarifas:** edita `tarifas_lantik.json` (o pide al equipo del proyecto que lo regenere desde `knowledge/estimation/tarifas-corporativas.yaml`), y en Copilot Studio elimina el archivo antiguo de Conocimiento y sube el nuevo. No hace falta tocar las instrucciones.
- **Cambian los criterios de complejidad:** mismo procedimiento con `glosario_y_reglas.md`.
- **Cambia el proceso de estimación:** actualiza `instrucciones.txt` y vuelve a pegarlo en la sección Instrucciones. Después, vuelve a **Publicar**.

## 6. Límites del MVP 1.0

- El agente **estima**; no hace seguimiento de proyectos ni compara estimado vs real (fases posteriores del roadmap).
- Solo conoce las unidades del catálogo corporativo v1.0. Si pides algo fuera de catálogo, lo declarará como "conocimiento faltante" en lugar de inventarse horas — es el comportamiento esperado.
- La estimación final siempre incluye supuestos y exclusiones: **léelos antes de trasladar las horas al cliente**.
