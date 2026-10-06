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
