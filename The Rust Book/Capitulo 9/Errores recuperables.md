# Definción
En la mayoria de errores no son tan serios como para detener el programa en su totalidad, sino que la razón por la que falla es facilmente controlada. Por ejemplo, si se trata de leer un fichero que no existe, igual es preferible crear dicho fichero en vez de terminar el proceso.

# Manejo con Result
El enum `Result` se define con dos variantes `Ok` y `Err`:
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
`T` y `E` son parámetros de tipo genérico, en este caso `T` representa el tipo de valor que será devuelto en caso de éxito, y `E` es el tipo de error devuelto en caso de fallo.

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```
En este caso, el valor de retorno de `File:open` es un `Result<T,E>`. El parámetro genérico de éxito, `T`, es de tipo `std::fs::File`, el cual es un manejador de ficheros. El caso de error `E` sera de tipo `std::io::Error` Este tipo de retorno indica que la llamada a `File::open` puede devolver un manejador de ficheros que puede leer y escribir en el, o bien puede fallar, por ejemplo porque el fichero no existe o no tiene permisos para acceder al fichero.

En caso de que funcione, el valor en `greeting_file_result` sera una instancia de `Ok` que contiene el manejador de ficheros. En caso de error, el valor será un `Err` que contenga más información sobre el tipo de error recibido. En ambos casos, será necesario controlar lo que suceda:
```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```
La rama de `Ok` asigna el manejador de fichero a `greeting_file`, el cual se puede utilizar para modificar o leer el fichero, mientra que la rama `Err` detiene la ejecución del código para avisar del problema con el fichero.

# match con diferentes errores
En el programa anterior, se detendrá la ejecución del programa independientemente del error recibido, pero podemos controlar mejor dependiendo del error recibido:
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error);
            }
        },
    };
}
```

El enum `io::ErrorKind` es provisto por la libreria estandar y tiene variantes representando los distintos errores que pueden resultar de una operación `io`. La variante que queremos usar es `ErrorKind::NotFound`, que indica que no existe el fichero.

## alternativas a usar `match` con `Result<T,E>`
match es una expresión muy útil pero tambien una primitiva. En el capitulo 13 se aprende más de cierres (_closure_) los cuales se utilizan con muchos métodos definidos en `Result<T,E>`. Estos métodos pueden ser mas concisos que usar `match` al manejar valores de `Result<T,E>`:
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

# Atajos para panic en error: `unwrap` y `expect`
Utilizar `match` funciona bien, pero puede ser muy verboso y no siempre comunica bien la intención. El tipo `Result<T,E>` tiene muchos métodos de ayuda que realiza tareas más específicas. el método `unwrap` es un método de atajo que actua como si fuese una expresión `match`:
```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```
En este caso, si el valor de `Result` es `OK`, `unwrap` devuelve el valor dentro del `Ok`. Si es `Err`, unwrap llama a la macro `panic!`.

De forma similar, el método `expect` permite escoger el mensaje transmitido al mensaje de error en `panic!`:
```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```
En código de producción, se suele escoger `expect` en vez de `unwrap` y ofrecer más contexto de por que la operación se espera que funcione siempre.

# Propagar errores
Cuando la implementación de una función llama a algo que pueda fallar, en vez de manejar el error en la función, se puede devolver el error a la llamada de la función para decidir que hacer en cada caso. Esto se conoce como propagar (_propagating_) el error y ofrece más control sobre el código de llamada, donde puede haber más información o lógica que dicte como debe manejarse.
## Ejemplo: leer un fichero
```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```
Esta función retorna un tipo `Result<T,E>`, en donde `T` es de tipo `String` y `E` de tipo `io::Error`.
En caso de que la función proceda con éxito, el código que llame a esta función recibirá un tipo de dato `Ok` que contiene una `String`. En caso de fallo, el código que llama a la función recibe un `Err` que contiene una instancia de `io::Error` la cual contiene más información de por qué no funcionó correctamente.

# Atajo para propagar errores con `?`
A continuación se muestra el código previo actualizado con el atajo `?`:
```rust

use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```
El interrogante `?` colocado después del valor `Result` esta definido para funcionar casi como lo haría una expresión match en el código previo.
Si el valor de `Result` es un `Ok`, el valor dentro del `Ok` se retorna de la expresión, y el programa continuará Si el valor es de tipo `Err`, el `Err` será retornado de la función como si se hubiese utilizado un `return` para propagar el error al código que lo llama.

Hay una diferencia entre las expresiones `match` y el atajo `?`: los valores de error con el operador `?` pasa a través de la funcion `from`, definido en el rasgo `From` en la librería estándar, la cual se utiliza para convertir valores de un tipo a otro. Cuando el operador `?` llama a la función `from`, el tipo de error recibido se convierte en el tipo de error definido en el retorno de la función actual. Esto es útil cuando una funcion retorna un tipo de error para representar las formas en que una fucnión peude fallar, incluso aunque múltiples partes puedan fallar por diferentes motivos.

En este caso, el `?` ak final de la llamada de `File::open` retornará un valor dentro de un `Ok` a la variable `username_file`. Si ocurre alguún error, el operador `?` retornará de la función y devuelve cualquier valor de `Err` al código que llame a la función.

Se puede acortar aun más la fucnion mediante encadenaciones de llamadas al método inmediatamente después de `?`:
```rust

use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}
```

## más atajos
Dado que leer un fichero a cadena es una operación común, la librería estandar provee la función `fs::read_to_string` la cual:
1. Abre un fichero
2. Crea una nueva `String`
3. Lee los contenidos del fichero
4. Almacena los contenidos en esa `String`
5. Lo devuelve (retorna, _return_)
```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

# Donde usar `?`
El operador `?` solo puede ser usado en funciones cuyo tipo de retorno es compatible con el valor de donde se usa `?`. Esto se debe a que el operador `?` esta definido para realizar un retorno más pronto de lo devido de un valor fuera de la función. El valor de dicho retorno debe ser de tipo `Result`, o `Option`, o aquel que implemente `FromResidual`.

Hasta ahora, las fucniones `main` implementadas retornan `()`. Por suerte, `main` también pueden devolver un `Result<(),E>`:
```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```
El tipo `Box<dyn Error>` es un objeto de rasgos (_trait object_), el cual se analiza con más detalle en el capítulo 17. Por ahora, `Box<dyn Error>` significa "Cualquier tipo de error".

Cuando una función `main` retorna un `Result<(),E>`, el ejecutable saldrá con un valor de `0` si `main` retorna `Ok(())`, y saldrá con un valor distinto de `0` en caso de error.

La función `main` puede retornar cualquier tipo que implemente el rasgo `std::process::Termination`, el cual contiene una función `report` que retorna un `ExitCode`.