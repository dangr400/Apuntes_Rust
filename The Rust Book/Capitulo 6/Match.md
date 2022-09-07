# Definición
La expresión de control de flujo Match permite comparar un valor contra una serie de patrones y ejecutar código en base a que patrón coincide con cual. Los patrones pueden estar compuestos de valores literales, nombres de variable, wildcars, y muchas otras cosas (en el cpitulo 18 se explica más detalladamente).

# Potencial
El poder de `match` viene de la expresividad de los patrones y del hecho que el compilador confirma que todos los posibles casos son controlados.

## Ejemplo: monedas
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```
Primero, se define la expresión `match` seguida de una expresión, en este acso el valor de `coin`. Es similar a una expresión con `if`, pero con una gran diferencia: `if` necesita que la expresión devuelva un [[Tipos de datos]] Booleano, pero en este caso puede ser cualquier valor.

Después vienen las ramificaciones del `match`. Una ramificación tiene dos partes:
- Un patrón
- código
La primera ramificación tiene un patrón que es el valor `Coin:Penny` y después el operador `=>` que separa el patrón del código a ejecutar (En este caso, solo devuelve el valor `1`.

Cada rama se separa de la siguiente con una coma (`,`).

Cuando se ejecuta la expresión `match`, compara el valor resultado contra el patrón de cada brazo, en orden. Si un patrón coincide en el valor, el codigo asociado con ese patrón se ejecuta. Si la comparación no coincide, la ejecución continua en la siguiente ramificación. Puede haber tantas ramificaciones como haga falta.

El código asociado con cada ramificación es una expresión, y el valor resultante de la expresión es el valor que se retorna para la expresión `match`

Para poder ejecuetar múltiples lineas de código, es encesario añadir llaves (`{}`), y la coma al final del bloque es opcional:
```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

# Patrones que se asocian a valores
Otra característica útil de las ramificaciones es que pueden bindearase (asociarse) a partes de lavores que coinciden con el patrón. De esta forma se puede extraer valores de las variantes de un enum:
```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```
```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

# Coincidencia con `Option<T>`
En la sección de [[Enums]], queriamos obtener el valor interno de `T` de un caso `Some` al utilizar `Option<T>`; tambien podemos manejar dicho tipo de dato utilizando `match` como en el ejemplo anterior: Comparamos las variantes de `Option<T>` con la misma forma de actuar de `match`.
```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
```
Cuando se llama `plus_one(five)`, la variable `x` en el cuepro de `plus_one` tendrá el valor `Some(5)`. Después, se compara con cada ramificación: Como no coincide con `None`, continua a la siguiente ramificación, en donde se compara `Some(5)` con `Some(i)`. Dado que coinciden, la `i` se bindea con el valor contenido en `Some`, por lo que tiene el valor `5`. El código es ejecutao, añadiendo 1 al valor de `i`, y creando un nuevo valor `Some` con el total dentro.

# Exaustivo
Los matches en Rust son exhaustivos: se necesita comprobar todas las posibilidades para que el código sea válido. Si no se contemplan todas las posibilidades, Rust no compilará el código.

# Patrones para capturar multiples situaciones
También se puede programar un match para comprobar cierto casos, y tener una acción por defecto para el resto.
```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

También existe un patrón para realizar captura de múltiples casos, pero sin utilizar el valor de la situación capturada: con el caracater `_` captura cualquier valor y no lo bindea a ese valor.
```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```

También se puede hacer que al recibir cualquier otro caso no se realice ninguna acción:
```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
```