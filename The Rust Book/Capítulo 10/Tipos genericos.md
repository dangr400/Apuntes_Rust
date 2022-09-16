# Uso
Se utilizan los genericos para crear definiciones para objemtos como funciones firma o structs, los cuales se puede entonces utilizar con distintos tipos de tipos de datos concretos.

## En funciones
Cuando se define una funcion que utiliza genericos, ponemos los genericos en la firma de la función donde, de normal, especificariamos el tipo de dato de los parametros y del tipo de retorno. Haciendolo así hace el código más flexible y permite mas funcionalidad a los que llamas a nuestra función mientras previene duplicación de código.

A continuación se muestra un ejemplo de funcion que vamos a generizar:
```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
```
Ambas funciones realizan la misma tarea pero con tipos de datos distintos: buscan el valor mayor en una lista de elementos, pero `largest_i32` utiliza datos de tipo `i32` y `largest_char` con `char`.

Para parametrizar los tipos en una nueva función única, es necesario nombrar el tipo del parámetro, tal como se hace con los valores de los parámetros de una función. Se puede usar cualquier identificador como nombre del tipo de parámetro, pero por convenio se utiliza solo una letra, y el type-naming en Rust es CamelCase, por lo que se utilizará `T` como tipo.

Cuando se utiliza un parámetro en el cuerpo de una función, hay que declararel nombre del parámetro en la firma para que el compilador sepa que significa el nombre. Similarmente, cuando utilizamos el nombre de un parámetro en la firma de una función, hay que declarar el tipo del parámetro antes de utilizarlo.

Para definir la funcion genérica `largest`, hay que colocar las declaraciones del nombre del tipo entre `<>`, en medio del nombre de la función y de la lista de parámetros:
```rust
fn largest<T>(list: &[T]) -> &T {
```
Se lee la definicion de la siguiente forma: la función `largest` es genérica sobre algún tipo `T`. Esta función tiene un parámetro llamado `list`, el cual es un trozo de valores de tipo `T`. La función devolverá una referencia a algún valor de tipo `T`.

A continuacion se muestra la función definida sobre tipo genérico en su firma. también se muestra como se puede llamar a la función bien con un trozo de valores `i32` como de valores `char`. Aunque este código no compilará todavía, se arregalrá más adelante:
```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```
```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
error: could not compile `chapter10` due to previous error
```
El texto de ayuda menciona `std::cmp::PartialOrd`, el cual es un rasgo, de lo que se habla en el punto [[Rasgos]]. Por ahora, solo hace falta saber que este error insta que el cuerpo de `largest` no funcionará para todos los tipos posibles que pueden ser `T`. Dado que queremos comparar valores de tipo `T` en el cuerpo, podemos solo usar tipos de datos que puedan ser ordenados. Para habilitar comparaciones, la librería estandar cuenta con el rasgo `std::cmp::PartialOrd` el cual se puede implementar en los tipos.  Siguiendo el ejemplo de la sugestion del compilador, se puede restringir los tipos validos para `T` a solo aquellos que implementen `PartialOrd` para que compile, ya que la librería estandar implementa este rasgo para tanto `i32` como `char`:
```Rust
fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
```

## En Structs
También se puede definir structs para que utilicen un parámetro de tipo genérico en uno o más campos utilizando la sintaxis de `<>`:
```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```
La sintaxis para usar genéricos en definiciones de struct es similar a la utilizada en definición de funciones. Primero, se declara el nombre del tipo del parámetro dentro de parentesis angulares (`<>`) justo despues del nombre de la struct. Después, se usa el tipo genérico en la definición del struct donde se especificaría el tipo concreto.

Notese que en la definición del struct solo se usa un tipo genérico, por lo que la definición dice que todos los campos genéricos (en este caso `x` e `y` ) utilizan el mismo tipo los dos, sea cual sea el tipo. Si se trata de crear una instancia de `Point<T>` con distintos valores, el código no compilará:
```rust
struct Point<T> {
    x: T,
    y: T,
}
// el siguiente codigo no compilará
fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```
```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0308]: mismatched types
 --> src/main.rs:7:38
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integer, found floating-point number

For more information about this error, try `rustc --explain E0308`.
error: could not compile `chapter10` due to previous error
```

Para definir un `Point` que acepte genéricos pero de distinto tipo, es necesario utilizar múltiples genéricos:
```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```
Se pueden usar tantos genéricos como se desée, pero el hecho de tener demasiados puede hacer difícil su comprensión, por lo que igual es necesario reestructurar el proyecto en pedazos más pequeños.

## En Enums
Se puede definir enums para conetener tipos de datos genéricos en sus variantes, tal como el enum provisto por la libreria estandar `Option<T>`:
```rust
enum Option<T> {
    Some(T),
    None,
}
```
Como se puede ver, `Option<T>` es un enum con un tipo genérico `T` con dos variantes: `Some`, el cual contiene un valor de tipo `T`, y `None` la cual no contiene ningun valor. Al utilizar el enum `Option<T>`, se puede expresar el concepto abstracto de un valor opcional, y dado que es genérico, se puede utilizar esta abstracción independientemente del tipo de dato opcional que sea.

También puede utilizarse múltiples tipos genéricos, como en `Result<T, E>`:
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
Cuando se identifiquen situaciones en el código con múltiples structs o enums que difieren solo en el tipo de valores que contienen, se puede evitar duplicación utilizando tipos genéricos en su lugar.

## En métodos
Se puede implementar métodos en structs o enums y utilizar tipos genéricos en sus definiciones también:
```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```
Notese que es necesario declarar `T` justo después de `impl` para poder usar `T` para especificar que se implementan métodos en el tipo `Point<T>`. Al declarar `T` como tipo genérico después de `impl`, Rust puede identificar que el tipo en `Point<T>` es un genérico en vez de uno concreto. Se podría escribir un nombre diferente para este parámetro genérico, pero usar el mismo nombre es convencional. Los métodos escritos en un `impl` que decaran el tipo genérico seran definidos en cualquier isntancia del tipo, independientemente del tipo concreto que acabe sustituyendo.

Tambiéns se puede definir restricciones en los tipos genéricos cuando se definen métodos en el tipo. Se podría, por ejemplo, implementar métodos solo en instancias `Point<f32>` en vez de instancias con cualquier genérico:
```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```
Aquí utilizamos el tipo concreto `f32`, lo que significa que no declaramos ningun tipo después de `impl`. Dicho de otra forma, significa que `Point<f32>` tendrá un método llamado `distance_from_origin`, otras instancias de `Point<T>` donde `T` no es de tipo `f32` no tendrán este método definido, ya que utiliza funciones solo disponibles para tipos de punto flotante.

Los tipos genéricos en la definición de un struct no siempre son los mismos que aquellos utilizados en los métodos asociados al struct. En el siguiente código se utilizan los tipos genericos `X1` e `Y1` para el struct `Point`, y `X2` e `Y2` para el método `mixup` para mostrar mejor el ejemplo:
```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```
En `main`, se define un `Point` que tiene un `i32` para `x` y un `f64` para `y`.
La variable `p2` es un struct de `Point` el cual tiene un pedazo de String en `x`, y un `char` para `y`.
Llamar `mixup` en `p1` con argumento `p2` nos da `p3`, el cual tendra un `i32` en `x`, y un `char` en `y`.

El propósito de este ejemplo es demostrar una situación en la cual algún parámetro esta declarado con `impl` y algunos son declarados en la definición del método. Aquí, los parámetros genéricos `X1` e `Y1` son declarados despues de `impl` porque van con la definición del struct. Los parámetros genericos `X2` e `Y2` son declarados despues de `fn mixup`, porque solo son relevantes al método. 
# Rendimiento del código usando genericos
Utilizar tipos genéricos no se ejecuta más despacio que con tipos concretos. Esto se debe gracias a la monomormización que realiza Rust del código que utilice tipos genéricos en tiempo de compilación.

Monoformización (_Monomorphization_) es el proceso de convertir codigo genérico en código específico mediante rellenar los tipos concretos que son utilizados cuando se compila. En este proceso, el compilador hace lo opuesto de los pasos de crear una función genérica: el compilador mira todos los lugares donde se llama a código genérico y genera código para los tipos concretos que llaman al código