# Referencias y préstamos
Una referencia es similar a un **puntero**. Es una dirección que podemos seguir para acceder la información almacenada en esa dirección; esa información está adueñada por otra variable.

A diferencia de un puntero, una referencia garantiza apuntar a un valor válido de un tipo particular durante la vida de la referencia.

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```
_Nótese que la función `calculate_length` recibe una referencia, no el tipo de variable en si_

El caracter `&` representa _referencia_, y permite referirse a un valor sin tener pertenencia de este. 

## Dereferencia
Existe un operador contrario a `&`, llamado dereferenciador (con el símbolo `*`).

# Referencias mutables
```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```
Para crear una referencia mutable, se utiliza la combinación `&mut`.

Las referencias mutables cuentan con una restricción: si existe una referencia mutable a un valor, **no puede haber otra referencia a ese valor**. Esta restricción existe para evitaruna _data race_; esto ocurre cuando sucede una de estas tres condiciones:
1. Dos o mas punteros acceden al mismo dato al mismo tiempo
2. Al menos uno de los punteros esta siendo utilizado para escribir información
3. No hay ningun mecanismo usado para sincronizar el acceso a la información

Para permitir multiples referencias mutables, se puede utilizar `{}` para crear un nuevo scope:
```rust
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
```

Tampoco se puede tener una referencia mutable mientras hay una referencia inmutable al mismo valor:

```rust
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
```

Hay que recalcar que el alcance de una referenica empieza donde es declarada, y termina donde se utiliza por ultima vez: esto permite que exista código como el siguiente:

```rust
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!("{}", r3);
```

## Referencias Colgadas (_Dangling References_)
En lenguajes con punteros, se da el caso de un puntero que referencia una sección de memoria que ha sido cedida a otro. En rust, el compilador garantiza que las referencias nunca serán colgadas: si existe unar eferencia, el compilador asegura que la información no saldrá del alcance antes de que la referencia lo haga.