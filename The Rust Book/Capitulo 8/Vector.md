# Definición
un tipo `Vec<T>` perimte almacenar más de un valor en una estructura de datos que pone los valores uno al lado del otro en memoria.

Solo pueden almacenar datos del mismo tipo.

Son útiles cuando se quiere tener una lsita de objetos, tales como las lineas de texto en un fichero o los precios de los objetos en una lista de la compra.

# Creación
Para crear un vector nuevo vacío, se llama a la funcion `Vec::new`, declarando el tipo de dato que va almacenar
```rust
   let v: Vec<i32> = Vec::new();
```

También se puede crear un vector con valores iniciales. Rust incluye la macro `vec!`, la cual crea u nuevo vector que contiene los valores que se le asignan:
```rust
    let v = vec![1, 2, 3];
```
Como se puede ver, no es necesario tampoco declarar el tipo de dato a almacenar, Rust lo infiere y asigna el tipo de dato por defecto (en este ejemplo, sería `i32`)

# Actualizar un vector
Para crear un vector y añadir elementos a este, es necesario declararlo con la palabra clave `mut`, y para agregar valores utilizamos el método `push`:
```rust
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
```

# Leer valores de un vector
Existen dos formas de referenciar un valor almacenado en un vector:
1. A través de índices, utilizando `&` y `[]` recuperamos una referencia al elemento del valor del indice
2. Utilizando el método `get` con el índice como parámetro, recuperamos un valor de tipo `Option<&T>`, el cual podemos usar con un match
```rust
let v = vec![1, 2, 3, 4, 5];
	// acceder con índices a un valor
    let third: &i32 = &v[2];
    println!("The third element is {}", third);
	// acceder con método get(i)
    let third: Option<&i32> = v.get(2);
    match third {
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }
```

Existen dos formas de acceder para escoger cómo se comporta el programa en el caso de utilizar un índice fuera de los límites del vector, bien sea con un [[panic!]] (opción 1) o con un None controlado con un [[Match]] (opcion 2)

Cuando existe una referencia válida, el comprobador de prestamos aplica las reglas de [[Pertenencia]] para asegurar que la referencia y cualquier otra referencia a los contenidos del vector sea válida.
```rust
 let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {}", first);
```
```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 | 
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 | 
8 |     println!("The first element is: {}", first);
  |                                          ----- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `collections` due to previous error
```

# Iterar sobre los valores de un Vector
Para recorrer los valores de un vector de uno en uno se utiliza un `for` loop en lugar de recorrer cada índice:
```rust
// referencias inmutables
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{}", i);
    }
```
```rust
// referencias mutables
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
```
Para modificr el valor en las mutables, hay que utilizar el operador dereferenciador `*` para obtener el valor en `i` antes de utilizar el operador de suma.

# Utilizar [[enums]] para almacenar múltiples tipos
Los vectores solo pueden almacenar valores del mismo tipo, pero se puede utilizar un enum con sus variantes enlazadas a un tipo de valor para almacenar distintos tipos de datos en un vector:
```rust
   enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
```

Rust necesita saber que tipos serán almacenados en el vector en tiempo de compilación para saber cuanta memoria en el [[Heap]] será necesaria para cada elemento. Con un enum y una expresión `match` Rust puede asegurarse en tiempo de compilación que cada posibilidad sea manejada, tal como se vio en el cap. 6.