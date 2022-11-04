# Validar referencias con tiempos de vida
Los tiempos de vida (**_lifetime_**) son otro tipo de genérico ya usado. En vez de asegurar que un tipo tiene un comportamiento, asegurar que las referencias son válidas tanto como necesiten serlo.

En Rust, cada referencia tiene un tiempo de vida (_lifetime_), que es el alcance para el cual la referencia es válida. La mayor parte del tiempo, los tiempos de vida son implícitos y se infieren. Sólo se debe anotar los tipos cuando múltiples tipos son posibles. De manera similar, debemos anotar tiempos de vida cuando los tiempos de vida de referencias pueden ser relacionados de distintas formas. Rust requiere anotar las relaciones utilizando parámetros de tiepos de vida genéricos para asegurar que las referencias actuales se utilizan en tiempo de ejecución son completamente válidas.

# Prevenir referencias colgadas con tiempos de vida
El principal objetivo de los tiempos de vida son prevenir referencias sueltas (_dangling references_), las cuales causan a un programa referenciar información otra que la información que se pretende referenciar.
```rust
fn main() {
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```
Considere el siguiente programa, con un alcance (_scope_) externo y otro interno. El alcance externo declara una variable llamada `r` sin valor inicial, y el alcance interno declara una variable `x` con valor inicial de 5. Dentro del alcance interno, intentamos darle el valor de `r` una referencia a la variable `x`. Cuando el alcance interno finaliza, tratamos de mostrar el valor almacenado en `r`. Este código no compilará porque el valor que referencia `r` ha salido del alcance antes de poder utilizarlo:
```console
cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 | 
9 |     println!("r: {}", r);
  |                       - borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
```
La variable `x` no vive lo suficiente. El motivo es que `x` estará fuera del alcance cuando finaliza en la linea 7. Pero `r` sigue siendo válida para el alcance externo; dado que el alcance es mayor, se dice que "vive más". Si Rust permitiese funcionar este código, `r` estaría referenciando memoria que está deslocalizada cuando `x` sale del alcance, y cualquier cosa que tratasemos de hacer con `r` no funcionaria correctamente.

Como hace Rust para determinar que el código es inválido? Utilizando un _**borrow checker**_ (comprobador de préstamos).
# Comprobador de prestamos
El compilador de Rust cuenta con un comprobador de préstamos (_**borrow checker**_) que compara los alcances para determinar si todos los préstamos son válidos.
```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```
Este código es igual al anterior, pero utiliza comentarios para mostrar los tiempos de vida de las variables. Aquí, se anota el tiempo de vida de `r` con `'a` y el de `x` con `'b`. Como puede observar, el bloque interno de `'b` es mucho más pequeño que el bloque externo de `'a`. En tiempo de compilación, Rust compara el tamaño de los dos tiempos de vida y observa que `r` tiene un tiempo de vida en `'a` pero con una referencia a memoria en el tiempo de vida en `'b`. El programa se rechaza porque `'b` es más corto que `'a`: el sujeto de la referencia no vive tanto como la referencia.
```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```
Este código corrige la referencia suelta y compila sin errores. Aquí, `x` tiene un tiempo de vida `'b`, el cual es en este caso más largo que `'a`. Esto quiere decir que `r` puede referenciar a `x` porque rust sabe que la referencia en `r` es siempre válida mientras `x` es válida.
# Tiempos de vida genéricos en funciones
Vamos a escribir una función que devuelve, de dos trozos de String, el más largo. Esta función toma dos trozos de String y devuelve un único trozo de string. Despues de implementar la función `longest`, el código ejemplo debería imprimir en pantalla `the longest string is abcd`:
```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```
Hay que recalcar que queremos que la función reciba trozos de String, que son referencias, enn vez de cadenas de caracteres (_strings_), porque no queremos que la función `longest` tome posesion de sus parámetros.

Si tratamos de implementar la función `longest` de la siguiente forma, no compilará:
```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` due to previous error
```
El texto de ayuda revela que el tipo de retorno necesita un parámetro de tipo de vida en el porque Rust no puede reconocer si la referencia retornada es de `x` o de `y`, lo cual tampoco se sabe hasta el momento de ejecutarlo.

Cuando se define esta función, no se sabe que tipo de dato concreto será pasado. Tampoco se sabe el tiempo de vida concreto de las referencias, por lo que no se puede comprobar los alcances como en el ejemplo anterior para comprobar si la referncia que devuelve es siempre válida. El comprobador de prestamos no puede determinarlo, porque no sabe como los tiempos de vida de `x` e `y` son en relación con el tiempo de vida del valor de retorno. Para arreglar este error, hay que añadir tiempos de vida genéricos como parámetros que definan la relación entre las referencias para que el comprobador de pertenencia pueda realizar su análisis.

# Sintaxis de anotaciones de tiempos de vida
las anotaciones de tiempos de vida no cambia como de largo vive una referencia. En su lugar, describen las relaciones entre tiempos de vida de multiples referencias sin afectar en sus tiempos de vida propios. Así como las funciones pueden aceptar cualqueir tipo cuando la firma especifica un tipo de parámetro genérico, las funciones pueden aceptar referencias con cualquier tiempo de vida especificando un parámetro genérico de tiempo de vida.

Las anotaciones de tiempo de vida tienen una sintaxis peculiar: los nombre de los parámetros de tiempo de vida empiezan con un apóstrofe (`'`) y suelen utilizar minúsculas y nombres cortos (de una letra o así), como los tipos genéricos. Mucha gente utiliza el nombre `'a` para la primera anotacion de tiempo de vida. Se colocan anotaciones de parámetros de tiempos de vida después del `&` en una referencia, utilizando un espacio para separar la anotación del tipo de dato de la referencia:
```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```
Por si solo, una anotación de tiempo de vida no tiene mucho significado, porque su objetivo es decir a Rust como se relacionan entre ellos múltiples parámetros de tiempo de vida genéricos.

# Anotaciones de tiempos de vida en firmas de funciones
Para utilizar anotaaciones de tiempos de vida en la firma de una función, necesitamos declarar los parámetros del tiempo de vida genéricos dentro de `<>` entre el nombre de función y la lista de parámetros, como si fuese un tipo genérico de datos.

Queremos que la firma exprese la siguiente limitación: la referencia retornada será valida tanto tiempo como ambos parámetros sean válidos. Eso es la relación entre tiempos de vida de los parámetros y el valor de retorno. Llamaremos el tiempo de vida como `'a` y despues añadirlo a cada referencia, como se muestra en el siguiente código:
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
Este código debería compilar y producir el resultado que queremos cuando se utilice con la función `main` del ejemplo anterior donde se declara su uso.

La firma de la función informa a Rust de que para un tiempo de vida `'a`, la funcion toma dos parámetros, los cuales ambos son trozos de String que viven al menos tanto como el tiempo de vida `'a`. La firma de la función también informa a Rust de que el trozo de string retornado de la funcion vivirá tanto como el tiempo de vida `'a`. En la práctica, esto quiere decir que el tiempo de vida de la referencia retornada por la función es el mismo que el tiempo de vida más corto de los valores referidos de los argumentos de la función. Estas relaciones son lo que queremos que Rust utilice cuando analiza el código.

Hay que recordar que, cuando se especifica el tiempo de vida de los parámetros en la firma de la función, no estamos cambiando el tiempo de vida de ninguno de los valores transmitidos o devueltos. Simplemente se esta especificando que el comprobador de préstamos debe rechazar cualquier valor que no se adhiera a estas restricciones. También hay que destacar que la función `longest` no necesita saebr exactamente cuanto van a vivir los parámetros `x` e `y`, solo que un alcance puede ser sustituido por `'a` que satisfaga esta firma.

Cuando se anotan los tiempos de vida, las anotaciones van en la firma de función, no en el cuerpo. Las anotaciones de tiempos de vida se vuelven parte del contrato de la funcion, justo como los tipos en la firma. Tener una firma con el contrato de tiempo significa que el análisis del compilador en Rust puede ser más simple. Si hay un problema con la forma en la que una función esta anotada o en la forma que es llamada, los errores en compilación pueden apuntar a que parte de nuestro codigo y las concesiones con más precisión.

Cuando se pasan referencias concretas a `longest`, el tiempo de vida concreto que es sustituido por `'a` es parte del alcance de `x` que se solapa con el alcance de `y`. En otras palabras, el tiempo de vida genérico`'a` va a recibir el tiempo de vida concreto que es equivalente a el tiempo de vida más corto de `x` e `y`. Dado que se ha anotado la referencia de retorno con el mismo tiempo de vida `'a`, la referencia que  se retorna sera también válida para la duración de los tiempos de vida más cortos de `x` e `y`.

```rust
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```
En este ejemplo, `string1` será valido hasta el final del alcance externo, `string2` será válido hasta el final del alcance interno, y `result` referencia algo que es válido hasta el fin del alcance interno. Ejecutando este código se observa que el comprobador de préstamos lo valida; compilará e imprimirá `The longest string is long string is long`.

Ahora, vamos a ver un ejemplo que muestra que el tiempo de vida de la referencia en `result` debe ser la más corta de los dos valores. se mueve la declaracion de la variable `result` fuera del alcance interno pero se deja el asignamiento del valor de `result` en el alcnace con `string2`. Después se mueve el `println!` que utiliza `result` al alcance externo:
```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```
Este código no compila:
```console
 cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `string2` does not live long enough
 --> src/main.rs:6:44
  |
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^^^^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!("The longest string is {}", result);
  |                                          ------ borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
```
El error muestra que para que `result` sea válido para la sentencia `println!`, `string2` necesitaría ser valida hasta el final del alcance externo. Rust sabe esto porque se han anotado los tiempos de vida de los parámetros de la funcion y valores de retorno usando la misma variable de tiempo de vida `'a`.

# Pensando en términos de tiempos de vida
La forma en la que se necesita especificar parámetros de tiempo de vida depende en lo que realice la función. Por ejemplo, si se cambia la implementación de la función `longest` para retornar siempre el primer parámetro en vez de la cadena más larga, no sería necesario especificar el tiempo de vida en el parámetro `y`:
```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```
Este código compilará. Se especifica el parámetro de tiempo de vida `'a` para `x` y para el valor de retorno, pero no para el parámetro `y`, ya que el tiempo de vida de `y` no tiene relación con el tiempo de vida de `x` o del valor de retorno.

Cuando se devuelve una referencia de una función, los parámetros de tiempo de vida para el valor de retorno necesita coincidir con el tiempo de vida de uno de los dos parámetros. Si la referencia retornada no se refiere a uno de los parámetros, se debe referir a un valor creado en esta función. Sin embargo, esto sería una referencia colgada (_dangling reference_) porque el valor saldrá del alcance al final de la función. Considere el siguiente intento de implementación de la funcion `longest` que no compilará:
```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```
En esta función, aunque se ha especificado el tiempo de vida `'a` para el tipo de retorno, esta implementación fallará al compilar porque el valor de retorno no esta relacionado al tiempo de vida de los parámetros de ninguna forma. A continuación se detalla el mensaje de error:
```console
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0515]: cannot return reference to local variable `result`
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ^^^^^^^^^^^^^^^ returns a reference to data owned by the current function

For more information about this error, try `rustc --explain E0515`.
error: could not compile `chapter10` due to previous error
```
El problema está en que `result` sale del alcance y se limpia al final de la fucnión `longest`. También estamos tratando de retornar una referencia a `result` desde la función. No hay forma de que podamos especificar parámetros de tiempo de vida que cambien la rerferencia colgada, y Rust no permite crear una referencia colagada. En este caso, el mejor arreglo sería retornar un tipo de dato que tiene pertenencia en vez de una referencia para que la funcion de llamada sea la responsable para limpiar el valor.

Por último, la sintaxis de tiempos de vida consiste en conectar los tiempos de vida de varios parámetros y retornar valores de funciones. Una vez estén conectados, Rust tendrá suficiente información para permitir operaciones seguras en memoria y no permitir operaciones que crearían punteros colgados o otra forma de violar la seguridad de la memoria.
# Anotaciones de tiempos de vida en definiciones de Structs
Hasta ahora, las structs definidas todas mantenían tipos con pertenencia. Se pueden utilizar structs para contener referencias, pero en ese caso se necesitaría añadir una anotacion de tiempo de vida en cada referencia en la definición del struct.
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```
Este struct llamado `ImportantExcerpt` contiene un trozo de stric. Tiene un único campo `part` que contiene un trozo de string (_string slice_), el cual es una referencia. De la misma forma que con tipos de datos genéricos, se declara el nombre del parámetro de tiempo de vida genérico dentro de `<>` despues del nombre del struct para poder usarlo en el cuerpo de la definición del struct. Esta anotación significa una instancia de `ImportantExcerpt` no puede vivir más que la referencia que contiene en el cmapo `part`.

La función `main` crea una instancia de `ImportantExcerpt` que conteine una referencia a la primera oración del `String` propiedad de la variable `novel`. La información en `novel` existe antes que se instancie `ImportantExcerpt`. Además, `novel` no sale del alcance hasta después de que `ImportantExcerpt` sale del alcance, por lo que la referencia en la instancia de `ImportantExcerpt` es válida.
# Elisión de tiempos de vida
Se acaba de ver que para cada referencia hay un tiempo de vida, y que se debe especificar en los parámetros para funciones o sturcts que utilicen referncias. Sin embargo, en el capitulo 4 se utilizaba una funcion que compilaba sin utilizar anotaciones de tiempos de vida:
```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```
La razon de que esta funcion compile sin anotaciones de tiempo de vida es historica: en versiones más antiguas de Rust, este código no compilaría porque cada referencia necesita un tiempo de vida explicito. En versiones antiguas, la firma de la función hubiese sido la siguiente:
```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```
Después de escribir mucho código en Rust, el equipo de Rust encontro que los programadores estaban insertando las mismas anotaciones de forma reiterada en situaciones particulares. Estas situaciones son predecibles y siguen unos patrones deterministicos. Los desarrolladores programaron estos patrones en el código del compilador para que el comprobador de prestamos pudiese inferir los tiempos de vida en estas situaciones y no necesitasen anotaciones explícitas. Esto es relevante porque es posible que aparezcan más patrones deterministicos que se añadan al compilador. En el futuro, puede que incluso menos anotaciones de tiempos de vida sean necesarias.

Los patrones programados en el análisis de Rust de referencias se llaman reglas de elision de tiempos de vida (_lifetime elision rules_). Estas no son reglas que deba seguir el programador; son un set de casos particulares que el compilador considerará, y que si el código encaja en estos casos, no será necesario escribir tiempos de vida explícitos.

Las reglas de elisión no proveen una inferencia plena. Si rust deterministicamente aplica las reglas pero existe cierta ambigüedad de que tiempos de vida tienen las referencias, el compilador no adivinará que tiempo de vida de las referencias restantes deberían tener. En vez de adivinar, el compilador dará un error que se puede resolver añadiendo anotaciones de tiempos de vida.

Los tiempos de vida en los parámetros de una función o método se llaman tiempos de vida de input (_input lifetimes_), y los tiempos de vida en valores de retorno se llaman tiempos de vida de output (_output lifetimes_).

El compilador utiliza tres reglas para averiguar los tiempos de vida de las referencias cuando no tienen anotaciones explícitas. La primera regla se aplica a tiempos de vida de input, y la segunda y tercera regla se aplica a tiempos de vida de output. Si el compilador llega al fin de las tres reglas y hay aún referencias para las cuales no puede averiguar los tiempos de vida, el compilador se detendrá con un error. Estas reglas se aplican a definiciones de funciones asi como en bloques `impl`.

La primera regla es que el compilador le asigna un parámetro de tiempo de vida a cada parámetro que es una referencia. Dicho de otra forma, una funcion con un parámetro recibe un parámetro de tiempo de vida (`fn foo<'a>(x: &'a i32)`), una funcion con dos parámetros recibe dos parámetros de tiempo de vida separados (`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`), y así con todas.

La segunda regla es que, si hay exactamente un parámetro de tiempo de vida de input, el tiempo de vida es asginado a todos los parámetros de tiempos de vida de output: `fn foo<'a>(x: &'a i32) -> &'a i32`.

La tercera regla es que, si hay múltiples parámetros de tiempos de vida de input, pero uno de ellos es `self` o `&mut self` porque es un método, el tiempode vida de `self` es asignado a todos los parámetros de tiempo de vida de output. Esta tercera regla hace los métodos mucho más facil de leer y escribir porque son necesarios muchos menos símbolos.

Un ejemplo: imaginemos que somos el compilador. Aplicaremos estas reglas para averiguar los tiempos de vida de las referencias en la firma de la función `first_word`. La firma comineza sin ningún tiempo de vida asociada a las referencias:
```rust
fn first_word(s: &str) -> &str {
```
El compilador aplica la primera regla, que especifica que cada parámetr recibe su propio tiempo de vida. Ahora la firma sería, por ejemplo, la siguiente:
```rust
fn first_word<'a>(s: &'a str) -> &str {
```
La segunda regla se aplica porque existe un tiempo de vida de input. La segunda regla especifica que el tiempo de vida de un parámetro de entrada se asigna al tiempo de vida del output, asi que la firma ahora es la siguiente:
```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```
Ahora, todas las referencias en esta funcion tienen tiempos de vida, y el compilador puede continuar con su análisis sin necesidad de que el programador anote los tiempos de vida en esta firma de función.

Veamos otro ejemplo, utilizando la función `longest` que no tenía ningun parámetro de vida cuando trabajamos con el:
```rust
fn longest(x: &str, y: &str) -> &str {
```
Aplicamos la primera regla: cada parámetro recibe su propio tiempo de vida. Esta vez son dos parámetros, por lo que tenemos dos tiempos de vida:
```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```
Se puede observar que la segunda regla no se aplica porque hay más de un tiempo de vida de input. La tercera regla tampoco se aplica, porque `longest` es una funcion en vez de un método, así que ninguno de los parámetros es `self`. Después de aplicar las tres reglas, aun no se ha averiguado cual es el tiempo de vida del valor de retorno. Esto es el motivo de por que recibimos un error intentando compilar el código en esta funcion: el compilador siguió los pasos de reglas de elisión de tiempos de vida pero no pudo averiguar cuales son todos los tiempos de vida de las referencias en la firma.

Dado que la tercera regla solo se aplica en firmas de métodos, vamos a ver los tiempos de vida en este contexto para ver por que con la tercera regla no es necesario anotar los tiempos de vida en firmas de métodos tan comunmente.


# Anotaciones de tiempos de vida en definición de métodos
Cuando se implementan métodos en un struct con tiempos de vida, utilizamos la misma sintaxis que el tipo genérico mostrado anteriormente. Donde se declara y utiliza los parámetros de tiempo de vida depende de si están relacionados a los campos del struct o los paráemtros y valores de retorno del método.

Los tiempos de vida para campos del struct siempre deben ser declarados después de la palabra clave `impl` y despues usarlo despues del nombre del struct, porque esos tiempos de vida forman parte del tipo del struct.

En firmas de métodos dentro de un bloque `impl`, las referencias pueden estar enlazadas a los tiempos de vida de las referencias en los campos del struct, o puede que sean idependientes. Además, las reglas de elision de tiempos de vida normalmente hacen que no sean necesarios los tiempos de vida en la firma de los métodos. Vamos ver una serie de ejemplos utilizando un struct llamado `ImportantExcerpt`.

Primero, utilizaremos un método llamado `level` cuyo parámetro es una referencia al `self` y cuyo valor de retorno es un `i32`, el cual no es una referencia:
```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```
La declaración del parámetro de tiempo de vida después de `impl` y su uso a continuación del nombre del tipo son necesarios, pero no es necesario anotar el tiempo de vida de la referencia al `self` debido a la primera regla de elisión.

Aqui se muestra un ejemplo de como se aplica la tercera regla de elisión:
```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```
Hay dos inputs de tiempos de vida, así que Rust aplica la primera regla de elisión y le da a `&self` y `announcement` sus propios tiempos de vida. Después, porque uno de los parámetros es `&self`, el tipo de retorno recibe el tiempo de vida de `&self`, y todos los tiempos de vida ya han sido registrados.
# Tiempo de vida estático
Un tiempo de vida especial es `'static`, el cual denota que la referencia afectada puede vivir el tiempo total de la duración del programa. Todos los literales de cadenas tienen un tiempo de vida `'static`, el cual se puede anotar de la siguiente forma:
```rust
let s: &'static str = "I have a static lifetime.";
```
El texto de esta cadena se almacena directamente en el binario del programa, el cual está disponible siempre. Debido a esto, el tiempo de vida de todos los literales es `'static`.

Puede que se vea la sugerencia de utilizar `'static` en los mensajes de error. Pero antes de especificar `'static` como tiempo de vida para una referencia, hay que pensar si la referencia que tienes debe vivir toda la duración del programa o no, o si se quiere que dure toda la duración. La mayoria de veces, un mensaje de error sugiriendo el tiempo de vida `'static` resulta de intentar crear una referencia suelta (_dangling reference_) o una discordancia de los tiempos de vida disponibles. En estos casos, la solución es reslolver estos problemas, en vez de especificar el tiempo de vida `'static`.
# Parámetros de tipo genérico, enlaces de rasgos, tiempos de vida, juntos
Vamos ver un ejemplo de la sintaxis de especificar parámetros genéricos, rasgos, y tiempos de vida en una función:
```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
Esta es la función `longest` definida previamente que retorna el más largo de dos pedazos de string (_string slices_). Pero ahora cuenta con un parámetro extra llamado `ann` del tipo genérico `T`, el cual puede ser rellenado con cualquier tipo que implemente el rasgo `Display` si es necesario. Debido a que los tiempos de vida son genéricos, las declaraciones del tiempo de vida `'a` y el parámetro genérico `T` van en la misma lista dentro de diples (<>) después del nombre de la función.