# M08 · ¿Llega a la maratón por debajo de 4 horas?
**Integrantes:** Martino García Bonzón
**IA declarada:** Gemini
**Pista:** Reglas

## 1. Formulación y PEAS
* **Pregunta:** ¿Llega el corredor a la maratón por debajo de las 4 horas?
* **Clase Positiva (1):** SÍ completa en < 240 min. **Clase Negativa (0):** NO completa en < 240 min.
* **Objetivo vs. Métrica:** El objetivo de negocio es clasificar correctamente al corredor para definir su estrategia de ritmo. La métrica principal a optimizar para responder la pregunta es la **Exactitud (Accuracy)**, usando la **Precisión** como control de riesgo para eliminar Falsos Positivos.
* **PEAS:**
  * **P (Rendimiento):** Maximizar Exactitud superando la línea base trivial, garantizando Precisión de 1.00 (0 FP).
  * **E (Entorno):** Corredores amateurs en preparación previa a maratón.
  * **A (Actuadores):** Ajuste de pacing de carrera, plan de entrenamiento o meta de tiempo.
  * **S (Sensores):** Historial previo (Ritmo 10K, Fondo más largo, Semanas sin lesión, Km semanales).

## 2. El sistema
* **Paradigma:** Sistema experto basado en **Reglas** extraídas de un Árbol de Decisión (`max_depth=3`) en `scikit-learn`.
* **Sin Data Leakage:** Se eliminó `caso` (ID) y solo se usan datos previos a la carrera.

**Reglas de Decisión:**
1. Si `Ritmo en 10K` <= 5.60:
   * Si `Fondo más largo` <= 21.00:
     * Si `Km semanales` <= 49.50 -> **Clase 1** | Si `Km semanales` > 49.50 -> **Clase 0**
   * Si `Fondo más largo` > 21.00 -> **Clase 1**
2. Si `Ritmo en 10K` > 5.60:
   * Si `Fondo más largo` <= 32.00 -> **Clase 0**
   * Si `Fondo más largo` > 32.00 -> **Clase 1**

**Caso nuevo:** Con `Ritmo 10K = 5.2` y `Fondo largo = 25`, se aplica Regla 1 (`5.2 <= 5.60`) y sub-regla (`25 > 21.00`), resultando en **Clase 1**.

## 3. Medición
**Entrenamiento (14 casos):**
* Matriz: TP:7, TN:7, FP:0, FN:0
* Exactitud: 1.00 | Precisión: 1.00 | Sensibilidad: 1.00 | Especificidad: 1.00

**Prueba (6 casos):**
* Matriz: TP:1, TN:3, FP:0, FN:2
* Exactitud: 0.67 | Precisión: 1.00 | Sensibilidad: 0.33 | Especificidad: 1.00

**Brecha Train vs. Test (Sobreajuste):** Caída de 1.00 a 0.67 por **Sobreajuste (Overfitting)** debido a muestra reducida ($N=14$). En test ($N=6$) erra 2 FN pero mantiene Precisión 1.00 (0 FP).

## 4. El error más caro
* **Falso Positivo (FP):** Predecir sub-4h a quien no puede. Provoca ritmo insostenible, colapso físico o lesiones.
* **Mitigación:** Se fijó `max_depth=3` para podar ramas y forzar condiciones estrictas para Clase 1, logrando **0 FP en prueba**.

## 5. ¿Comparado contra qué?
* **Línea base trivial:** Predecir siempre la clase mayoritaria (Clase 0). Obtiene 50.0% de exactitud (3/6).
* **Quién lo midió:** Medido directamente por nosotros sobre `df_test` ($N=6$).
* **Comparación:** Nuestro sistema (66.7%) supera en 16.7% a la línea base con 0 FP.

## 6. Riesgo
* **Datos:** Kilometraje y ritmos previos -> **Modelo:** Reglas del árbol -> **Decisión:** Pacing de carrera -> **Consecuencia:** Acierto/FN lleva a carrera segura/conservadora; se elimina riesgo de lesión severa por FP.

## 7. Bitácora
**Uso de IA (Gemini):**
* **Pedimos:** Auditoría de rúbrica, extracción de reglas y estructuración.
* **Aceptamos:** Exactitud como métrica principal alineada a la pregunta y Precisión como control de riesgo.
* **Rechazamos:** Modelos complejos (Random Forest) para evitar mayor sobreajuste en $N=14$.

**Historial:**
* **Corrida 1 (Nota 6/10):** Fallas en PEAS, pista, reglas explícitas, sobreajuste y línea base.
* **Corrida 2:** Pista `Reglas`, reglas explícitas, manual caso nuevo, clase positiva (1), Exactitud principal, causa sobreajuste ($N=14$), `max_depth=3` para 0 FP y medición línea base sobre `df_test`.
