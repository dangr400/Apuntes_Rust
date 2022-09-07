# Definición
_Ownership_, o Pertenencia, es un conjunto de reglas que gobiernan como un  programa de Rust gestiona la memoria.

Todos los programas deben gestionar como se maneja la memoria mientras se ejecuta el código: Algunos utilizan un colector de basura que comprueba segmentos de código que no se utilizaron en "X" tiempo, mientras que otros reservan y liberan memoria explicitamente.

En Rust, la memoria se gestiona a través de un sistema de pertenencia con un set de reglas que el compilador comprueba. Si alguna de las reglas es violada, el programa no compilará. También, ninguna de las capacidades de la pertenencia retrasarán la velocidad del programa en ejecución.

Este concepto está relacionado con el [[Stack]] y [[Heap]]. El stack almacena valores en el orden que los recibe, y los elimina al revés (el último recibido será el último en eliminar). En el heap, se busca un espacio libre para la información pedida, y se retorna un puntero a dicha posición de memoria.

# Reglas
Estas son las reglas que sigue un programa de Rust respecto a la pertenencia:
- Cada valor en Rust tiene un dueño (_owner_)
- Solo puede haber un dueño de cada vez
- Cuando el dueño está fuera de alcance (_out of scope_), se elimina el valor

# Alcance de una variable
```rust
	{                      // s is not valid here, it’s not yet declared
        let s = "hello";   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid
```

Existen dos puntos clave respecto a una variable:
1. Cuando la variable (s) entra en el alcance (_scope_), es válida
2. Permanece válida hasta que salga del alcance (_scope_)

## Aplicado a un tipo `String`
Los [[Tipos de datos]] mostrados hasta ahora tienen un tamaño conocido, por lo que pueden introducirse en el stack y sacados de este cuando su alcance se termina, y pueden ser copiados para crear una instancia nueva. 

Para observar el comportamiento del [[Heap]], utilizamos el tipo `String`. Para las situaciones en las que se quiera almacenar una secuencia de caracteres de tamaño variable existe `String`: este tipo de dato gestiona información almacenada en el [[Heap]], el cual es posible de almacenar una cantidad de texto desconocida en tiempo de compilación. 

Se puede crear un `String` utilizando la función `from`:
```rust
let s = String::from("hello");
```
El operador `::` permite nombrar la funcion que se encuentra en el tipo de dato en lugar de utilizar el nombre completo. (La sintaxis se analiza más detenidamente en el Capítulo 5, [[Sintaxis de métodos]])
Este tipo de cadena de caracteres puede ser mutable:
```rust
    let mut s = String::from("hello");

    s.push_str(", world!"); // push_str() appends a literal to a String

    println!("{}", s); // This will print `hello, world!`
```

## Memoria y asignación de esta
En caso de un literal, sabemos el contenido en tiempo de compilación. Esto hace que el texto se codifique directamente en el ejecutable.

Con el tipo `String`, para poder tener un texto mutable que pueda crecer, es necesario almacenar una cantidad de memoria en el heap para almacenar el contenido, cuyo tamaño es desconocido en tiempo de compilación. Esto quiere decir que:
- La memoria debe ser pedida al asignador de memoria en tiempo de ejecución
- Es necesaria una forma de devolver la memoria al asignador cuando terminemos el uso de dicho `String`

El primer punto se logra al llamar a `String::from`, mientras que la segunda se cumple automaticamente una vez que termina el _scope_ de la variable.

Cuando una variable sale del _scope_, Rust llama a una función llamada `drop` para devolver la memoria al asignador. Esto es equivalente a los patrones RAII (Resource Acquisition Is Initialization)

### Interacciones: Move
```rust
let s1 = String::from("hello");
    let s2 = s1;

    println!("{}, world!", s1);
```
En este caso el programa no compilará correctamente porque s1 deja de ser válida como variable para evitar un _double free error_ (es una copia parcial), un error debido a que s2 y s1 son punteros a la misma dirección de memoria, **s2 no es una copia del valor en s1**. 
Rust nunca crea automaticamente copias profundas (_deep copy_), por lo que copiar variables de este modo es poco costoso en tiempo de ejecución.

### Interacciones: Clone
Si queremos hacer una copia profunda (_deep copy_) de una variable, se puede llamar al método `clone` para crear un clon de la variable (se analiza mas detenidamente en Capitulo 5)

```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
```

### Copias : información en el stack
```rust
    let x = 5;
    let y = x;

    println!("x = {}, y = {}", x, y);
```
El ejemplo mostrado en este bloque, a pesar de ir en contra de lo explicado previamente, **funciona**. Esto se debe a que los tipos de datos con un tamaño conocido en tiempo de compilación se almacenan en su totalidad en el [[Stack]], por lo que crear copias de dichas variables es rápido.

### Rasgo copy (`copy trait`)
Existe una anotación especial en Rust llamada rasgo `Copy`, el cual se puede poner en tipos que se almacenan en el [[Stack]]. Los tipos que implementan este rasgo hacen que las variables que lo usan no se mueven, sino que se copian, haciendolas válidas aun después de asignarlas a una nueva variable.

## Pertenancia en funciones

Los mecanismos de pasar valores a una funcion son similares a los de asignar un valor a una variable: Pasar un valor a una función la movera o copiará, justo como una asignación

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```
En este ejemplo, si tratamos de usar `s` despues de la llamada a `takes_ownership`, Rust lanzaría un error en tiempo de compilación

### Valores de retorno y alcance
Retornar valores también transfiere la pertenencia:

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

La pertenencia de una variable sigue el mismo patrón siempre: **asignar un valor a otra variable lo mueve.** Cuando la variable que contiene información del [[Heap]] sale de su alcance, el valor se limpiará con `drop`, salvo que la pertenencia del valor se haya movido a otra variable.

