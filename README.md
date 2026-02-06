
---

# APE 9 – Procesamiento Paralelo en Clúster Local (Simulado)

**Asignatura:** Arquitectura de Ordenadores

**Unidad 3:** CPU, GPU y dispositivos de E/S

## 1. Introducción y Objetivos

El presente experimento tiene como propósito evaluar la eficiencia del procesamiento paralelo frente al secuencial en tareas de cálculo intensivo (**CPU-bound**). Se busca cuantificar el impacto del **overhead** de coordinación en el modelo de multiprocesamiento y determinar el **Speedup** alcanzado en función de la arquitectura del procesador local.

## 2. Identificación de Recursos de Hardware

Determinación de la topología del procesador para establecer el límite teórico de paralelismo (núcleos lógicos).

```python
import multiprocessing
import os
import time
import math

n_nucleos = multiprocessing.cpu_count()
print(f"Núcleos lógicos (hilos) disponibles: {n_nucleos}")
print(f"Sistema Operativo: {os.name}")

```

## 3. Definición de la Carga de Trabajo ()

La variable **N** define la magnitud del volumen de datos a procesar (iteraciones). En esta práctica, el valor de  debe ajustarse proporcionalmente a la capacidad de cómputo del hardware: un  elevado permite que el tiempo de ejecución útil supere los costos de latencia por orquestación de procesos.

```python
# Magnitud de la carga de trabajo (ajustada para stress-testing)
N = 10_000_000 

def tarea_intensiva(i: int) -> float:
    """Simulación de tarea CPU-bound utilizando la FPU (Floating Point Unit)."""
    return math.sqrt(i) * math.sin(i) + math.cos(i)

```

## 4. Ejecución Secuencial

Establecimiento de la línea base (baseline) de rendimiento mediante una ejecución de hilo único.

```python
print(f"Iniciando fase secuencial (N = {N})...")
t0 = time.perf_counter()
resultados_seq = [tarea_intensiva(i) for i in range(1, N + 1)]
t1 = time.perf_counter()

tiempo_secuencial = t1 - t0
print(f"Tiempo de ejecución secuencial: {tiempo_secuencial:.4f} s")

```

## 5. Ejecución Paralela (Simulación de Clúster)

Implementación de un modelo de multiprocesamiento para mitigar las limitaciones del **Global Interpreter Lock (GIL)** de Python, distribuyendo la carga de trabajo entre los núcleos lógicos detectados.

```python
from multiprocessing import Pool

def ejecutar_paralelo(n: int, num_procesos: int):
    with Pool(processes=num_procesos) as pool:
        return pool.map(tarea_intensiva, range(1, n + 1))

if __name__ == "__main__":
    procesos = multiprocessing.cpu_count()
    print(f"Lanzando ejecución paralela con {procesos} procesos...")
    
    t0 = time.perf_counter()
    resultados_par = ejecutar_paralelo(N, num_procesos=procesos)
    t1 = time.perf_counter()

    tiempo_paralelo = t1 - t0
    print(f"Tiempo de ejecución paralela: {tiempo_paralelo:.4f} s")

```

## 6. Evaluación de Rendimiento y Speedup

Cálculo de la aceleración relativa mediante la métrica de Speedup y análisis de eficiencia.

```python
speedup = tiempo_secuencial / tiempo_paralelo
eficiencia = (speedup / procesos) * 100

print(f"Speedup calculado: {speedup:.2f}x")
print(f"Eficiencia de paralelización: {eficiencia:.2f}%")

```

## 7. Análisis Técnico y Conclusiones

1. **¿Por qué el procesamiento paralelo puede reducir el tiempo de ejecución?** El paralelismo explota la capacidad multi-núcleo del procesador mediante el paralelismo a nivel de proceso (PLP). Al particionar la carga de trabajo  en subconjuntos independientes que se ejecutan simultáneamente en distintas unidades de ejecución, el tiempo de procesamiento total se reduce de forma inversamente proporcional a la disponibilidad de recursos físicos, optimizando el rendimiento de la CPU.
2. **¿Qué ocurre con el overhead (costos de coordinación) y cómo afecta a los resultados?** El *overhead* comprende la latencia introducida por el sistema operativo y el intérprete al gestionar la creación de procesos hijos, la serialización de datos (*pickling*) y la comunicación inter-procesos (IPC). Si la granularidad de la tarea es insuficiente (valor de  reducido), el costo temporal de esta gestión administrativa supera la ganancia de velocidad, resultando en un Speedup ineficiente o inferior a 1.
3. **¿Qué factores de hardware influyen (núcleos, caché, memoria RAM)?** La arquitectura de la jerarquía de memoria (cachés L1/L2/L3) influye en la latencia de acceso a datos de cada hilo. Asimismo, el ancho de banda de la memoria RAM puede convertirse en un cuello de botella si múltiples núcleos realizan peticiones simultáneas (contención de memoria). En sistemas con gráficos integrados, la memoria compartida añade una capa de latencia adicional que penaliza el paralelismo masivo.
4. **¿Cuándo NO conviene paralelizar?** La paralelización es contraproducente en algoritmos con alta dependencia de datos secuenciales, en tareas dominadas por operaciones de Entrada/Salida (I/O-bound) donde la CPU permanece en estado de espera, o cuando la carga de cómputo es tan trivial que no justifica el consumo de recursos requerido para orquestar múltiples procesos.
---

Para completar tu informe utilizando Google Colab, seguiremos un procedimiento técnico que te permita obtener datos reales y comparables. Ten en cuenta que, por defecto, las instancias gratuitas de Google Colab suelen asignar 2 núcleos, lo que es ideal para demostrar por qué en entornos limitados el paralelismo a veces es menos eficiente (debido al overhead).

Aquí tienes el paso a paso detallado para realizar tu prueba y obtener los valores para la tabla:

Paso 1: Preparación en Google Colab
Entra a Google Colab.

Crea un "Nuevo cuaderno".

En la esquina superior derecha, asegúrate de estar conectado a un entorno de ejecución (debe decir "RAM" y "Disco").

Paso 2: Ejecución de las Pruebas
Copia y ejecuta las siguientes celdas de código una por una para obtener tus resultados.

1. Identificación del Hardware (Para la columna "CPU / Núcleos")
Python
import multiprocessing
n_nucleos = multiprocessing.cpu_count()
print(f"Número de núcleos detectados: {n_nucleos}")
Valor a anotar: El número que aparezca (probablemente 2).

2. Definición de la Tarea y Valor de N (Para el campo "N =")
Usaremos el valor estándar de tu práctica previa.

Python
import math
import time
from multiprocessing import Pool

N = 2_000_000 # Define este valor en tu informe

def tarea(i):
    return math.sqrt(i) * math.sin(i) + math.cos(i)

Valor a anotar: En tu informe, en el campo N =, escribe 2,000,000.

3. Prueba Secuencial (Para la columna "Tiempo Secuencial (s)")
Python
t0 = time.perf_counter()
resultados_seq = [tarea(i) for i in range(1, N + 1)]
t1 = time.perf_counter()
tiempo_seq = t1 - t0
print(f"Tiempo Secuencial: {tiempo_seq:.4f} s")
4. Prueba Paralela (Para la columna "Tiempo Paralelo (s)")
Python
if __name__ == "__main__":
    t0 = time.perf_counter()
    with Pool(processes=n_nucleos) as pool:
        pool.map(tarea, range(1, N + 1))
    t1 = time.perf_counter()
    tiempo_par = t1 - t0
    print(f"Tiempo Paralelo: {tiempo_par:.4f} s")
5. Cálculo de Speedup (Para la columna "Speedup (x)")
Python
speedup = tiempo_seq / tiempo_par
print(f"Speedup: {speedup:.2f}x")
Paso 3: Llenado del Informe
Una vez tengas los resultados en pantalla, completa tu fila en la tabla del Análisis Comparativo Grupal.

Ejemplo de cómo quedarían tus datos (basado en una ejecución estándar de Colab):

Integrante: Tu nombre.

CPU / Núcleos: 2 (o lo que detectó el código).

Tiempo Secuencial (s): El valor de la celda 3.

Tiempo Paralelo (s): El valor de la celda 4.

Speedup (x): El valor de la celda 5.

Recomendación para el Análisis del Informe
Como verás en los datos de tus compañeros (como el caso de Alejandro que obtuvo un 0.40x con 12 núcleos), en tareas con un N relativamente bajo o en entornos con pocos núcleos (Colab), es normal que el tiempo paralelo sea mayor que el secuencial.

Esto sucede por el Overhead de coordinación: el tiempo que el sistema operativo pierde creando procesos y repartiendo datos en Google Colab es mayor que el tiempo de cálculo real, ya que solo hay 2 núcleos para procesar. Puedes usar esta observación técnica para completar las "Respuestas Generales al Cuestionario".
