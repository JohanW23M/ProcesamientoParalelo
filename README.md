
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
