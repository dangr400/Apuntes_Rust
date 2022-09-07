# Tipo de dato Slice
Un slice permite referenciar una secuencia de elementos contiguos en una colección en vez de la colección entera. Es un tipo de referencia, por lo que no posée pertenencia.

## String Slice
Es una referencia a parte de un `String`:
```rust
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
```
Se crea un slice utilizando un rango entre corchetes, donde el primer elemento es la posicion inicial del slice y la última es uno mas que la última posicion del slice. 

Se puede ignorar el elemento incial si se quiere empezar el slice en el principio de la cadena
```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Tambien se puede ignorar el elemento final si se quiere referenciar hasta el final de la cadena:
```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

También se peude ignorar ambos para slicear toda la cadena;
```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

### Ejemplo: recuperar primera palabra en cadena
```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

## String literals
Una cadena literal se almacena en el binario. Realmente, una variable que almacene un literal es del tipo `&str`, es decir, un slice apuntando a un punto especifico en el binario. Esto explica también por que son inmutables, ya que la referencia es inmutable.

# Slices como parámetros
Un cambio destacable que puede mejorar la funcion de `first_word` sería cambiar el parámetro a un slice para admitir tanto `&String` como `&str`:
```rust
fn first_word(s: &str) -> &str {
```
Si hay un slice de string, podemos pasarlo como parámetro directamente. Si tenemos un `String`, podemos pasar un slice de este, o una referencia al `String`.

Definir una funcion para recibir una referencia a String convierte la API más genérica y útil sin perder funcionalidad:

```rust
fn main() {
    let my_string = String::from("hello world");

    // `first_word` works on slices of `String`s, whether partial or whole
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // `first_word` also works on references to `String`s, which are equivalent
    // to whole slices of `String`s
    let word = first_word(&my_string);

    let my_string_literal = "hello world";

    // `first_word` works on slices of string literals, whether partial or whole
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

# Otros slices
Existe un tipo más genérico de slice, del tipo `&[<tipo_dato>]`. Funciona de la misma forma que un slice de String, pero se puede utilizar para muchos tipos de colecciones.