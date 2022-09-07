# Descripción del proyecto
Se procede a crear un sistema que va a "pensar" en un número entre el 1 y el 100, y el usuario debe adivinar qué número es. Si el usuario acierta, el sistema informa al usuasrio y se detiene la ejecución del programa.

# Iteraciones
## 1ª Iteración: Procesar la entrada del usuario
La primera parte del juego de la adivinanza es pedir un input del usuario, procesarlo, y comprobar que el input está en el formato esperado (en este caso, debe ser un número).
para ello, se utiliza la siguiente tira de código:
```rust
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```
A continuación se analiza cada sección del código:

### Importar "crates"

```rust
use std::io;
```
Para obtener el input del usuario y despues plasmar el resultado como salida, es necesaria la librería `io` en el "scope". Esta librería viene de la libreria estandar, conocida como `std`. Por defecto, rust tiene un set de items definidos en la librería estandar que incluye al scope de cada programa, los cuales se llaman preludios (_prelude_ en inglés).
Para más información sobre estas, continue a la [documentación de los preludes](https://doc.rust-lang.org/std/prelude/index.html).

### Crear función principal

```rust
fn main() {
```
La función main, la cual es el punto de entrada al programa, ya explicado en la [[Anatomía de un programa en Rust]]. `fn` indica que es una funcion, los `()` vacíos indican que no hay parámetros, y `{` indica que inicia el cuerpo de la función.

### Plasmar info. en el "display"

```rust
    println!("Guess the number!");
    println!("Please input your guess.");

```
println! es una macro que plasma una cadena en la pantalla, para informar al usuario de qué va el juego y qué debe hacer.

### Inicializar variable y almacenar info.

```rust
   let mut guess = String::new();
```
`let mut` inicializa la variable **mutable** "guess". El símbolo igual (=) le dice a Rust que queremos enlazar algo a la variable. A la derecha del signo = esta el valor que `guess` está enlazado, que es el resultado de llamar `String::new`, una funcion que devuelve una nueva isntancia de `String`. `String` es un tipo de cadena provisto por la librería estandar, el cual es un pedazo de texto que puede ampliarse, codificado en **UTF-8**.

la sintaxis de `::` en `String::new` indica que `new` es una función asociada al tipo `String` (Es decir, el texto a la derecha  de los `::` es una función _interna_ del tipo declarado a la izq.)

Una **función asociada** es una función implementada en un tipo de dato (en este caso, `String`). `new` crea una nueva cadena de caracteres vacía (en otros tipos de datos, la función `new` creará una instancia "vacía" de dicho tipo).

### Recibir "input" del usuario

```rust
    io::stdin()
        .read_line(&mut guess)
```
En estas líneas, se llama a la función `stdin` del módulo `io` (importado al inicio del fichero).
Si no se hubiese importado la librería `io`, se podría acceder a la funcioon llamandola tal que así:
```rust
    std::io::stdin()
```

La función `stdin` devuelve una instancia de `std::io::Stdin`, el cual es un tipo que representa un manejador para la entrada **estandar** de la terminal. 

La siguiente línea (`.read_line(&mut guess)`) llama al método `read_line` en el manejador, para recibir el input del usuario. También, estamos pasando `&mut guess` como el argumento de `read_line` para decirle dónde debe almacenar dicha cadena. El rol de `read_line` es coger lo que sea que el usuario escriba en la entrada estandar y agregarlo (_append_) en la cadena (sin sobreescribir su contenido), para que podamos pasar la cadena como un argumento. Es necesario que el argumento sea mutable para poder cambiar su contenido.

El **`&`** indica que el argumento es una **[[referencia]]**, la cual ofrece una forma para acceder, desde una seccion del código, a una pieza de información sin necesidad de copiar dicha información a memoria múltiples veces. Las referencias, por defecto, son **inmutables**. Por eso es necesario añadir `&mut guess`, en vez de `&guess`, para poder mutarla.

### Manejando errores potenciales con el tipo _Result_
```rust
        .expect("Failed to read line");
```
`read_line`, a parte de cumplir su función explicada previamente (almacenar el input), también devuelve un valor **`Result`**. Este tipo de valor es un **enumerado**, también llamado **enum**, el cual es un tipo de dato que puede ser uno de sus múltiples estados. Cada posible estado se llama **variante** (_variant_ en inglés).

El propósito de los **`Result`** es para codificar la información de manejo de errores. Las variantes de Result son `Ok` y `Err`. `Ok` indica que la operación ha sido exitosa, y `Err` que se produjo un error. `Err` también contiene información de cómo o por qué ha fallado la operación.

Así como `String` tenía métodos definidos en el, `Result` y el resto de tipos también tienen métodos. En una instancia de `Result`, hay un método llamado `expect` el cual puede ser llamado. 

En este caso, si la instancia de `Result` es un valor de `Err`, `expect` causará que el programa crashee, y mostrará la cadena que se pasa como argumento. Si el método`read_line` devuelve un `Err`, lo más probable es que sesa resultado de un error vienendo del sistema operativo sobre el que se ejecuta el programa. 
Si la instancia de `Result` devuelve un `Ok`, `expect` recoge el valor retornado, el cual está almacenado en `Ok`, y devuelve únicamente ese valor para que pueda ser usado. En este caso, el valor es el numero de bytes en el input del usuario.

Si no se llama al método `expect`, el programa compilará igual, pero el resultado de la compilación avisará de que no se está gestionando dicho posible error.

La manera correcta de suprimir los avisos es escribiendo código que gestione dichos errores, pero en este caso es suficiente con detener el programa con un crash.

### Plasmar valores con `println!` y _Placeholders_
```rust
    println!("You guessed: {guess}");
```
Esta línea de código imprime la cadena que contiene el input del usuario. El par `{}` actúa como un _placeholder_, es decir, es donde se mantiene la posición del valor. Se puede:
- Referenciar dentro del propio par de llaves (como en el ejemplo)
- Referenciar en los paréntesis de la macro `println!` 
	(`println!("x = {} and y = {}", x, y)`)

Hasta aquí, se finaliza la 1ª iteración

## 2ª iteración: Incorporar crates
Ahora que ya controlamos el input del usuario, pasamos a la 2ª parte: hacer que el programa "piense" en un número aleatorio. La librería estandar no tiene incorporado una funcionalidad de crear números aleatorios, pero sí que existe una _crate_ con dicha funcionalidad (llamada `rand`)

### Utilizar crates para obtener más funcionalidad
El programa hasta ahora, que estabamos creando, es una _crate binaria_, es decir, un ejecutable. la `rand` _crate_ es una _crate de libreria_, la cual contiene código que será usado en otros programas y no se puede ejecutar por si sólo.

La coordinación de cargo de crates externas es el punto clave de Cargo. Antes de que se pueda escribir código que utilice la librería `rand`, es necesario **modificar el fichero Cargo.toml** paraincluir la crate `rand` como una dependencia. Para ello, se incluye despueś de la cabecera `[dependencies]` para indicar al Cargo que crates necesita y que versión de estas. En este caso, escribimos lo siguiente:
```toml
[dependencies]
rand = "0.8.3"
```

### Versionado Semántico (SemVer)
El número introducido es, en realidad, una abreviatura de "^0.8.3", el cual indica que es cualquier versión que sea al menos `0.8.3`, pero por debajo de `0.9.0`. Esto se debe a la compatilidad de las APIs públicas de dichos crates, para asegurar su operabilidad.

### Crates.IO
Es la página donde los Rustáceos (programadores de Rust) suben su código abierto para que otros lo puedan utilizar.

### Post-actualización
después de actualizar el registro, Cargo comprueba la sección de `[dependencies]` y descarga las _crates_ que todavía no han sido descargadas. En este caso, a pesar de que solo incluimos la _crate_ `rand`, se descargan otras crates (de las cuales depende `rand`). Cargo se encarga de gestionar, de la mejor forma posible, las dependencias (si se modifica solo el código y no las dependencias, solo buildea el código, y así).

### Asegurar reproducibilidad en las _Builds_
Cargo tiene un mecanismo para asegurarse que se pueda construir el mismo artefacto cada vez que cualquiera haga una _build_ de este: Cargo solo utilizará las versiones de las dependencias que se especifican, salvo que se le indique lo contrario. Para esto existe el documento `Cargo.lock`, para que Cargo, una vez buildee el proyecto, se refiera a las versiones establecidas en este documento, en vez de buscarlas por si mismo a la hora de buildear.

### Actualizar Crate para obtener una nueva versión
Cargo cuenta con el comando `update` para actualizar una crate. Este comando ignora el documento _Cargo.lock_, y averigua todas las últimas versiones de las especificaciones en el documento _Cargo.toml_. Una vez averiguadas, se sobreescribe el ficher _Cargo.lock_, **Recordando que no llegará la version 0.9.x** en este caso. Para que la contemplase como opción, haría falta indicarlo en el doc _Cargo.toml_

(para más info del ecosistema de versionado y demás, continúe al [ecosistema de Rust](https://doc.rust-lang.org/cargo/reference/publishing.html))

## 3ª Iteracion: Generar un número aleatorio
Actualizamos el código en _src/main.rs_ como se indica:
```rust
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```


## 4ª Iteracion: Comparar nº adivinado con nº secreto

TODO : CONTINUARÁ