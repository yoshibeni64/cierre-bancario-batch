## Día 3 · Parámetros, fallas y reinicio

### Boleto de salida

**1. ¿Qué diferencia hay entre una JobInstance y una JobExecution? Usa como ejemplo el cierre del 25.**

Una JobInstance representa un cierre que se define mediante parámetros, en este caso la fecha 2026-12-25. En cambio una JobExecution representa un intento concreto de ejecutar esa instancia. En el cierre del 25 hubo una misma JobInstance con dos JobExecution: la primera falló porque no existía el archivo y la segunda terminó correctamente cuando el archivo llegó.

**2. ¿En qué caso Spring Batch se niega a correr un cierre, y en qué caso lo reinicia?**

Spring Batch se niega cuando se intenta ejecutar nuevamente una JobInstance que ya terminó correctamente, indicado por la palabra COMPLETED. Si la instancia anterior terminó en FAILED, permite otra JobExecution de la misma instancia, es decir, un reinicio.

**3. En el reinicio del día 5, ¿por qué el step de carga leyó 10 movimientos y no 20?**

Porque Spring Batch guarda el estado en el que se encuentra la ejecución anterior y permite reanudar el proceso. Los movimientos que ya habían sido procesados no se vuelven a leer, solo se procesan los movimientos pendientes.

**4. ¿Qué diferencia hay entre un movimiento filtrado y uno omitido?**

Un movimiento filtrado es leído correctamente, pero el procesador decide que no debe ejecutar el Writer, por lo que no se considera un error. Un movimiento omitido skipped ocurre cuando hay una excepción que Spring Batch está configurado para tolerar y registrar como un error que se pueda omitir.

**5. ¿Por qué importa el código de salida, si el estado ya queda en las tablas?**

Porque el código de salida EXIT_CODE comunica el resultado de la ejecución hacia fuera de Spring Batch, por ejemplo a un script, sistema operativo o herramienta que lanzó el proceso. El estado STATUS sirve para conocer el estado dentro de Spring Batch, mientras que el código de salida permite saber si el proceso terminó correctamente o no.
## Día 2 · El primer chunk

### Boleto de salida

**1. ¿Qué diferencia hay entre un step de tipo Tasklet y uno de tipo chunk?**

Un Tasklet realiza una tarea completa, mientras que un chunk procesa los datos en grupos, leyendo, procesando y escribiendo cierta cantidad de elementos por transacción.

**2. ¿Qué hace cada una de las tres piezas de un chunk? ¿Cuál es opcional?**

El Reader lee los datos, el Processor es opcional y los transforma o valida y el Writer escribe los datos procesados. .

**3. Con 45 movimientos y chunks de 10, ¿cuántos commits habría? ¿Y con chunks de 50?**

Con chunks de 10 se necesitarían 5 commits pues en 5 chunks caben 50 movimientos, el mínimo suficiente para los 45 movimientos.

Con chunks de 50 se necesitaría solo un commit ya que los 45 movimientos caben en un solo chunk.

**4. ¿Por qué el Escritor recibe el chunk completo y no un movimiento a la vez?**

Porque puede escribir los elementos en grupo dentro de una misma transacción de forma eficiente.

**5. Mi predicción de la MP-3, paso 1: ¿qué habría pasado sin el Procesador?**

Los movimientos pasarían directamente del Reader al Writer, sin realizar ninguna transformación o validación intermedia.
