# Definición
Un struct, o estructura, es un tipo de dato customizado que permite empaquetar y nombrar valores relacionados que crean un grupo con sentido.

Un simil con lenguajes orientados a objetos, un struc es como  los atributos de un objeto

# Definir e Instanciar structs
Un struct es similar a una tupla, en que ambos mantienen múltiples valores relacionados.
A diferencia de una tupla, en una structura se peude nombrar cada pieza de información para aclarar que significa cada valor.

Para definir un Struct, se utiliza la palabra clave `struct` y el nombre de la estructura. Después, dentro de las llaves (`{}`) se define los nombres y tipos de de información, llamados campos 
 o _fields_:
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

Para utilizar una struct después de definirla, se crea una _instancia_ de dicha struct, especificado valores concretos para cada campo:
```rust
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
}
```

Para acceder a un valor específico, se utiliza notación de punto:
```rust
fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

Nótese que la instancia completa, para ser mutable algun campo, debe ser mutable en su plenitud; Rust no permite marcar ciertos campos como mutables.

## Utilizando el init de campo acortado
Si el parámetro y el nombre del campo del struct son iguales, se puede utilizar el _field init shorthand_ para no repetir el nombre de los campos:
```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

## Crear instancias desde otra instancia con sintaxis de actualización de struct (Struct Update Syntax)
```rust
fn main() {
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
}
```
El código de ejemplo crea una nueva instancia con los valores de otra instancia. Se puede lograr el mismo objetivo con menos código:
```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```
el valor `..user1` debe ir al final para especificar que cualquier campo que falte por asignar recoja el valor de la instancia (en este caso, user1)

## Utilizar tuplas de struct sin campos de nombre para crear diferentes tipos
Rust soporta el uso de tuplas de struct (_tuple struct_). Tienen el añadido significado del nombre de struct pero sin nombres asociados a los campos, tan solo tienen el tipo de dato de los campos.
Se definen con la palabra clave `struct`, seguido del nombre y los tipos de datos que almacena:
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

## "Unit-like" Structs sin campos
Se puede definir un struct sin ningun tipo de dato asociado. Este struct se llama _unit-like_ porque se comporta similarmente a `()`, el tipo unitario de tupla.
```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```
