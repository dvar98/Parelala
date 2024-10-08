### Laboratorio Complejo: **Simulación del Modelo de Ising 2D usando MPI**

#### Objetivo:
Implementar una simulación del **Modelo de Ising 2D** para estudiar el comportamiento de un material magnético a diferentes temperaturas. El laboratorio se enfocará en la paralelización de la simulación en múltiples nodos utilizando MPI, distribuyendo la grilla de simulación entre los diferentes procesos, y resolviendo problemas de **sincronización**, **comunicación intensiva** y **balanceo de carga**.

#### Descripción del problema:
El **Modelo de Ising** es una representación matemática del magnetismo en materiales, donde una grilla bidimensional representa un conjunto de átomos o partículas, y cada sitio de la grilla tiene un "spin" que puede estar en uno de dos estados (+1 o -1). La simulación evoluciona iterativamente, ajustando los spins en función de la interacción con sus vecinos.

El laboratorio debe abordar la paralelización de este modelo dividiendo la grilla entre diferentes procesos y coordinando la simulación a través de mensajes entre procesos para garantizar la coherencia de los bordes entre subgrillas.

### Características Avanzadas:
1. **Sincronización de fronteras**: Cada proceso tiene una subgrilla del total de la simulación. Al actualizar los estados de los spins, los procesos deben intercambiar información con sus vecinos para actualizar los bordes.
2. **Distribución dinámica**: La grilla se redistribuye si alguna máquina es más rápida, para optimizar el rendimiento.
3. **Medición del magnetismo global**: A medida que el sistema evoluciona, se deben calcular propiedades globales como la magnetización total, lo cual implica una reducción y comunicación masiva.
4. **Variación de la temperatura**: El laboratorio incluirá diferentes configuraciones de temperatura para observar cómo afecta a las transiciones de fase magnética.

### Código en C usando MPI

```c
#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <time.h>

#define L 100  // Tamaño de la grilla
#define J 1.0  // Interacción de los spins
#define STEPS 1000  // Número de pasos de la simulación
#define TEMPERATURE 2.0  // Temperatura del sistema

// Función para inicializar la grilla con spins aleatorios (+1 o -1)
void initialize_grid(int grid[L][L]) {
    for (int i = 0; i < L; i++) {
        for (int j = 0; j < L; j++) {
            grid[i][j] = (rand() % 2) ? 1 : -1;
        }
    }
}

// Función que calcula la energía de interacción de un spin con sus vecinos
double calculate_energy(int grid[L][L], int x, int y) {
    int up = (x == 0) ? L - 1 : x - 1;
    int down = (x == L - 1) ? 0 : x + 1;
    int left = (y == 0) ? L - 1 : y - 1;
    int right = (y == L - 1) ? 0 : y + 1;
    
    return -J * grid[x][y] * (grid[up][y] + grid[down][y] + grid[x][left] + grid[x][right]);
}

// Actualización de spins usando el algoritmo de Metropolis
void metropolis_step(int grid[L][L], double temperature) {
    for (int i = 0; i < L; i++) {
        for (int j = 0; j < L; j++) {
            int x = rand() % L;
            int y = rand() % L;
            double delta_E = -2 * calculate_energy(grid, x, y);
            if (delta_E < 0 || (rand() / (double)RAND_MAX) < exp(-delta_E / temperature)) {
                grid[x][y] *= -1;
            }
        }
    }
}

// Función para calcular la magnetización total del sistema
double calculate_magnetization(int grid[L][L]) {
    double magnetization = 0;
    for (int i = 0; i < L; i++) {
        for (int j = 0; j < L; j++) {
            magnetization += grid[i][j];
        }
    }
    return magnetization;
}

// Intercambio de fronteras entre procesos vecinos
void exchange_borders(int grid[L][L], int rank, int size, MPI_Comm comm) {
    // Fronteras de los procesos para intercambio
    // Implementar código de MPI_Send/MPI_Recv o MPI_Isend/MPI_Irecv para intercambio de fronteras
}

int main(int argc, char** argv) {
    int rank, size;
    int grid[L][L];  // Grilla de spins
    double global_magnetization = 0;
    MPI_Init(&argc, &argv);  // Inicializa MPI
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    srand(time(NULL) + rank);  // Inicializar aleatoriamente para cada proceso

    // Cada proceso inicializa su parte de la grilla
    initialize_grid(grid);

    for (int step = 0; step < STEPS; step++) {
        metropolis_step(grid, TEMPERATURE);

        // Intercambio de fronteras entre los procesos
        exchange_borders(grid, rank, size, MPI_COMM_WORLD);

        // Calcular la magnetización local
        double local_magnetization = calculate_magnetization(grid);

        // Sumar todas las magnetizaciones locales
        MPI_Allreduce(&local_magnetization, &global_magnetization, 1, MPI_DOUBLE, MPI_SUM, MPI_COMM_WORLD);

        if (rank == 0 && step % 100 == 0) {
            printf("Paso: %d, Magnetización global: %f\n", step, global_magnetization / (L * L));
        }
    }

    MPI_Finalize();
    return 0;
}
```

### Explicación del código:
1. **Paralelización del modelo de Ising**:
   - Cada proceso simula una porción de la grilla de spins. La grilla se divide entre los diferentes procesos en filas o columnas.
   - Se usa el algoritmo de **Metropolis** para actualizar los spins en función de la energía local y la temperatura.
   - En cada paso, los procesos intercambian los valores de los spins en las fronteras con sus procesos vecinos para garantizar la coherencia de los bordes.
   
2. **Intercambio de fronteras**:
   - Se utiliza **MPI_Send** y **MPI_Recv** (o sus versiones no bloqueantes **MPI_Isend** y **MPI_Irecv**) para intercambiar las filas o columnas de los bordes de la grilla entre procesos vecinos.

3. **Cálculo de la magnetización**:
   - Cada proceso calcula la magnetización de su subgrilla local.
   - Usamos **MPI_Allreduce** para sumar las magnetizaciones locales y obtener la magnetización global, que se imprime en el proceso maestro.

4. **Sincronización**:
   - A lo largo de la simulación, los procesos necesitan sincronizarse en cada paso para intercambiar fronteras y calcular propiedades globales como la magnetización.

### Complejidad adicional:
1. **Optimización del intercambio de bordes**: Implementar versiones más avanzadas de comunicación con buffers o usando **MPI_Sendrecv** para minimizar el overhead de comunicación.
2. **Balanceo dinámico de la carga**: Redistribuir la grilla si algunos procesos terminan antes que otros.
3. **Escalabilidad**: Ejecutar el laboratorio con un mayor número de procesos (ejemplo: dividiendo la grilla en subgrillas más pequeñas) y analizar el rendimiento.
4. **Estudio de transición de fase**: Añadir análisis del comportamiento del sistema a diferentes temperaturas para observar el cambio de magnetización, lo cual requiere simulaciones adicionales y análisis más detallados.

### Instrucciones para ejecución:
1. **Compilar el código con MPI**:
   ```bash
   mpicc -o ising_model ising_model.c
   ```

2. **Ejecutar el programa en varias máquinas**:
   ```bash
   mpirun -np 4 --hostfile hosts ./ising_model
   ```

Este laboratorio es significativamente más complejo, ya que involucra simulaciones físicas, intercambio de fronteras entre procesos, cálculos globales a través de reducciones y la posibilidad de analizar un fenómeno físico interesante como la transición de fase en materiales magnéticos.
