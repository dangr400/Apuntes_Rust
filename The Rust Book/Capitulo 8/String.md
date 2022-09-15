# Definición
Un tipo de dato `String` en Rust es un tipo de cadena de caracteres que:
- puede crecer
- puede mutar
- Puede cambiar de dueño
- Está codificado en UTF-8

Está incluida en la librería estandar de Rust en vez de estar incluida en el núcleo del programa
# Crear un String
Muchas de las operaciones disponibles en un [[Vector]] `Vec<T>` estan disponibles con `String`, ya que String es realmente implementado como un envoltorio de un vector de bytes, con una serie de capacidades, restricciones y garantías extra. 

Debido a esto, se crea una nueva String con la función `new`:
```rust
	let mut s = String::new();
```

Con esta declaración se crea una nueva cadena de caracteres llamada `s` en la cual se puede cargar datos detro.

Si se quiere cargar un String con datos iniciales se puede utilizar el método `to_string`, disponible en cualquier tipo que implemente el rasgo `Display`, como las cadenas:
```rust
    let data = "initial contents";

    let s = data.to_string();

    // the method also works on a literal directly:
    let s = "initial contents".to_string();
```

También se puede utilizar la función `String::from` para crear una cadena. Su uso es el mismo que `to_string`:
```rust
    let s = String::from("initial contents");
```

# Actualizar un String
Un `String` peude crecer y su contenido puede modificarse, justo como un vector `Vec<T>`
## Adjuntar con `push_str` y `push`
```rust
    let mut s = String::from("foo");
    s.push_str("bar");
```
El método `push_str` toma un pedazo de cadena porque no queremos tomar la pertenencia del parámetro
```rust
    let mut s1 = String::from("foo");
    let s2 = "bar";
	s1.push_str(s2);   // s2 sigue siendo utilizable después de esta linea
    println!("s2 is {}", s2);
```

El método `push` toma un caracter como parámetro y lo agrega en el `String`:
```rust
    let mut s = String::from("lo");
    s.push('l');
```

## Concatenar con operador `+` o con macro `format!`
```rust
	let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```
Hay que fijarse en la forma de como funciona el operador `+`. el operador `+` utiliza el método `add`, cuya firma es la siguiente:
```rust
fn add(self, s: &str) -> String {
```
Necesita un tipo y una referencia como parámetros, por lo que utilizamos `s1` como tipo y `s2` como referencia. Debido a esto, s1 queda inutilizable después de la concatenación, pero `s2` sigue siendo utilizable.
```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = s1 + "-" + &s2 + "-" + &s3;
    // s1 queda inutilizable, pero s2 y s3 siguen siendo válidas
```

Con la macro `format!` podemos realizar una concatenación más simple de observar:
```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{}-{}-{}", s1, s2, s3);
```
El código generado por format utiliza referencias por lo que esta llamada no toma posesión de ninguno de los parámetros, haciendo s1,s2,s3 válidas.

# Indexar Strings
En Rust no se permite acceder a valores individuales de un String con índices, debido a como se almacena un String en memoria:
```rust
    let s1 = String::from("hello");
    let h = s1[0];
```
```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0277]: the type `String` cannot be indexed by `{integer}`
 --> src/main.rs:3:13
  |
3 |     let h = s1[0];
  |             ^^^^^ `String` cannot be indexed by `{integer}`
  |
  = help: the trait `Index<{integer}>` is not implemented for `String`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `collections` due to previous error
```
## Representación interna
Ya que los valores en String son codificados en UTF-8, acceder con un índice solo devolvería el byte apuntado en ese índice, por lo que devolverá un valor que no es simbólico (no es el caracter en si). Otro motivo es que las operaciones de indexación deben tomar un tiempo constante O(1), lo cual no sucede en este caso ya que debe recorrer el String para asegurarse que de indexa los valores correctos.

## Particionar Strings
Si de verdad es necesario utilizar índices, se puede crear índices con un rango válido (dado que se almacenan los caracteres en 2 bytes, tienen que ser slices en potencias de 2):
```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

En caso de utilizar slices con tamaños de caracter inválido (por ejemplo, `&hello[0..1]`), Rust detendrá la ejecución con un [[panic!]] en tiempo de ejecución:
```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/collections`
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', library/core/src/str/mod.rs:127:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrac
```

# Iterar sobre Strings
La mejor forma de operear en piezas de strings es ser explícitos sobre si se quieren caracteres o bytes:
1. Con el método `chars` separará los caracteres de una cadena, devolviendo `char` para operar sobre estos:
```rust
for c in "Зд".chars() {
    println!("{}", c);
}
```
```text
З
д
```
2. Con el método `bytes` se devuelve el byte en crudo:
```rust
for b in "Зд".bytes() {
    println!("{}", b);
}
```
```text
208
151
208
180
```