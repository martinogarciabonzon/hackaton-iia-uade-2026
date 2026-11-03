# M08 · ¿Llega a la maratón por debajo de 4 horas?
**Integrantes:** Martino García Bonzón
**IA declarada:** Gemini
**Pista:** Reglas

## 1. Formulación y PEAS
* **Pregunta:** ¿Llega a la maratón por debajo de 4 horas?
* **P (Rendimiento):** El objetivo de negocio es proteger la integridad física del corredor, evitando sobreesfuerzos. La métrica técnica elegida es maximizar la **Precisión** de la clase positiva, minimizando los Falsos Positivos.
* **E (Entorno):** Corredores amateurs en etapa de preparación previa a una maratón.
* **A (Actuadores):** Ajuste de la estrategia de carrera (pacing), replanificación del entrenamiento semanal, o redefinición del objetivo de tiempo.
* **S (Sensores):** Historial de entrenamientos (Fondo más largo, Ritmo en 10K, Semanas sin lesión, Km semanales).

## 2. El sistema
* Sistema de clasificación binaria utilizando un Árbol de Decisión (max_depth=3) en scikit-learn.
* El sistema es reproducible y utiliza únicamente datos históricos de entrenamiento previos a la carrera para predecir, garantizando la ausencia de fuga de datos.

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

## 4. El error más caro
* El error más caro es el **Falso Positivo**: predecir que el corredor bajará las 4 horas cuando no tiene la capacidad física real. Esto provocaría que el corredor salga a un ritmo insostenible, arriesgándose a calambres, lesiones agudas ("chocar contra el muro") o problemas de salud severos.
* **Mitigación:** Utilizamos un modelo restrictivo que logró 0 Falsos Positivos en la prueba (Precisión 1.00), priorizando pecar de conservadores (asumiendo Falsos Negativos aceptables).

## 5. ¿Comparado contra qué?
* **Línea base trivial:** Un modelo conservador extremo que asuma que *ningún* corredor logrará bajar de las 4 horas (predecir siempre clase 0). Dado el balance del dataset de prueba, esta regla lograría una Exactitud del 50%.
* Nuestro sistema supera la línea base trivial alcanzando un 67% de Exactitud en prueba, aportando valor predictivo sin sacrificar la seguridad (Precisión perfecta).

## 6. Riesgo
* **Datos:** Se toman los registros de kilometraje, ritmos y lesiones del plan de entrenamiento.
* **Modelo:** El Árbol de Decisión clasifica el potencial del corredor.
* **Decisión:** El entrenador o corredor decide si sale a buscar la marca de sub-4h o si adopta un ritmo conservador.
* **Consecuencia:** Si el modelo acierta, el corredor optimiza su carrera de forma segura. Si falla (FN), el corredor se frustra por correr lento pero termina sano. Se evita activamente la peor consecuencia (riesgo físico severo derivado de un FP).

## 7. Bitácora

**Uso de la IA (Gemini):**
* **Qué le pedimos:** Ayuda para auditar el dataset, estructurar el PEAS según la rúbrica y verificar matemáticamente las matrices de confusión para evitar topes de penalización.
* **Qué aceptamos y por qué:** Aceptamos la sugerencia de cambiar nuestra métrica objetivo de "Exactitud" a "Precisión". Comprendimos que optimizar la Exactitud era peligroso para el negocio, ya que debíamos penalizar específicamente el Falso Positivo para proteger la integridad física del corredor.
* **Qué rechazamos y por qué:** Le prohibimos a la IA programar el código completo de una vez o usar algoritmos complejos. Elegimos mantener un Árbol de Decisión muy simple (`max_depth=3`) y calcular los pasos localmente, ya que con solo 14 filas de entrenamiento, un modelo complejo iba a generar un sobreajuste masivo sin sentido.

**Historial de Corridas:**
* **Corrida 1:** Envío inicial del modelo base restrictivo priorizando 0 Falsos Positivos. (Esperando feedback).
