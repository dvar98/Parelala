### 1. **División del trabajo en múltiples procesos (Paralelismo de datos)**
El **Modelo de Ising** utiliza una **grilla bidimensional de spins**, donde cada sitio interactúa solo con sus vecinos. Esta naturaleza local de las interacciones lo hace muy adecuado para ser paralelizado, porque podemos dividir la grilla en secciones más pequeñas (subgrillas) que pueden ser procesadas de forma **independiente** por diferentes procesos. Cada proceso actualiza los spins en su subgrilla sin interferir directamente con los demás, lo que reduce la necesidad de sincronización global.

- **Paralelo**: Cada proceso ejecuta una parte de los cálculos al mismo tiempo, lo que reduce el tiempo total de simulación en comparación con una ejecución secuencial.
- **Distribuido**: Estos procesos pueden ejecutarse en diferentes máquinas dentro de un clúster, distribuyendo la carga de trabajo entre varios nodos. Esto permite manejar simulaciones más grandes (grillas más grandes) que no cabrían en la memoria de una sola máquina.

### 2. **Comunicación entre procesos (Comunicación Distribuida)**
En este laboratorio, los procesos que trabajan en diferentes partes de la grilla deben intercambiar información sobre los **bordes de sus subgrillas** con los procesos vecinos. Esto se logra mediante **comunicación entre procesos** usando **MPI** (Message Passing Interface), un estándar para la programación distribuida.

- **Distribuida**: La comunicación entre procesos es fundamental cuando se trabaja en un entorno distribuido (varias máquinas). Los procesos no comparten memoria, por lo que deben intercambiar mensajes (enviando y recibiendo datos) para asegurarse de que los cálculos sean correctos en los bordes compartidos de las subgrillas.
  
- **Paralela**: Aunque hay comunicación, los procesos siguen ejecutando su trabajo en paralelo. Solo cuando necesitan actualizar sus fronteras, se comunican brevemente con otros procesos. El objetivo es minimizar la cantidad de comunicación para mantener la mayor parte del tiempo en cálculos locales (alta computación y baja comunicación).

### 3. **Sincronización de fronteras y coherencia global**
En una simulación secuencial, las actualizaciones de los spins en la grilla pueden hacerse de forma global. Sin embargo, en una simulación paralela, cada proceso actualiza una parte de la grilla, pero al mismo tiempo, las fronteras entre subgrillas deben ser **sincronizadas** entre procesos. Esta sincronización ocurre a través de comunicación **inter-procesos** (usando funciones como **MPI_Send** y **MPI_Recv** para intercambiar los bordes).

- **Distribuida**: Los procesos deben sincronizarse para asegurar que los cálculos sean correctos en los puntos donde sus subgrillas se encuentran.
- **Paralela**: Mientras tanto, los cálculos dentro de cada subgrilla pueden ejecutarse en paralelo sin necesidad de sincronización constante.

### 4. **Reducción y cálculo de propiedades globales (MPI_Reduce, MPI_Allreduce)**
A lo largo de la simulación, debemos calcular propiedades globales del sistema, como la **magnetización total**. Como cada proceso solo conoce la magnetización de su subgrilla, es necesario combinar los resultados de todos los procesos para obtener el valor global. Aquí es donde se usan las **operaciones de reducción** de MPI, como **MPI_Reduce** o **MPI_Allreduce**, para sumar los resultados locales en un valor global.

- **Distribuida**: Los procesos distribuyen sus resultados locales a través de mensajes para combinarlos en un valor global.
- **Paralela**: El cálculo de las propiedades locales (magnetización de la subgrilla) ocurre en paralelo antes de realizar la reducción.

### 5. **Escalabilidad (Aprovechamiento de recursos distribuidos)**
Uno de los objetivos clave de la programación paralela y distribuida es la **escalabilidad**: la capacidad de resolver problemas más grandes o hacer cálculos más rápido al aumentar los recursos disponibles (más procesos, más nodos). En este laboratorio, al dividir la grilla y distribuir el trabajo entre múltiples procesos, podemos realizar simulaciones de mayor tamaño o mayor resolución que serían imposibles en una máquina individual debido a limitaciones de memoria o tiempo de procesamiento.

- **Distribuida**: Al aprovechar múltiples máquinas o nodos en un clúster, puedes aumentar la capacidad computacional de forma escalable. Por ejemplo, en lugar de simular una grilla de 100x100 en una sola máquina, puedes simular una de 1000x1000 dividiendo la carga entre muchas máquinas.
  
### 6. **Balanceo de carga dinámico (Distribución eficiente del trabajo)**
A medida que la simulación avanza, algunos procesos podrían tardar más que otros en realizar los cálculos (por ejemplo, debido a variaciones en la complejidad local de las interacciones entre spins). Un laboratorio más complejo podría incluir **balanceo de carga dinámico**, donde se redistribuye el trabajo entre los procesos si algunos terminan antes que otros, para evitar tiempos de inactividad.

- **Distribuida**: El balanceo de carga es crucial en entornos distribuidos donde las máquinas pueden tener diferentes capacidades. Redistribuir la carga permite que el trabajo se realice de manera más eficiente.
- **Paralela**: Mientras se redistribuye el trabajo, los procesos continúan ejecutando sus cálculos en paralelo.

### Resumen:
El uso de **programación paralela** permite que el trabajo de la simulación se realice simultáneamente en varios procesadores o núcleos, reduciendo el tiempo de ejecución. La **programación distribuida** permite que este trabajo se divida entre diferentes máquinas en una red (clúster), lo cual es necesario para manejar grandes cantidades de datos o simulaciones complejas, como la del Modelo de Ising, donde la grilla completa podría no caber en la memoria de una sola máquina.

**Paralela y distribuida**:
- **Paralela**: Ejecutar cálculos en varias partes de la grilla al mismo tiempo.
- **Distribuida**: Dividir los cálculos entre varias máquinas/nodos, cada uno ejecutando una parte de la simulación y comunicándose entre sí.

Esto convierte la simulación en un excelente ejemplo de **cómputo paralelo distribuido**, donde los recursos de hardware (múltiples CPUs y nodos) se utilizan de manera eficiente para resolver un problema grande y complejo.
