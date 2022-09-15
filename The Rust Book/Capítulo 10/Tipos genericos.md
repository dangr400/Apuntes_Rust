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
