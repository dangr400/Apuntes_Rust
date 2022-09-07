# Funcion Main
>fn main() { }

Una función se define con la palabra reservada **fn**.
`main` es siempre la primera sección del código que se ejecuta en todos los programas ejecutables de [[Rust]].

En el ejemple de arriba, se observa que no se pasan parámetros. Si los hubiese, irían nombrados en el paréntesis.

# Formato
>Para adherirse al estándar de código de Rust, existe la herramienta de formateo automático `rustfmt` 

- Las indentaciones son con **4 espacios**, no una tabulación
- Las lineas se terminan con punto y coma (`;`), lo que indica que esta expresión ha terminado y la siguiente está lista para continuar
- Las llamadas se representan con su nombre, seguido de paréntesis -> call(parametros)
- las [[macros de Rust]] siguen el siguente formato: `nombre_macro!` (! al final del nombre)

# Variables
para iniciar una variable, se utiliza la palabra `let`
ejemplo:
```rust
    let manzanas = 5;
```
Por deefecto, las variables en rust son **inmutables**: una vez se les asigna un valor, no puede modificarse dicho valor.
Para convertir una variable a mutable, hay que añadir la palabra clave `mut`:
```rust
let apples = 5; // inmutable
let mut bananas = 5; // mutable
```

## Tipos de Variables
- ### String
- ### ¿?
# Palabras Reservadas
Existen tanto [[Palabras Reservadas]] como [[Palabras Reservadas Futuras]]