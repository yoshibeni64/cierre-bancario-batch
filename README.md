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
rodrigo@DESKTOP-GPU0MG4:~/cierre-bancario-batch$

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

## Día 4 · De MySQL a MongoDB

### Boleto de salida

**1. ¿Qué hace cada uno de los tres steps de tu Job, y de qué tipo es cada uno?**

`verificarArchivoStep`: Comprueba que exista el archivo de movimientos que corresponde a la fecha del cierre y muestra cuántos movimientos contiene. Es un Step de tipo Tasklet.

`cargarMovimientosStep`: Lee los movimientos del archivo `.csv`, los procesa y los registra en la tabla `movimiento` de MySQL. Es un Step de procesamiento por chunks de 10 elementos por chunk.

`publicarSaldosStep`: Obtiene los saldos agrupados de la base de datos en MySQL y los guarda en la colección `saldos` de MongoDB. También es un Step de procesamiento por chunks. Para ejecutarse primero deben completarse los dos Steps anteriores.

**2. ¿Por qué el cierre del 9 no duplicó los saldos, y el del 10 (sin `@Id`) sí?**

Porque la clase `SaldoCuenta` tenía un campo identificado como `@Id`. MongoDB reconoció ese valor como `_id`, por lo que al guardar nuevamente un saldo de una cuenta, se actualiza el documento existente identificado con ese `_id` en lugar de crear uno nuevo.

Por otro lado, al quitar `@Id`, MongoDB genera un `_id` diferente para cada documento guardado, por lo que se insertan documentos nuevos cada vez que se ejecuta el cierre en lugar de actualizar los existentes.

**3. ¿Al reiniciar el cierre del 11, por qué no se cargó otra vez el archivo?**

Porque Spring Batch guarda el estado de la ejecución en su repositorio. Cuando un Job falla y se reinicia con los mismos parámetros, Spring Batch puede recuperar el estado anterior y continuar desde donde quedó, evitando volver a procesar los datos que ya habían sido procesados correctamente.

**4. ¿Qué diferencia hay entre `spring-boot-starter-data-mongodb` y Spring Batch MongoDB (`batch-data-mongodb`)?**

El primero se usa para integrar una aplicación Spring Boot con MongoDB, proporcionando herramientas de Spring Data MongoDB, como `MongoTemplate` y los repositorios.

El segundo proporciona herramientas específicas para utilizar MongoDB dentro de un proceso batch, como `MongoItemWriter`, que permite escribir los datos procesados por un Step directamente en MongoDB.

## Lo que aprendí esta semana

Un proceso batch permite procesar una cantidad de información de forma automática y organizada. Un Job está formado por uno o varios Steps que se ejecutan en un orden programado. Por ejemplo, en esta práctica, primero se verifica el archivo, luego se cargan los movimientos en MySQL y finalmente se publican los saldos en MongoDB. Los Steps pueden trabajar como Tasklet o mediante procesamiento por chunks. Spring Batch guarda información de cada ejecución en su repositorio, lo que permite saber el estado del proceso y si se completó adecuadamente. Cuando un Step falla, el Job puede reiniciarse y continuar utilizando el estado guardado, evitando repetir el trabajo que ya terminó correctamente.
