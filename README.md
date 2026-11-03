# M08 · ¿Llega a la maratón por debajo de 4 horas?
**Integrantes:** Martino García Bonzón
**IA declarada:** Gemini
**Pista:** Reglas

## 1. Formulación y PEAS
* **Pregunta:** ¿Llega el corredor a la maratón por debajo de las 4 horas?
* **Clase Positiva (1):** SÍ completa en < 240 min. **Clase Negativa (0):** NO completa en < 240 min.
* **Objetivo vs. Métrica:** El sistema optimiza la **Exactitud (Accuracy)** en entrenamiento para responder la pregunta. No se optimizó sobre la prueba. La **Precisión** actúa como control de riesgo de negocio (evaluado a posteriori).
* **PEAS:**
  * **P (Rendimiento):** Maximizar Exactitud superando línea base.
  * **E (Entorno):** Corredores amateurs en preparación.
  * **A (Actuadores):** Ajuste de pacing de carrera o replanificación de meta.
  * **S (Sensores):** Historial (Ritmo 10K, Fondo más largo, Semanas sin lesión, Km semanales).

## 2. El sistema
* **Paradigma:** **Reglas** de Árbol de Decisión (`max_depth=3`). Sin Data Leakage (variables tomadas previo a la carrera, sin ID).

**Reglas de Decisión:**
1. Si `Ritmo en 10K` <= 5.60:
   * Si `Fondo más largo` <= 21.00:
     * Si `Km semanales` <= 49.50 -> **Clase 1**
     * Si `Km semanales` > 49.50 -> **Clase 0**
   * Si `Fondo más largo` > 21.00 -> **Clase 1**
2. Si `Ritmo en 10K` > 5.60:
   * Si `Fondo más largo` <= 32.00 -> **Clase 0**
   * Si `Fondo más largo` > 32.00 -> **Clase 1**

**Instrucciones para correr en vivo:** Identificar variables y bajar por las reglas:
* **Ejemplo A:** Ritmo=5.2, Fondo=25 -> Regla 1 (5.2 <= 5.60) -> Sub-regla (25 > 21.00) -> Resultado: **Clase 1**.
* **Ejemplo B:** Ritmo=5.8, Fondo=25 -> Regla 2 (5.8 > 5.60) -> Sub-regla (25 <= 32.00) -> Resultado: **Clase 0**.

## 3. Medición
**Entrenamiento (14 casos):** TP:7, TN:7, FP:0, FN:0 | Exactitud: 1.00 | Prec: 1.00 | Rec: 1.00
**Prueba (6 casos):** TP:1, TN:3, FP:0, FN:2 | Exactitud: 0.67 | Prec: 1.00 | Rec: 0.33

**Brecha:** La caída a 0.67 es **Sobreajuste**. Con 6 casos, errar uno altera 16.7%. La alta varianza impide afirmar generalización robusta.

## 4. El error más caro
* **Falso Positivo (FP):** Predecir sub-4h a quien no puede. Riesgo grave de lesión.
* **Qué cambiamos:** Introdujimos un sesgo conservador limitando a `max_depth=3` a priori. Sin embargo, el sustento de tener exactamente 0 FP en prueba **son los 6 casos** (el azar de una muestra minúscula), no una garantía del modelo.

## 5. ¿Comparado contra qué?
* **Línea base trivial:** Predecir clase 0 (mayoritaria/conservadora). Da 50.0% (3/6). Medido por nosotros en `df_test`.
* Nuestro sistema (66.7%) supera la base en 16.7%.

## 6. Riesgo
Datos previos -> Árbol -> Decisión de pacing -> FN: carrera segura/lenta; FP (riesgo severo) mitigado.

## 7. Bitácora
**Uso de IA (Gemini):**
* **Qué pedimos:** Extracción de reglas y estructuración de la rúbrica.
* **Qué aceptamos:** Definir la Exactitud como métrica principal alineada a la pregunta.
* **Qué rechazamos:** Modelos ensamblados complejos (ej. Random Forest) para no oscurecer la interpretabilidad de las reglas.

**Historial:**
* **Corrida 1 y 2:** Fallas en PEAS, justificación de sobreajuste, Data Leakage al justificar métricas y manual de uso.
* **Corrida 3:** Explicación de alta varianza estadística y límite de generalización. Falta de detalle en bitácora IA.
* **Corrida 4:** Se detalla uso de IA. Se asume que el sustento de 0 FP son los 6 casos de prueba (suerte estadística) y se identifica que reglas profundas (como Km semanales) están sostenidas por apenas 1 o 2 casos de entrenamiento, volviendo al modelo altamente inestable si ingresan datos nuevos.
