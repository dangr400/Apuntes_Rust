# Definición
Un enum permite definir el tipo de valor al enumerar sus posibles _variantes_:
```rust
enum IpAddrKind {
    V4,
    V6,
}
```

# Declaración
Se puede crear instancias de un enum de la siguiente forma:
```rust
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
```

Hay que destacar que las variantes de un enum se espacia de nombres (_namespaced_) bajo su identificador, y se utiliaz dos doble puntos (`::`) para separarlos. Esto ayuda en que ambos valores (`IpAddrKind::V4` y `IpAddrKind::V6`) son del mismo tipo (`IpAddrKind`). Esto permite definir una funcion que reciba un valor de este tipo, y llamarla con cualquiera de sus variantes:
```rust
fn route(ip_kind: IpAddrKind) {}
```

```rust
    route(IpAddrKind::V4);
    route(IpAddrKind::V6);
```

# Asignacion de valores
Se puede introducir valores directamente en cada una de las variantes de un enum: En vez de definirlo dentro de un struct, tal que así:
```rust
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind,
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from("::1"),
    };
```

Se puede definir de la siguiente forma:
```rust
    enum IpAddr {
        V4(String),
        V6(String),
    }

    let home = IpAddr::V4(String::from("127.0.0.1"));

    let loopback = IpAddr::V6(String::from("::1"));
```

Añadimos información a cada variante del enum, por lo que no hay necesidad de una struct para almacenar los datos

## Cómo funciona
El nombre de cada variante de un enum que definimos también se convierte en una funcion que construye una instancia del enum. En el ejemplo anterior, `IpAddr::V4()` es una función que recibe un `String` como parámetro y devuelve una instancia del tipo `IpAddr`.

Cada variante puede tener diferentes tipos y cantidad de datos asociados:
```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
```

## Ejemplo: IpAddr en libreria estandar
```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Como se puede observar, se puede poner cualquier tipo de información en una variante de enum, incluidos [[Structs]].

# Asignacion de métodos
Otra similitud con los [[Structs]] es que se puede definir métodos con `impl` para enums:
```rust
fn main() {
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
        Write(String),
        ChangeColor(i32, i32, i32),
    }

    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from("hello"));
    m.call();
}
```

# El enum `Option` y sus ventajas sobre valores nulos
Rust no cuenta con el valor nulo (null). En su lugar, cuenta con un enum que puede codificar el concepto de un valor presente o absento.
```rust
enum Option<T> {
    None,
    Some(T),
}
```
Este enum es `Option<T>` y está definido en la [libreria estandar](https://doc.rust-lang.org/std/option/enum.Option.html). Están incluidas en el preludio, por lo que no es necesario meterla en el Scope explicitamente (Scope o alcance, definido en [[Pertenencia]]). Sus variantes también se incluyen en el preludio, por lo que se puede usar `Some` y `None` directamente sin el prefijo `Option::`.

La `<T>` es un parámetro de tipo genérico. Esto significa que la variante `Some` puede almacenar un tipo de dato de cualquier tipo, y que cada tipo concreto que se utilice en lugar de `T` hace el tipo `Option` de un tipo distinto. Esto se explica más detalladamente en el capítulo 10. 

## Por qué es útil utilizar None y Some
Si intentamos compilar el siguiente código, resultará en error:
```rust
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
```

Este error significa que Rust no entiende como sumar un `i8` y un `Option<i8>`, ya que son distintos tipos. El compilador debe asegurarse de que se maneja el caso en el que la variable de tipo `Option`  pueda ser nula para poder usarse. Es decir, hay que transformar la variable `Option<T>` a `T` para poder realizar operaciones de `T`.

Como norma general, para utilizar un valor `Option<T>` necesitas código que maneje cada variante, bien sea un valor `Some(T)` o un valor `None`. Para eso la expresión [[Match]] es un control de flujo que maneja estas variantes. Ejecutará código diferente en cada variante de un enum, y el código descrito puede utilizar dichos datos.
