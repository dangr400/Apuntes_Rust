# Macro `panic!`
Existen dos formas de causar un panic en la práctica:
1. Realizar una accion que causa al codigo un panic (como acceder a un indice inexistente en un array)
2. Llamando explícitamente a la macro `panic!`

En ambos casos, se causa un panic en el programa. Por defecto, estos panic mostraran un mensaje de error, rebobinarán, limpiarán el stack y saldrán. Con una variable de entorno, se puede hacer que Rust muestre el stack de llamadas para facilitar la búsqueda del origen del panic.

## Rebobinar el stack o abortar en respuesta
Por defecto, cuando sucede un panic, el programa comienza a rebobinar (_unwind_), lo que quiere decir que Rust retrocede en el stack y limpia la información de cada funcion que encuentre. Sin embargo, este proceso lleva una buena cantidad de trabajo, por lo que existe la opcion de abortar, el cual finaliza el programa sin realizar limpieza (esta queda delegada al S.O.).
Para abortar, es necesario incluir `panic = 'abort'` en el fichero de configuracion _Cargo.toml_ en el perfic que se desee.

Por ejemplo, si se desea abortar en un panic en el proyecto de release, hay que añadirlo bajo ese perfil:
```toml
[profile.release]
panic = 'abort'
```

## Usando el retroceso en un panic!
En C, intentar leer mas allá del límite de una estrucuta de datos es comportamiento sin definir. Puede recuperar lo que sea que haya en ese límite, esto se conoce como _buffer overread_ y puede desembocar en vulnerabilidades de seguridad si un atacante es capaz de manipular el indice de tal forma que acabe leyendo información privilegiada.

En Rust, se detiene la ejecución del programa en caso de que suceda algo como esto.
```rust
// documento: src/main.rs
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```
```console
RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/std/src/panicking.rs:483
   1: core::panicking::panic_fmt
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/panicking.rs:85
   2: core::panicking::panic_bounds_check
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/panicking.rs:62
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/slice/index.rs:255
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/slice/index.rs:15
   5: <alloc::vec::Vec<T> as core::ops::index::Index<I>>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/alloc/src/vec.rs:1982
   6: panic::main
             at ./src/main.rs:4
   7: core::ops::function::FnOnce::call_once
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/ops/function.rs:227
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```
El retroceso (_backtrace_) es una lsita de las funciones que han sido llamadas para llegar a este punto. Se empieza desde arriba hasta llegar a los ficheros que hayamos escrito nosotros.