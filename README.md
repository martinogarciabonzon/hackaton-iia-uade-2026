# M08 · ¿Llega a la maratón por debajo de 4 horas?
**Integrantes:** Martino García Bonzón
**IA declarada:** Gemini
**Pista:** Reglas

## 1. Formulación y PEAS
* **Pregunta:** ¿Llega el corredor a la maratón por debajo de las 4 horas?
* **Clase Positiva (1):** SÍ completa en < 240 min. **Clase Negativa (0):** NO completa en < 240 min.
* **Objetivo vs. Métrica:** El sistema optimiza estrictamente la **Exactitud (Accuracy)** en entrenamiento para responder la pregunta de la ficha. No optimiza la precisión sobre la prueba; los Falsos Positivos se evalúan a posteriori como riesgo de negocio.
* **PEAS:**
  * **P (Rendimiento):** Maximizar Exactitud superando la línea base trivial.
  * **E (Entorno):** Corredores amateurs en preparación.
  * **A (Actuadores):** Ajuste de pacing de carrera o replanificación.
  * **S (Sensores):** Historial (Ritmo 10K, Fondo más largo, Semanas sin lesión, Km semanales).

## 2. El sistema
* **Paradigma:** **Reglas** de Árbol de Decisión (`max_depth=3`). Sin Data Leakage.

**Reglas de Decisión:**
1. Si `Ritmo en 10K` <= 5.60:
   * Si `Fondo más largo` <= 21.00:
     * Si `Km semanales` <= 49.50 -> **Clase 1**
     * Si `Km semanales` > 49.50 -> **Clase 0**
   * Si `Fondo más largo` > 21.00 -> **Clase 1**
2. Si `Ritmo en 10K` > 5.60:
   * Si `Fondo más largo` <= 32.00 -> **Clase 0**
   * Si `Fondo más largo` > 32.00 -> **Clase 1**

**Instrucciones para correr en vivo (Paso a paso):** 
Identificar las variables del nuevo corredor y bajar manualmente por las reglas.
* **Ejemplo A:** Ritmo=5.2, Fondo=25 -> Va a Regla 1 (5.2 <= 5.60) -> Sub-regla (25 > 21.00) -> Resultado: **Clase 1**.
* **Ejemplo B:** Ritmo=5.8, Fondo=25 -> Va a Regla 2 (5.8 > 5.60) -> Sub-regla (25 <= 32.00) -> Resultado: **Clase 0**.

## 3. Medición
**Entrenamiento (14 casos):** TP:7, TN:7, FP:0, FN:0 | Exactitud: 1.00 | Prec: 1.00 | Rec: 1.00
**Prueba (6 casos):** TP:1, TN:3, FP:0, FN:2 | Exactitud: 0.67 | Prec: 1.00 | Rec: 0.33

**Brecha y Generalización:** La caída de 1.00 a 0.67 es **Sobreajuste**. Con solo 6 casos en prueba, un solo caso que cambie altera la Exactitud en ~16.7%. Debido a esta altísima varianza, **no se puede afirmar que el modelo generalice robustamente**.

## 4. El error más caro
* **Falso Positivo (FP):** Predecir sub-4h a quien no puede. Causa riesgo grave de lesión.
* **Mitigación (Sin Data Leakage):** Elegimos `max_depth=3` *a priori* basándonos estrictamente en el tamaño minúsculo del entrenamiento ($N=14$) para evitar sobreajustar peor. **NO** lo ajustamos mirando la prueba. Lograr 0 FP en prueba fue un resultado empírico de un modelo simple, no un hiperparámetro forzado evaluando sobre la prueba.

## 5. ¿Comparado contra qué?
* **Línea base trivial:** Predecir clase 0. Da 50.0% (3/6). Medido por nosotros en `df_test`.
* Nuestro sistema (66.7%) supera la base en 16.7%.

## 6. Riesgo
Datos previos -> Árbol -> Decisión de pacing -> FN: carrera conservadora; FP (riesgo severo) no sucedió empíricamente.

## 7. Bitácora
* **Corrida 1 (6/10):** Fallas en PEAS, pista, reglas, sobreajuste.
* **Corrida 2 (6/10):** Tope max 6 por presunta fuga de datos (optimizar mirando prueba). Falta generalización y casos vivos.
* **Corrida 3:** Se aclara que `max_depth=3` se fijó *a priori* por $N=14$, manteniendo pura la prueba. Se define Exactitud como único optimizador. Se agregan 2 casos en vivo. Se asume que 1 error (16.7%) impide afirmar generalización.
