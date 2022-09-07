# Preámbulo
Hay que tener en cuenta que rust es un lenguaje tipado estático (es decir, el tipo de dato está ligado a la variable), así que los tipos de las variables deben ser conocidos en la compilación.
El compilador puede inferir el tipo de dato, pero en ciertas situaciones es necesario describir el tipo que es. Un jemeplo sería el siguiente:
```rust
let guess: u32 = "42".parse().expect("Not a number!");
```
Si no se agregase la anotación del tipo (`u32`, que es un entero sin signo de 32 bits), saltaría un error en la fase de compilación.
# Scalar (Escalar)
Un tipo escalar representa un único valor. Rust cuenta con 4 tipos primarios de escalares:
## Integer
Un _integer_ es un numero sin componente fraccionada (un número entero). Puede ser positivo o negativo, si se añade signo al tipo o no. Los numeros con signo se almacenan utilizando "[complemento a dos](https://es.wikipedia.org/wiki/Complemento_a_dos)". Las variables pueden almacenar de -(2^{n-1}) hasta **2^{n-1} - 1**, ambos incluidos, para los de signo; para los sin signo, sería desde **0** hasta 
 **2^{n} -1**.

Se reconocen los siguientes tipos:
longitud|Con Signo|Sin Signo
--------|---------|---------
8 bits|i8|u8
16 bits|i16|u16
32 bits|i32|u32
64 bits|i64|u64
128 bits|i128|u128
arquitectura|isize|usize
El tamaño de `isize` y `usize` dependerán de la arquitectura del ordenador en donde se ejecute el programa: si es una arquitectura de 32 bits, será de 32; si es de 64 bits, 64. 

También se puede escribir literales de _integers_, a continuación se listan los posibles literales:
Literal|Ejemplo
----------|------
Decimal|**98_222**
Hex|**0x**ff
Octal|**0o**77
Binario|**0b**1111_0000
Byte (solo u8)|b'A'
Se pueden realizar [[Operaciones numéricas]] con estos típos de datos.
### Overflow de Entero
Si se produce un overflow (por ejemplo, tipo `u8` mantiene entre 0 y 255 y tratamos de asignarle el valor 256), dependiendo de la situación, Rust se comportará de distinta forma:
- Si está en modo depuración (debug), incluye checks para que, en caso de overflow, el programa paniquée ([[panic!]]), y termine la ejecución con información del error.
- Si está en modo "producción" (release), el programa no incluye dichos checks: en su lugar, realiza un "wrap-around" al mínimo de los valores que puede mantener (en este ejemplo, si se quiere almacenar 256 en un `u8` almacenará el valor **0**; si fuese 257, sería **1**, y así)

Se puede manejar explícitamente la posibilidad de overflow:
- Envolver en todos los modos con los métodos de `wrapping_*`, por ejemplo `wrapping_add`
- Devolver el valor `None` si hay un overflow con los métodos en `checked_*`
- Devolver el valor y un booleano indicando si hay overflow con los métodos de `overflowing_*`
- Saturar en los valores máximos y mínimos con los métodos de `saturating_*`
## Punto Flotante
Rust cuenta con dos tipos primitivos para numeros de punto flotante:
1. `f32` (flotante de 32 bits)
2. `f64` (flotante de 64 bits)
Por defecto es `f64`.
```rust
fn main() {
    let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```
Se pueden realizar [[Operaciones numéricas]] con estos típos de datos.

## Booleanos
Un tipo booleano solo tiene dos posibles valores: `true` o `false`. Los booleanos tiene 1 byte de tamaño, y en Rust se especifican con la palabra clave `bool`:
```rust
fn main() {
    let t = true;
	let f = false;
    let f: bool = false; // with explicit type annotation
}
```
Este tipo de valor se utiliza principalmente para condicionales y [[control de flujo]] del programa
## Caracter
Es el tipo de dato más primitivo para alfabéticos, en Rust. Se declara con la palabra `char`. Pueden ser, por ejemplo:
```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // with explicit type annotation
    let heart_eyed_cat = '😻';
}
```
Nótese que se especifica el literal con comillas simples ( **'** ), al contrario que con literales de cadena, los cales utilizan doble comilla ( **"** ).

El tamaño de un char es de 4 bytesy representa un valor de escalar [[Unicode]], lo que significa que puede representar mucho más que caracteres ASCII.
# Compound (Compuesto)
Los tipos compuestos pueden agrupar múltiples valores en un tipo. Existen dos tipos de datos compuestos en Rust: 
## 1. Tuplas
Una tupla es una forma genérica de agrupar un numero de valores con una variedad de tipos en un solo tipo compuesto.

Tienen un tamaño establecido: **Una vez declarado, no puede crecer o decrecer en tamaño**.

Creamos tuplas escribiendo una lista de tipos de valores entre paréntesis y con comas (observe ejemplo)
```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```
En el ejemplo, la variable `tup` está enlazada a la tupla completa, ya que se considera un único elemento compuesto.

### Acceso a los valores de una tupla
Para recuperar los valores individuales de la tupla, se puede utilizar "pattern matching" para desestructurar la tupla (observar ejemplo):

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {y}");
}
```
En el ejemplo, se crea primero la tupla y se enlaza a la variable tup. Despues se utiliza un patron con `let` para recuperar `tup` y transformarla en tres valores separados, `x`, `y`, `z` . Este proceso se llama **desestructuración**. Finalmente se printea el valor de `y` (6.4).

También se puede acceder al valor de una tupla directamente utilizando un punto (`.`), seguido del índice del valor al que se quiere acceder (observar ejemplo):
```rust
n main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

### Tupla unidad ("unit")
Una tupla sin valores se llama _unit_. Este valor y su tipo correspondiente se representan con `()` y representa un valor vacío o un valor de respuesta vacío. **Las expresiones devuelven, implicitamente, el valor de _unit_ si no devuelven ningún otro valor.**
## 2. Arrays
Un array es una coleccion de valores **que comparten el mismo tipo de dato**, es decir, todos los elementos de un array son del mismo tipo.

Se define un array con una serie de valores, contenidos entre corchetes, separados con comas (observe ejemplo):
```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```
Son ñutiles para cuando se quiere guardar la información en un _stack_ en lugar de un _heap_ (refierase al capitulo 4 -> [[Pertenencia]]), o cuando se quiera asegurar de que siempre hay un número fijo de elementos.

Un array no es tan flexible como un [[Vector]] (refierase al capitulo 8-> [[Colecciones]]), ya que estos pueden crecer o decrecer en tamaño de elementos, entre otras cosas. Si se duda entre utilizar un array y un vector, lo más probable es que sea necesario un vector.
Aun así, los arrays son más útiles cuando se sabe el número de elementos que no necesitan ser cambiados.

Se define el tipo de un array cn corchetes, donde se escribe el tipo de dato que almacena, y, separado con punto y coma (`;`), el nº de elementos en el array:
```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```
También se peude inicializar un array para que contenga el mismo valor para cada elemento especificando el valor inicial seguido de un punto y coma de la longitu del array:
```rust
let a = [3; 5]; // array de 5 elementos, cada uno con el valor 3
// equivalente a:
let a = [3,3,3,3,3];
```
### Acceso a los elementos
Dado que es un bloque de memoria de tamaño fijo, se puede acceder en el _stack_. Se pueden acceder a los elementos utilizando índices, tal que así:
```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0];
    let second = a[1];
}
```

Si se accede a un punto fuera del stack (en el ejemplo anterior, en el índice `9` ), el programa compilará correctamente, pero sufrirá un error de ejecución, sin permitir un acceso al indice de memoria.

## [[Enums]]