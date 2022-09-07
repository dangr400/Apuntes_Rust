# Definición
La sintaxis `if let` permite combinar `if` y `let` en una forma menos verbosa de manejar valores que coincide con un patrón mientras ignora el resto.

Considere el siguiente código:
```rust
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }
```

Ahora, el siguiente código se comporta de la misma forma que el ejemplo anterior:
```rust
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
```

# Sintaxis
Recibe un patrón y una expresión separada por un signo `=`. Funciona de la misma forma que un `match`, donde la expresión se ofrece al `match`, y el patrón es la primera ramificación. En este caso, el patrón es `Some(max)`, y el `max` se bindea al valor dentro de `Some`. Se puede, entonces, usar el valor `max` en el cuerpo del bloque de la misma forma que se utiliza `max` en el correspondiente ramificación del `match`. El código en el bloque `if let` no se ejecutará si el valor no coincide con el patrón.

También se puede añadir un bloque `else`: El bloque de código que va en `else` es el mismo que un bloque de código que iría con el caso `_` en una expresión `match` equivalente al `if let` y `else`:
```rust
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
```
```rust
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }
```