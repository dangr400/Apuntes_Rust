# En Rust
Por defecto, las variables son inmutables. Esto ayuda a aplicar concurrencia y procesamiento en paralelo en Rust, aunque también está abierta la opción de convertirlas a mutables con la palabra clave `mut`.

# Declaración
Para declarar una variable, se utiliza la palabra clave `let`, acompañada del nombre de la variable:

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
	x = 6;
    println!("The value of x is: {x}");

}
```
La variable `x`, una vez asignada, no puede ser reinstanciada. Si intentamos ejecutar el código obtenemos el siguiente error:
```console
 cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("The value of x is: {x}");
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

For more information about this error, try `rustc --explain E0384`.
error: could not compile `variables` due to previous error
```

Para poder mutar x, debería estar instanciada de la siguiente forma:
```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {x}");
	x = 6;
    println!("The value of x is: {x}");

}
```
