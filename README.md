# Cierre bancario con Spring Batch

**Autor:** José Rodrigo Benítez Rivera

## Cómo correrlo

```
docker compose up -d --wait
./correr.sh 2026-09-30 prueba
./ver-batch.sh
```

## Día 1 · Mi primer Job

### Boleto de salida

1. **¿Qué diferencia hay entre un proceso batch y la API REST de la Semana 3? Da dos.**

   Un proceso batch ejecuta tareas de forma programada o por lotes con el fin de procesas grandes cantidades de datos de forma automática y divisible para no sobrecargar un servidor. Por lotro lado una API REST responde a peticiones enviadas por los usuarios o por otros sistemas en tiempo real.

2. **¿Qué es un Job, qué es un Step y qué es un Tasklet?**

   Un Job representa el trabajo completo que se quiere ejecutar.

   Un Step es un paso dentro de un Job.

   Un Tasklet es una tarea concreta que se ejecuta dentro de un Step y que realiza una acción específica.

3. **¿Con tus tablas: qué diferencia hay entre una JobInstance y una JobExecution?**

   Una JobInstance representa una ejecución lógica de un Job y se identifica por el nombre del Job y sus parámetros, por ejemplo, la fecha del cierre bancario. Si se vuelve a utilizar la misma fecha y los mismos parámetros, se trata de la misma JobInstance.

   Una JobExecution representa un intento concreto de ejecutar esa JobInstance. Una misma JobInstance puede tener varias JobExecution si una ejecución falla y posteriormente se vuelve a intentar. En las tablas de Spring Batch se pueden observar estos intentos y su estado de completado.


4. **¿Por qué Spring Batch no deja correr dos veces el cierre del 28?**

   Porque el cierre del día 28 tiene los mismos parámetros y, por lo tanto, corresponde a la misma JobInstance. 

5. **(MP-4, paso 6) Si mañana llega el archivo del 25 y corres otra vez el cierre del 25, ¿será otra instancia u otra ejecución de la misma? ¿Por qué lo crees?**

   Será otra JobExecution de la misma JobInstance, pero solo si se utilicen exactamente los mismos parámetros del cierre del día 25. La JobInstance se identifica por el nombre del Job y sus parámetros, mientras que cada intento de ejecución genera una JobExecution diferente.
