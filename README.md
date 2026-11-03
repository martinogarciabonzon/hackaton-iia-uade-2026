# M08 · ¿Llega a la maratón por debajo de 4 horas?
**Integrantes:** Martino García Bonzón
**IA declarada:** Gemini
**Pista:** Reglas

## 1. Formulación y PEAS
* **Pregunta:** ¿Llega el corredor a la maratón por debajo de las 4 horas?
* **Clase Positiva (1):** El corredor SÍ logra completar la maratón en menos de 4 horas (< 240 minutos).
* **Clase Negativa (0):** El corredor NO logra completar la maratón por debajo de 4 horas (>= 4 horas).
* **Objetivo de Negocio vs. Métrica Técnica:** El objetivo de negocio es clasificar correctamente la capacidad del corredor para asignarle una estrategia de carrera acorde a su potencial. Para responder a la pregunta planteada, la métrica técnica principal a optimizar es la **Exactitud (Accuracy)**, utilizando la **Precisión** como métrica de control de riesgo crítico para minimizar los Falsos Positivos.
* **PEAS:**
  * **P (Rendimiento):** Maximizar la Exactitud (Accuracy) en la respuesta del problema (superando la línea base trivial) asegurando al mismo tiempo una Precisión de 1.00 (0 Falsos Positivos).
  * **E (Entorno):** Corredores amateurs en etapa de preparación previa a una maratón.
  * **A (Actuadores):** Ajuste de la estrategia de ritmo (pacing), replanificación del entrenamiento semanal, o redefinición del objetivo de tiempo a uno más conservador.
  * **S (Sensores):** Historial de entrenamientos previos (Ritmo en 10K, Fondo más largo, Semanas sin lesión, Km semanales).

## 2. El sistema
* **Paradigma:** Sistema experto basado en **Reglas** extraídas de un Árbol de Decisión (`max_depth=3`) entrenado en `scikit-learn`.
* **Ausencia de Data Leakage:** Se descartaron variables identificadoras (`caso`) y se usaron únicamente métricas previas al día de la maratón.

**Reglas de Decisión del Árbol:**
1. Si `Ritmo en 10K` <= 5.60 min/km:
   * Si `Fondo más largo` <= 21.00 km:
     * Si `Km semanales` <= 49.50 km -> **Clase 1** (Llega sub-4h)
     * Si `Km semanales` > 49.50 km -> **Clase 0** (NO llega sub-4h)
   * Si `Fondo más largo` > 21.00 km -> **Clase 1** (Llega sub-4h)
2. Si `Ritmo en 10K` > 5.60 min/km:
   * Si `Fondo más largo` <= 32.00 km -> **Clase 0** (NO llega sub-4h)
   * Si `Fondo más largo` > 32.00 km -> **Clase 1** (Llega sub-4h)

**Instrucciones para correr un caso nuevo:**
Cualquier evaluador puede ejecutar un caso nuevo siguiendo las reglas anteriores o mediante el modelo en Python. Por ejemplo, ante un nuevo corredor con `Ritmo en 10K = 5.2 min/km` y `Fondo más largo = 25 km`, se evalúa la Regla 1 (`5.2 <= 5.60`), luego la sub-regla (`25 > 21.00`), concluyendo con resultado **Clase 1**.

## 3. Medición
**Entrenamiento (14 casos):**
* Matriz de confusión: Verdaderos Positivos (7), Verdaderos Negativos (7), Falsos Positivos (0), Falsos Negativos (0).
* Exactitud (Accuracy): 1.00
* Precisión: 1.00
* Sensibilidad (Recall): 1.00
* Especificidad: 1.00

**Prueba (6 casos):**
* Matriz de confusión: Verdaderos Positivos (1), Verdaderos Negativos (3), Falsos Positivos (0), Falsos Negativos (2).
* Exactitud (Accuracy): 0.67
* Precisión: 1.00
* Sensibilidad (Recall): 0.33
* Especificidad: 1.00

**Explicación de la brecha Entrenamiento vs. Prueba (Sobreajuste):**
La brecha entre el 1.00 de entrenamiento y el 0.67 de prueba es un fenómeno de **Sobreajuste (Overfitting)**. Debido al tamaño acotado del conjunto de entrenamiento ($N=14$), el algoritmo memorizó patrones específicos de esos datos. Al evaluar en prueba ($N=6$), el modelo perdió algo de generalización (errando en 2 Falsos Negativos), pero mantuvo un comportamiento conservador alcanzando 100% de Precisión sin cometer Falsos Positivos.

## 4. El error más caro
* El error más caro es el **Falso Positivo (FP)**: predecir que el corredor bajará las 4 horas cuando no tiene la capacidad física real. Esto llevaría a salir a un ritmo insostenible, provocando colapso físico, calambres graves o lesiones.
* **Qué cambiamos en el modelo para cometerlo menos:** Limitamos el hiperparámetro de profundidad a `max_depth=3` para podar ramas complejas y forzar condiciones estrictas para la Clase 1 (exigiendo ritmos rápidos en 10K o fondos largos mayores a 32 km). Esto redujo a **0 los Falsos Positivos en la prueba**.

## 5. ¿Comparado contra qué?
* **Línea base trivial:** Un modelo trivial que prediga siempre la clase mayoritaria/conservadora (Clase 0: NO baja las 4h). Dado el balance del conjunto de prueba (3 casos 0 y 3 casos 1), la exactitud de esta regla básica es de 50.0% (0.50).
* **Quién lo midió:** Fue medido directamente por nosotros sobre el conjunto oficial de prueba (`df_test`, $N=6$).
* **Comparación:** Nuestro sistema (66.7% de Exactitud) supera en 16.7 puntos porcentuales a la línea base trivial (50.0%), demostrando valor predictivo real con 0 Falsos Positivos.

## 6. Riesgo
* **Datos:** Registros de kilometraje y ritmo previos a la maratón.
* **Modelo:** Árbol de decisión basado en reglas explicitadas.
* **Decisión:** Definición de la estrategia de ritmo (pacing) para el día de la carrera.
* **Consecuencia:** Un acierto o un FN lleva a una carrera segura o conservadora; se mitiga totalmente la consecuencia grave (lesiones por salir sobreexigido) al no registrar Falsos Positivos.

## 7. Bitácora
**Uso de la IA (Gemini):**
* **Qué le pedimos:** Auditoría de requisitos, extracción de reglas lógicas del modelo y estructuración del README según la rúbrica del bot.
* **Qué aceptamos y por qué:** Reorganizar la formulación para declarar a la Exactitud como métrica principal alineada a la pregunta, manteniendo la Precisión como control de riesgo.
* **Qué rechazamos y por qué:** Modelos complejos o ensambles (Random Forest) que hubiesen oscurecido la interpretación y causado mayor sobreajuste en un dataset de solo 14 filas.

**Historial de Corridas:**
* **Corrida 1 (Nota: 6/10):** Agujero principal: desalineación en el PEAS de la métrica objetivo respecto a la pregunta, falta de definición de clase positiva, pista técnica no válida (`Atletismo`), falta de reglas explícitas del árbol e instrucciones de uso, y falta de justificación del sobreajuste y medición de la línea base.
* **Corrida 2:** Se cambió la pista a `Reglas`, se explicitaron las reglas lógicas extraídas y el manual de evaluación para casos nuevos. Se definió la Clase Positiva (1) y la Exactitud como métrica principal. Se detalló la causa del sobreajuste (Train 1.00 vs Test 0.67 por $N=14$), se justificó `max_depth=3` como mitigador de Falsos Positivos y se detalló la medición de la línea base sobre `df_test`.
