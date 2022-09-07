# Descripción
Shadowing consiste en declarar una nueva variable con el mismo nombre que otra variable que ya existía previamente.

# En Rust
```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}"); // x = 12
    }

    println!("The value of x is: {x}"); // x = 6
}
```
Pra ensombrecer una variable, solo hay que redefinirla como una variable nueva. Cuando se ensombrece, afecta hasta que **termina el nivel en el que fue ensombrecida**.

# Diferencias con mutabilidad
Shadowing puede parecer igual a una mutabilidad, pero está reescribiendo la variable en el scope que se realiza la nueva declaración y manteniendo la inmutabilidad una vez modificada, en vez de modificar la variable en si, como se haría con una variable declarada con `mut`.

Además, con mutabilidad, es necesario mantener el tipo de variable (si la primera es un String, debe permanecer como tal), mientras que con un ensombrecimiento, se puede cambiar el tipo de dato que alamcena.

