# Monitoreo-del-patron-y-frecuencia-respiratoria.
Sara Damaris Vasquez Cardenas y Paula Andrea Vega Pardo


---

## INTRODUCCIÓN

En esta práctica se diseñó e implementó un sistema capaz de monitorear el proceso respiratorio de un individuo sano, capturar la señal mediante un sensor físico, digitalizarla usando un microcontrolador y posteriormente procesarla en MATLAB para obtener su frecuencia dominante en el dominio del tiempo y la frecuencia.

Se evaluó además la influencia del habla en el patrón respiratorio, comparando la señal en condición de reposo y durante lectura.

---


# PARTE A – DISEÑO DEL SISTEMA DE ADQUISICIÓN

---

##  Revisión del proceso respiratorio

La respiración es un proceso fisiológico que permite el intercambio de gases entre el organismo y el medio ambiente. Este proceso involucra la expansión y contracción de la caja torácica gracias a la acción del diafragma y los músculos intercostales.

### Variables físicas involucradas:

- Volumen pulmonar  
- Presión intratorácica  
- Flujo de aire  
- Movimiento torácico  
- Frecuencia respiratoria  

En esta práctica se decidió medir el movimiento torácico, ya que existe una relación directa entre la expansión del tórax y el ciclo respiratorio.

La frecuencia respiratoria normal en adultos sanos es:

- 12–20 respiraciones/minuto
- Equivalente a **0.2–0.33 Hz**

Durante el habla puede aumentar hasta aproximadamente **0.4–0.5 Hz**.

---

##  Selección del sensor

Se utilizó un FSR (Force Sensitive Resistor).

- Opera entre 3.3V y 5V.
- Alta sensibilidad a cambios de presión.
- Fácil integración con microcontroladores.
- Bajo costo.

El sensor fue colocado en la región torácica y asegurado mediante una banda elástica alrededor del tórax, con el objetivo de garantizar un contacto continuo y estable con la superficie corporal durante todo el proceso de medición. Esta fijación permitió minimizar interferencias externas y reducir la presencia de artefactos generados por movimientos no asociados directamente al proceso respiratorio, mejorando así la calidad y confiabilidad de la señal adquirida.

---

##  Sistema de adquisición

El sistema está compuesto por:

- Sensor FSR
- ESP32
- ADC de 12 bits (0–4095)
- Comunicación serial a 115200 baudios
- Frecuencia de muestreo: 1000 Hz


##  Código Arduino – Explicación

```cpp
const int potPin = 34;
const int fs = 1000;
const uint32_t Ts = 1000000 / fs;

uint32_t t0 = 0;

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  analogSetPinAttenuation(potPin, ADC_11db);
  t0 = micros();
}

void loop() {
  if (micros() - t0 >= Ts) {
    t0 += Ts;
    int adcValue = analogRead(potPin);
    Serial.println(adcValue);
  }
}
```


En el código se define una frecuencia de muestreo de 1000 Hz, lo que permite capturar adecuadamente las variaciones temporales de la señal respiratoria. El conversor análogo-digital se configura con una resolución de 12 bits, proporcionando valores digitales entre 0 y 4095 y asegurando una buena sensibilidad ante cambios en la señal del sensor. El control del tiempo de muestreo se realiza mediante la función `micros()`, lo que garantiza que cada muestra sea adquirida a intervalos regulares y uniformes. Finalmente, cada valor digitalizado es enviado de forma continua a través del puerto serial, permitiendo la visualización en tiempo real y el posterior procesamiento de los datos adquiridos.


---

##  Análisis del Serial Plotter

<img width="1600" height="1600" alt="image" src="https://github.com/user-attachments/assets/f48325c7-77df-4e6e-94cb-32cc365b991b" />


La señal observada en el Serial Plotter corresponde a la señal respiratoria cruda.

### Características observadas:

La señal observada presenta variaciones lentas en el tiempo, una componente DC elevada y oscilaciones de carácter periódico, características típicas de una señal respiratoria adquirida a partir del movimiento torácico. La componente DC elevada se debe principalmente a la presión inicial ejercida por la banda elástica sobre el sensor y a la posición fija del mismo sobre el tórax, lo que genera un nivel de referencia constante alrededor del cual oscila la señal.

Desde el punto de vista fisiológico, los incrementos en el valor de la señal se asocian con la fase de **inspiración**, ya que durante esta etapa el tórax se expande y aumenta la presión ejercida sobre el sensor. De manera contraria, las disminuciones en la señal corresponden a la **espiración**, cuando el tórax se contrae y la presión sobre el sensor disminuye. De este modo, cada ciclo completo de subida y bajada en la señal representa un ciclo respiratorio.

En condiciones de reposo, la señal presenta un comportamiento más estable y periódico, lo que refleja un patrón respiratorio regular controlado de forma automática por el sistema nervioso central. Por el contrario, durante el habla se observa una mayor variabilidad en la señal, con cambios más abruptos y una menor regularidad temporal, debido a la intervención del control voluntario de la respiración para permitir la fonación.

Adicionalmente, se evidencian componentes de ruido y artefactos asociados a movimientos del cuerpo, ajustes del sensor y limitaciones propias del sistema de adquisición. La presencia de estos elementos no deseados justifica la necesidad de aplicar técnicas de **filtrado digital**, con el fin de mejorar la calidad de la señal y resaltar la información respiratoria relevante.


---

# PARTE B – PROCESAMIENTO EN MATLAB

---

##  Código de adquisición en MATLAB

```matlab
clc
clear
close all

puerto   = "COM3";
baudrate = 115200;
fs       = 1000;

s = serialport(puerto, baudrate);
configureTerminator(s,"LF");
flush(s);

for k = 1:2
    respuesta = inputdlg("Ingrese el tiempo de grabación en segundos:");
    tiempo_grabacion = str2double(respuesta{1});
    N = fs * tiempo_grabacion;

    datos = zeros(N,1);
    t = (0:N-1)/fs;

    i = 1;
    while i <= N
        if s.NumBytesAvailable > 0
            valor = str2double(readline(s));
            if ~isnan(valor)
                datos(i) = valor;
                i = i + 1;
            end
        end
    end

    nombre_archivo = "senal_respiracion_" + k + ".mat";
    save(nombre_archivo,"datos","t","fs")
end

clear s
```

Este código permite realizar una **captura temporizada de la señal respiratoria**, ya que el número total de muestras se determina a partir del tiempo de grabación ingresado por el usuario y la frecuencia de muestreo establecida. De esta manera, se garantiza que cada adquisición tenga una duración específica y controlada, lo que facilita posteriormente la comparación entre distintas condiciones experimentales.

Además, el programa guarda automáticamente los datos adquiridos en archivos con extensión `.mat`, los cuales incluyen el vector de muestras, el vector de tiempo y la frecuencia de muestreo. Este formato permite conservar la información de manera organizada y lista para su posterior procesamiento en MATLAB, sin pérdida de datos ni necesidad de conversiones adicionales.

Finalmente, el uso de un ciclo que realiza dos grabaciones consecutivas posibilita la **separación de condiciones experimentales**, como reposo y habla. Esto facilita el análisis comparativo entre ambos estados fisiológicos, permitiendo evaluar cómo varía la señal respiratoria bajo diferentes situaciones.

---

##  Reducción de frecuencia (Decimación)

```matlab
fsn = 20;
k = round(fs/fsn);
dr = decimate(datos, k);
fsr = fs/k;
```

La frecuencia de muestreo de la señal se reduce de 1000 Hz a 20 Hz mediante un proceso de decimación, con el fin de adecuar la tasa de muestreo al contenido espectral real de la señal respiratoria. Dado que la respiración es un fenómeno de baja frecuencia, generalmente inferior a 1 Hz en individuos sanos, una frecuencia de muestreo de 20 Hz resulta más que suficiente para capturar correctamente la dinámica del proceso respiratorio sin pérdida de información relevante, cumpliendo además con el criterio de Nyquist.

Esta reducción en la frecuencia de muestreo permite disminuir la cantidad de datos a procesar, optimizando el tiempo de cálculo y el uso de recursos computacionales, sin comprometer la calidad del análisis. Asimismo, al trabajar con una frecuencia más acorde con la señal de interés, se facilita el diseño de filtros digitales y la interpretación de los resultados en el dominio del tiempo y la frecuencia.


---

##  Filtrado digital

Filtro Butterworth pasa banda:

- Frecuencia baja: 0.1 Hz
- Frecuencia alta: 0.6 Hz
- Orden: 3

```matlab
[b,a] = butter(3,[0.1 0.6]/(fsr/2),'bandpass');
df = filtfilt(b,a,dr);
```

Se usa `filtfilt` para evitar desfase.

El filtro diseñado corresponde a un filtro pasa banda Butterworth que permite aislar únicamente la información asociada al proceso respiratorio. En primer lugar, elimina la componente DC presente en la señal, la cual está relacionada con el nivel de presión constante ejercido por la banda elástica sobre el sensor y no contiene información dinámica relevante del ciclo respiratorio. En segundo lugar, atenúa el ruido de alta frecuencia, que puede deberse a interferencias electrónicas, pequeñas vibraciones o movimientos corporales no relacionados directamente con la respiración. Finalmente, el filtro conserva únicamente la banda de interés comprendida entre 0.1 Hz y 0.6 Hz, rango en el cual se encuentra la frecuencia respiratoria normal de un individuo sano. De esta manera, se obtiene una señal más limpia, estable y representativa del patrón respiratorio real, facilitando su análisis en el dominio del tiempo y de la frecuencia.

### Señal en reposo con filtro
<img width="991" height="630" alt="image" src="https://github.com/user-attachments/assets/55bbc040-06c6-41f1-9b1c-f5e40fea7909" />

### Señal hablando con filtro
<img width="991" height="630" alt="image" src="https://github.com/user-attachments/assets/c9d7960d-483b-4b3e-8a96-fe21ea7896b0" />


---

##  Análisis espectral (FFT)

```matlab
L = length(df);
f = (0:L-1)*(fsr/L);

Ff = abs(fft(df));
Ff = Ff(1:floor(L/2));
f = f(1:floor(L/2));

[~,ifil] = max(Ff);
ffd = f(ifil);
```

---

# RESULTADOS

## Señal en reposo

<img width="1002" height="669" alt="image" src="https://github.com/user-attachments/assets/5a74ed31-5c98-427f-b84b-b37adcc58a0b" />


Frecuencia dominante: 0.25 Hz

Conversión:

0.25 × 60 = **15 respiraciones/min**

La señal en condición de reposo presenta un patrón estable y periódico, lo que indica que el proceso respiratorio se desarrolla de manera regular y sin alteraciones externas significativas. Las oscilaciones son uniformes en el tiempo, reflejando una frecuencia respiratoria constante y controlada automáticamente por el sistema nervioso central.

Además, se observa una baja variabilidad en la amplitud y en la duración de los ciclos respiratorios, lo que es característico de un individuo sano en estado de reposo. Este comportamiento confirma que la respiración es rítmica y mantiene una relación equilibrada entre inspiración y espiración.


---

## Señal durante habla

<img width="987" height="669" alt="image" src="https://github.com/user-attachments/assets/a2b9bbf6-1ab1-4df3-a69a-d5066b264568" />

Frecuencia dominante: 0.375 Hz

Conversión:

0.375 × 60 = **22.5 respiraciones/min**

La señal respiratoria durante la condición de habla presenta una mayor frecuencia, lo que indica un incremento en el número de ciclos respiratorios por unidad de tiempo. Este aumento está asociado a la necesidad de coordinar la respiración con la fonación, lo que modifica el ritmo respiratorio normal.

Asimismo, se observa una mayor variabilidad y un patrón menos periódico en comparación con la señal en reposo. Esto se debe a la intervención del control voluntario de la respiración durante el habla, lo que genera irregularidades en la duración y amplitud de los ciclos respiratorios, reflejando un patrón más complejo y menos estable.


---

# ANÁLISIS DE LAS GRÁFICAS

### Dominio del tiempo:

- Reposo → señal periódica.
- Habla → mayor irregularidad.
En el dominio del tiempo se observa que, en condición de reposo, la señal respiratoria presenta un comportamiento claramente periódico, con ciclos bien definidos y repetitivos en el tiempo. En contraste, durante el habla la señal muestra una mayor irregularidad, evidenciando variaciones en la amplitud y en la duración de los ciclos respiratorios, producto de la coordinación entre la respiración y la fonación.


### Dominio de la frecuencia:

- Energía concentrada < 1 Hz.
- Pico dominante claro en cada condición.
- Confirmación del correcto filtrado.
En el dominio de la frecuencia se observa que la mayor parte de la energía de la señal se encuentra concentrada por debajo de 1 Hz, lo cual es coherente con la naturaleza de baja frecuencia del proceso respiratorio. Además, se identifica un pico dominante claramente definido en cada condición analizada, correspondiente a la frecuencia respiratoria principal. La presencia de estos picos bien delimitados confirma que el filtrado aplicado fue adecuado, ya que permitió eliminar componentes no deseadas y resaltar de manera efectiva la información respiratoria relevante.

---

# INTERPRETACIÓN FISIOLÓGICA

En condición de reposo, la respiración está regulada principalmente por el centro respiratorio bulbar, el cual funciona de manera automática e involuntaria. Este control genera un ritmo respiratorio estable y periódico, manteniendo una frecuencia constante acorde con las necesidades metabólicas del organismo.

Durante el habla, además del control automático, interviene el control voluntario de la respiración para permitir la producción de la voz. En este caso, la espiración suele prolongarse para sostener la fonación y, como consecuencia, puede aumentar la frecuencia respiratoria y alterarse la regularidad del patrón respiratorio.


---

#  ANÁLISIS DE RESULTADOS

---

##  Análisis 1: Comparación entre reposo y verbalización

Al comparar la señal respiratoria de un individuo sano en condiciones de reposo y durante tareas de verbalización, se evidencian diferencias claras tanto en la frecuencia respiratoria como en la relación entre inhalaciones y exhalaciones.

En condición de reposo, la señal presenta un patrón más periódico y estable, con una frecuencia dominante cercana a **0.25 Hz**, equivalente a aproximadamente **15 respiraciones por minuto**, valor que se encuentra dentro del rango fisiológico normal para un adulto sano. En este estado, la relación entre inhalación y exhalación es relativamente equilibrada, ya que el control respiratorio es predominantemente automático y regulado por el centro respiratorio bulbar.

Por el contrario, durante tareas de verbalización o lectura, la señal respiratoria muestra un aumento en la frecuencia dominante hasta valores cercanos a **0.375 Hz**, equivalentes a aproximadamente **22.5 respiraciones por minuto**. Además, el patrón respiratorio se vuelve menos regular, evidenciando una modificación en la relación inhalación/espiración. En este caso, la espiración tiende a prolongarse debido a la fonación, mientras que las inhalaciones se vuelven más cortas y frecuentes.

Estas diferencias se deben a la intervención del control voluntario de la respiración durante el habla, el cual modifica el patrón respiratorio normal para permitir la producción de la voz, afectando tanto la frecuencia como la morfología de la señal respiratoria.

---

## Análisis 2: Alcance y limitaciones del sistema propuesto

El sistema desarrollado durante la práctica demuestra ser eficaz para el monitoreo básico del proceso respiratorio y la estimación de la frecuencia respiratoria en individuos sanos. Entre los principales alcances del sistema se encuentran:

- Capacidad de detectar cambios en la frecuencia respiratoria.
- Sensibilidad suficiente para identificar diferencias entre estados fisiológicos (reposo vs verbalización).
- Bajo costo y fácil implementación.
- Posibilidad de análisis en tiempo real y posterior procesamiento digital.

Sin embargo, el sistema también presenta algunas limitaciones importantes al considerar su uso para la detección de patologías respiratorias:

- El sensor FSR mide de forma indirecta la respiración a través del movimiento torácico, lo cual puede verse afectado por movimientos corporales ajenos al proceso respiratorio.
- No permite medir directamente variables como el flujo de aire o el volumen pulmonar.
- La señal puede verse afectada por la colocación del sensor, la tensión de la banda y la morfología del sujeto.
- No permite diferenciar de manera precisa entre distintos tipos de patologías respiratorias, sino únicamente detectar alteraciones generales en el patrón respiratorio.

Por lo tanto, aunque el sistema es útil como herramienta educativa y de monitoreo básico, su aplicación clínica requeriría el uso de sensores adicionales y técnicas de procesamiento más avanzadas.

---

#  CONCLUSIONES

Al finalizar la práctica se concluye que el sistema implementado permite capturar de manera adecuada la señal respiratoria y analizar sus características tanto en el dominio del tiempo como en el dominio de la frecuencia. Se logró evidenciar que variables físicas como el **movimiento torácico**, la **frecuencia respiratoria** y la **regularidad del patrón respiratorio** son especialmente relevantes para el monitoreo del proceso respiratorio.

De estas variables, la frecuencia respiratoria y la morfología de la señal resultan ser las más adecuadas para detectar posibles anomalías respiratorias, ya que cambios en estas pueden estar asociados a estados patológicos o a alteraciones en el control respiratorio. Asimismo, el análisis en frecuencia permitió cuantificar objetivamente estos cambios, facilitando la comparación entre distintas condiciones fisiológicas.

---

#  PREGUNTAS PARA LA DISCUSIÓN

---

## Pregunta 1  
**¿Son los patrones respiratorios y frecuencias respiratorias iguales o diferentes en cada caso? ¿A qué se debe esto?**

Los patrones respiratorios y las frecuencias respiratorias son diferentes entre las condiciones de reposo y verbalización. En reposo, la respiración es más regular y presenta una frecuencia menor, mientras que durante el habla la frecuencia respiratoria aumenta y el patrón se vuelve menos periódico.

Esto se debe a que durante la verbalización interviene el control voluntario de la respiración, el cual modifica la relación entre inhalación y espiración para permitir la producción de la voz. En cambio, en reposo, la respiración es controlada principalmente de forma automática por el sistema nervioso central.

---

## Pregunta 2  
**¿Cuáles serían las ventajas y desventajas de emplear múltiples sensores para el monitoreo del proceso respiratorio? ¿Cuáles podrían ser las razones?**

El uso de múltiples sensores para el monitoreo del proceso respiratorio presenta varias ventajas, entre ellas:

- Mayor precisión en la medición de distintas variables fisiológicas.
- Posibilidad de medir simultáneamente movimiento torácico, flujo de aire y volumen pulmonar.
- Reducción del impacto de artefactos y ruido.
- Mejor caracterización de posibles patologías respiratorias.

No obstante, también presenta desventajas:

- Mayor complejidad del sistema.
- Incremento en el costo.
- Mayor incomodidad para el sujeto de prueba.
- Mayor complejidad en el procesamiento y sincronización de las señales.

Las razones para emplear múltiples sensores estarían asociadas principalmente a aplicaciones clínicas o diagnósticas, donde se requiere una caracterización más completa y precisa del proceso respiratorio.


