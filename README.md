# Monitoreo-del-patron-y-frecuencia-respiratoria.
Sara Damaris Vasquez Cardenas y Paula Andrea Vega Pardo

---

#  LABORATORIO 1 

---

# INTRODUCCIÓN

En esta práctica se diseñó e implementó un sistema capaz de monitorear el proceso respiratorio de un individuo sano, capturar la señal mediante un sensor físico, digitalizarla usando un microcontrolador y posteriormente procesarla en MATLAB para obtener su frecuencia dominante en el dominio del tiempo y la frecuencia.

Se evaluó además la influencia del habla en el patrón respiratorio, comparando la señal en condición de reposo y durante lectura.

---

# OBJETIVOS DE LA PRÁCTICA

Al finalizar esta práctica se logró:

1. Identificar las variables físicas involucradas en el proceso respiratorio.
2. Diseñar un sistema capaz de monitorear la respiración.
3. Capturar y manipular vectores en MATLAB.
4. Evaluar la influencia del habla en la frecuencia respiratoria.
5. Representar la señal en el dominio del tiempo y de la frecuencia.
6. Interpretar fisiológicamente los resultados obtenidos.
7. Documentar el proceso en GitHub.

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

En esta práctica se decidió medir el **movimiento torácico**, ya que existe una relación directa entre la expansión del tórax y el ciclo respiratorio.

La frecuencia respiratoria normal en adultos sanos es:

- **12–20 respiraciones/minuto**
- Equivalente a **0.2–0.33 Hz**

Durante el habla puede aumentar hasta aproximadamente **0.4–0.5 Hz**.

---

##  Selección del sensor

Se utilizó un FSR (Force Sensitive Resistor).

- Opera entre 3.3V y 5V.
- Alta sensibilidad a cambios de presión.
- Fácil integración con microcontroladores.
- Bajo costo.

El sensor fue colocado en la región torácica y asegurado con una banda elástica para:

- Garantizar contacto continuo.
- Minimizar interferencias.
- Reducir artefactos por movimiento.



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

### Explicación:

- Se define una frecuencia de muestreo de 1000 Hz.
- Se configura el ADC en resolución de 12 bits.
- Se controla el tiempo de muestreo usando `micros()`.
- Cada muestra es enviada por puerto serial.

Esto garantiza muestreo uniforme y transmisión continua de datos.

---

##  Análisis del Serial Plotter

La señal observada en el Serial Plotter corresponde a la señal respiratoria cruda.

### Características observadas:

- Variaciones lentas.
- Componente DC elevada.
- Oscilaciones periódicas.

Lo que nos indica que:
- Subida → Inspiración.
- Bajada → Espiración.

En reposo:
- Señal más estable y periódica.

Durante habla:
- Mayor variabilidad.
- Cambios más abruptos.

Se evidenció ruido y artefactos, justificando el uso de filtrado digital.

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

Este código permite:

- Captura temporizada.
- Almacenamiento en archivos `.mat`.
- Separación de condiciones (reposo y habla).

---

##  Reducción de frecuencia (Decimación)

```matlab
fsn = 20;
k = round(fs/fsn);
dr = decimate(datos, k);
fsr = fs/k;
```

Se reduce la frecuencia de 1000 Hz a 20 Hz, suficiente para señales respiratorias (<1 Hz).

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

Este filtro:

- Elimina componente DC.
- Elimina ruido de alta frecuencia.
- Conserva banda respiratoria.

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

Frecuencia dominante: 0.25 Hz

Conversión:

0.25 × 60 = **15 respiraciones/min**

Características:

- Patrón estable.
- Oscilaciones regulares.
- Baja variabilidad.

---

## Señal durante habla

Frecuencia dominante: 0.375 Hz

Conversión:

0.375 × 60 = **22.5 respiraciones/min**

Características:

- Mayor frecuencia.
- Mayor variabilidad.
- Patrón menos periódico.

---

# ANÁLISIS DE LAS GRÁFICAS

### Dominio del tiempo:

- Reposo → señal periódica.
- Habla → mayor irregularidad.

### Dominio de la frecuencia:

- Energía concentrada < 1 Hz.
- Pico dominante claro en cada condición.
- Confirmación del correcto filtrado.

---

# INTERPRETACIÓN FISIOLÓGICA

En reposo:
- Control automático del centro respiratorio bulbar.
- Ritmo estable.

Durante el habla:
- Interviene control voluntario.
- Se prolonga la espiración.
- Aumenta la frecuencia respiratoria.

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


