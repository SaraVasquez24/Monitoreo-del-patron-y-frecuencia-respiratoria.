# Monitoreo-del-patron-y-frecuencia-respiratoria.
Sara Damaris Vasquez Cardenas y Paula Andrea Vega Pardo

---

## 1. Introducción

La respiración es un proceso fisiológico esencial que permite el intercambio gaseoso entre el organismo y el medio externo. Durante la inhalación, el oxígeno ingresa a los pulmones y atraviesa la membrana alveolocapilar hacia el torrente sanguíneo, mientras que en la exhalación el dióxido de carbono es expulsado como producto del metabolismo celular. Este proceso ocurre de manera rítmica y automática bajo el control del sistema nervioso central. La frecuencia respiratoria constituye uno de los signos vitales más importantes en la práctica clínica, ya que alteraciones en su valor pueden indicar deterioro fisiológico, insuficiencia respiratoria o eventos cardiovasculares inminentes. Diversos estudios han demostrado que cambios en la frecuencia respiratoria pueden anticipar eventos clínicos graves con alta especificidad, incluso antes de que otros signos vitales presenten alteraciones significativas.

En esta práctica se diseñó e implementó un sistema de monitoreo respiratorio con el fin de capturar el patrón respiratorio y calcular la frecuencia respiratoria de un individuo sano bajo dos condiciones específicas: reposo y verbalización. El propósito principal fue evaluar la influencia del habla sobre el comportamiento del ciclo respiratorio.

---

## 2. Objetivos

El objetivo general de la práctica fue evaluar la influencia del habla sobre el patrón y la frecuencia respiratoria. De manera específica, se buscó identificar las variables físicas involucradas en el proceso respiratorio, seleccionar un sensor adecuado para medir dichas variaciones, diseñar un sistema de adquisición y digitalización de la señal, capturar registros de 30 segundos en dos condiciones distintas, analizar las señales en el dominio del tiempo y de la frecuencia, e interpretar los resultados desde el punto de vista fisiológico.

---

## 3. Variable física y selección del sensor

La variable física seleccionada para esta práctica fue la variación mecánica del tórax asociada a la expansión y contracción durante el ciclo respiratorio. Cuando el sujeto inhala, el volumen torácico aumenta debido a la contracción del diafragma y los músculos intercostales, generando una expansión de la caja torácica. En la exhalación ocurre el fenómeno contrario. Estas variaciones pueden medirse indirectamente mediante sensores sensibles a cambios de presión.

Se seleccionó un sensor FSR (Force Sensitive Resistor), el cual modifica su resistencia eléctrica en función de la presión aplicada sobre su superficie. Este sensor fue elegido debido a su bajo costo, facilidad de implementación, compatibilidad con voltajes entre 3.3 y 5 V, y su carácter no invasivo. Aunque el FSR presenta una respuesta no lineal y puede ser sensible a movimientos externos, resulta adecuado para aplicaciones experimentales donde se requiere detectar cambios mecánicos relativos más que valores absolutos de presión.

---

## 4. Adaptación del sensor al sujeto

El sensor fue ubicado en la región lateral del tórax, aproximadamente sobre la zona costal, y fijado mediante una banda elástica alrededor del pecho. Esta disposición permitió mantener un contacto constante entre el sensor y la superficie corporal, reduciendo interferencias por desplazamiento. La presión ejercida por la expansión torácica durante la inhalación produce un aumento en la señal medida, mientras que la exhalación genera una disminución relativa.

---

## 5. Diseño del sistema de adquisición

La señal proveniente del sensor fue digitalizada utilizando el convertidor análogo-digital (ADC) integrado en la ESP32, configurado con una resolución de 12 bits. Esto permite obtener valores digitales en un rango de 0 a 4095. Se seleccionó una frecuencia de muestreo de 1000 Hz, valor suficientemente alto para capturar con precisión una señal respiratoria cuya frecuencia típica se encuentra entre 0.1 Hz y 0.5 Hz en adultos sanos.

La comunicación entre la ESP32 y MATLAB se realizó mediante puerto serial a una velocidad de 115200 baudios, garantizando una transmisión estable de los datos en tiempo real.

---

## 6. Funcionamiento del código en la ESP32

El código implementado en la ESP32 configura inicialmente la comunicación serial y la resolución del ADC. Posteriormente, dentro del ciclo principal, se utiliza la función micros() para garantizar una adquisición temporizada precisa. Se calcula el periodo de muestreo como el inverso de la frecuencia seleccionada, asegurando que cada muestra se tome aproximadamente cada 1 milisegundo. En cada iteración se realiza la lectura del pin analógico conectado al sensor FSR y el valor obtenido es enviado inmediatamente por el puerto serial hacia el computador.

Este procedimiento permite obtener una señal digital continua correspondiente a las variaciones mecánicas del tórax durante la respiración.

---

## 7. Funcionamiento del código en MATLAB

En MATLAB se estableció la comunicación serial indicando el puerto correspondiente y la velocidad de transmisión. Se creó un vector de almacenamiento de tamaño fijo que actúa como una ventana deslizante de un segundo de duración. Cada nuevo dato recibido desde la ESP32 es convertido a formato numérico y añadido al final del vector, desplazando los valores anteriores.

La señal es graficada en tiempo real mediante la actualización dinámica de los datos en la figura. Este procedimiento permite visualizar el patrón respiratorio mientras se adquiere la señal, facilitando la verificación del correcto funcionamiento del sistema.

Posteriormente, se capturaron 30 segundos de señal en condición de reposo y 30 segundos durante verbalización, almacenando cada registro en archivos .mat para su análisis posterior.

---

## 8. Análisis en el dominio de la frecuencia

Para determinar la frecuencia respiratoria dominante se aplicó la Transformada Rápida de Fourier (FFT) a cada señal. La FFT permite representar la señal en el dominio de la frecuencia e identificar el componente espectral con mayor amplitud dentro del rango fisiológico esperado. La frecuencia dominante obtenida en Hertz fue multiplicada por 60 para expresarla en respiraciones por minuto.

---

## 9. Análisis de resultados

En condición de reposo, el patrón respiratorio presentó una forma periódica relativamente estable, con ciclos bien definidos y amplitud uniforme. La frecuencia respiratoria se mantuvo dentro de los valores normales para un adulto sano. Durante la verbalización, se observó una alteración en el patrón respiratorio caracterizada por una exhalación prolongada y una mayor variabilidad en la amplitud. Esto se debe a que la producción del habla requiere un flujo de aire continuo durante la espiración, lo cual modifica la relación temporal entre inhalación y exhalación.

Desde el punto de vista fisiológico, la respiración durante el habla deja de ser completamente automática y pasa a estar parcialmente controlada de forma voluntaria, afectando el ritmo y la duración de cada fase del ciclo respiratorio.

---

## 10. Respuestas a las preguntas de discusión (Ítem 15)

En relación con la primera pregunta, los patrones respiratorios y las frecuencias respiratorias no son iguales en reposo y durante la verbalización. En reposo, la respiración es más rítmica y simétrica, mientras que durante el habla la exhalación se prolonga para permitir la emisión de sonidos. Esto genera una modificación en la relación inhalación/exhalación y puede producir ligeras variaciones en la frecuencia respiratoria. Esta diferencia se debe a la interacción entre el sistema respiratorio y el sistema fonador, donde la espiración cumple un papel fundamental en la producción del habla.

Respecto a la segunda pregunta, el uso de múltiples sensores para el monitoreo respiratorio podría aumentar la precisión y confiabilidad del sistema, permitiendo detectar asimetrías torácicas o mejorar la relación señal-ruido. Sin embargo, también implicaría un aumento en la complejidad del sistema, mayor procesamiento de datos, incremento en el costo y posible incomodidad para el paciente. La decisión de utilizar múltiples sensores dependería del nivel de precisión requerido y de la aplicación clínica específica.

---

## 11. Conclusiones

La variación mecánica del tórax demostró ser una variable adecuada para el monitoreo experimental del proceso respiratorio. El sensor FSR permitió capturar el patrón respiratorio de manera efectiva bajo condiciones controladas. Se comprobó que la verbalización modifica significativamente el comportamiento del ciclo respiratorio, principalmente prolongando la fase espiratoria. El sistema desarrollado cumple con los objetivos planteados en la práctica y demuestra la viabilidad de implementar sistemas de adquisición simples para el análisis básico de señales fisiológicas.

---

## 12. Bibliografía

Bronzino, J. D., Biomedical Engineering Fundamentals, CRC Press.  
Fieselmann et al., Journal of General Internal Medicine.  
Subbe et al., Anaesthesia.
