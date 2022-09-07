# Definición
Una caja (o _crate_) es la menor cantidad de código que el compilador de Rust considera de cada vez, incluso si se ejecuta `rustc` en vez de `cargo` y se pasa un solo documento de código fuente, el compilador considera ese fichero como una caja.

Una caja puede conetener módulos, y los módulos pueden ser definidos en otros ficheros que se compilan con la caja.

# Tipos
## Cajas binarias
Las cajas binarias (_binary Crates_) son progrmaas que se pueden compilar en un ejecutable, como un servidor o un programa de linea de comandos.

Cada tipo de este tipo de caja debe tener una función llamada `main` que define lo que ocurre cuando el ejecutable se ejecuta.
## Cajas de librería
Son cajas que no cuentan con una función `main`, y no se compilan a un ejecutable. En su lugar, definen funcionalidades que serán compartidas entre múltiples proyectos. Por ejemplo, la caja `rand` utilizada en [[Ejemplo 1. Adivinar el número]] provée la funcionalidad de generar números aleatorios. La mayor parte de las veces, cuando un desarrollador dice "crate" se refieren a una librería.

## Caja raiz (_crate root_)
Es un fichero de código en el cual el compilador  de RUst empieza y crea el módulo basae de tu caja.