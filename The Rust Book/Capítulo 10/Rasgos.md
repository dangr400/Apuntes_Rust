# Definición
Un rasgo (_trait_) define funcionalidad a un tipo particular y que puede compartirse con otros tipos. Se pueden utilizar rasgos para definir comportamiento común de forma abstracta. Se pueden utilizar limite de rasgos (_trait bounds_) para especificar que un tipo genérico puede ser de cualquier tipo que tenga cierto comportamiento

Los rasgos son similares a las interfaces (_interfaces_) en otros lenguajes con ciertas diferencias.

# Definir un rasgo
El comportamiento de un rasgo consiste de los métodos que podemos llamar en ese tipo. Diferentes tipos comparten el mismo comportamiento si podemos llamar a los mismos métodos en todos esos tipos.

Las definiciones de tipos son una forma de agrupar métodos juntos para definir un set de comportamientos necesarios para lograr algún propósito.

Por ejemplo, digamos que hay múltiples structs que tienen varios tipos y cantidad de texto: un struct `NewsArticle` que contiene una noticia en cierto lugar y un `Tweet` que puede tener como máximo 280 caracteres con metainformación adicional que indica si es un nuevo tweet, un retweet o una respuesta a otro tweet.

Queremos hacer una libreria agaregadora de info llamada `aggregator` que pueda mostrar resúmenes de datos que pueden estar almacenado en una instancia de `NewsArticle` o de `Tweet`. Para realizarlo, necesitamos hacer un resumen para cada tipo, y pediremos el resumen llamando al método `summarize` en cada instancia. A continuación se muestra la definición del rasgo público `Summary` que expresa este comportamiento:
```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```
Aquí se declara el rasgo usando la palabra clave `trait` y despeus el nombre del rasgo, en este caso `Summary`. También hemos definido el rasgo como `pub` para que las cajas (_crates_) que dependan de esta caja puedan usar este rasgo también. Dentro de las llaves (`{}`), se declara el método firma que describe los comportamientos de los tipos que implementan este rasgo, en este caso es `fn summarize(&self) -> String`.

Después del método firma, en vez de proveer una implementación entre llaves, se utiliza un punto y coma. Cada tipo que implemente este rasgo debe proveer su propio comportamiento para el cuerpo del método. El compilador obligara a que cualquier tipo que cuente con el rasgo `Summary` tenga el método `summarize` deefinido con la misma firma (en este caso, `fn summarize(&self) -> String`).

Un rasgo puede contar con múltiples métodos en su cuerpo; se listan una por linea y cada linea termina en punto y coma (`;`).

# Implementar un rasgo en un tipo
Ahora que esta definido las firmas para los métodos en el rasgo `Summary`, se puede implementar en los tipos del agregador de info:
```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```
Aquí se muestra una implementación del rasgo `Summary` en las structs de `NewsAritcle` y `Tweet`.
Implementar un rasgo en un tipo es similar a implementar funcione regulares. La diferencia está en que después de `impl` se escribe el nombre del rasgo a implementar, seguido de la palabra clave `for`, y después se especifica el nombre del tipo al que queremos implementarle el rasgo. Dentro del bloque de `impl` se pone los mñetodos firma que el rasgo define. En vez de añadir puntos con coma (`;`) despues de cada firma,  se utilizan llaves (`{}`) y se rellena el cuerpo del método con el comportamiento específico que queremos que los métodos del rasgo implementen.

Ahor que la librería ha implementado el rasgo `Summary` en `NewsArticle` y `Tweet`, usuarios de la caja pueden llamar a los métodos del rasgo en instancias de `NewsArticle` y `Tweet` de la misma forma que se llaman a métodos regulares. La única diferencia es que el usuario debe traer el rasgo al alcance (_scope_) así como a los tipos. Se muestra un ejemplo de como una caja binaria utilizaría la librería `aggregator`:
```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

Otras cajas que dependan de la caja `aggregator` pueden traer al alcance el rasgo `Summary` para miplementar `Summary` en sus tipos. Una restrición que hay que destacara es que solo se puede implementar un rasgo si al menos uno de los rasgos o el tipo es local a nuestra caja. Por ejemplo, se puede implementar rasgos de la librería estandar como `Display` en un tipo propio como `Tweet` como parte de la funcionalidad de la caja `aggregator`, porque el tipo `Tweet` es local a nuestra caja `aggregator`. También se puede implementar `Summary` en `Vec<T>` en la caja `aggregator`, porque el rasgo `Summary` es local a nuestra caja `aggregator`.

Pero no se puede implementar rasgos externos en tipos externos. Por ejemplo, no se puede implementar el rasgo `Display` en `Vec<T>` en nuestra caja `aggregator`, porque `Display` y `Vec<T>` son ambos definidos en la librería estandar y no son locales a nuestra caja `aggregator`. Esta restricción es parte de una propiedad llamada coherencia (_coherence_), y más especificamente la "regla del huerfano" (_orfan rule_), nombrada así porque el padre tipo no esta presente. Esta regla asegura que el código de otra gente no pueda romper tu código y viceversa. Sin esta regla, dos cajas podrían implementar el mismo rasgo para el mismo tipo, y Rust no sabría que implementación utilizar.

# Implementaciones por defecto
A veces es útil tener un comportamiento por defecto par algunos o todos los métodos enun rasgo en vez de requerir implementaciones para todos los métodos en todos los tipos. Después, a medida de que se implementa el rasgo en un tipo particular, se peude sobreescribir el comportamiento por defecto de cada método.
```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```
En este caso, se especifica una cadena por defecto para el método `summarize` del rasgo `Summary` en vez de solo definir la firma del método.

Para utilizar una implementación por defecto para resumir instancias de `NewsArticle`, se especifica un bloque `impl` vacío con `impl Summary for NewsArticle {}`.

Aunque no se definen el método `summarize` en `NewsArticle` directamente, se provee una implementación por deffecto y especificado que `NewsArticle` implementa el rasgo `Summary`. Como resultado, se puede llamar el método `Summarize` En una instancia de `NewsArticle`:
```rust
let article = NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("Iceburgh"),
        content: String::from(
            "The Pittsburgh Penguins once again are the best \
             hockey team in the NHL.",
        ),
    };

    println!("New article available! {}", article.summarize());
```
La salida del código será `New article available! (Read more...)`.
Crear una implementación por defecto no necesita que se cambie nada sobre la implementación de `Summary` en `Tweet` en el ekemplo anterior de `Tweet`. Esto se debe a que la sintaxis de sobreescribir una implementación por defecto es la misma que la sintaxis de implementar un método de un rasgo que no tiene implementación por defecto.

Las implemenaciones por defecto pueden llamar otros métodos del mismo rasgo, incluso si esos métodos no tienen una implementación por defecto. De esta forma, un rasgo puede proveer funcionalidad util y solo requiere de implementadores para especificar una parte de esta funcionalidad. Por ejemplo, se puede definir en el rasgo `Summary` el método `summarize-author` cuya implementación es necesaria, y después definir el método `summarize` par tener una implementación que llame al método `summarize_author`:
```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```
Para utilizar esta versión de `Summary`, es necesario definir `summarize_author` cuando se implemente el rasgo en un tipo:
```rust
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```
Después de definir `summarize_author`, se puede llamar `summarize` en instancias del struct `Tweet`, y la implementaión por defecto de `summarize` llamará a la definición de `summarize_author` que se provée. Dado que está implementado el método `summarize_author`, el rasgo `Summary` da el comportamiento del método `summarize` sin necesitar escribir más código:
```rust
let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
```
# Rasgos como parámetros
A continuación se va explorar como utilizar rasgos para definir funciones que aceptan diferentes tipos de datos. Utilizaremos el rasgo `Summary` implementado previamente para definir una función `notify` que llama al método `summarize` en su parámetro `item`, el cual es algún tipo que implementa el rasgo `Summary`:
```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```
En vez de un tipo concreto para el parámetro `item`, se especifica la palabra clave `impl` y el nombre del rasgo. Este parámetro acepta cualquier tipo que implemente el rasgo especificado. En el cuerpo de `notify`, se puede llamar cualquier método de `item` que venga del rasgo `Summary`, tal como `summarize`. Se puede llamar a `notify` y pasar una instancia de `NewsArticle` o de `tweet`. El código que llame a la función con cualquier otro tipo no compilará dado que no implementan dicho rasgo.

## Sintaxis de enlace de rasgos
la sintaxis de `impl Rasgo` funciona para casos directos, pero en realidad es azucar sintáctico (_syntax sugar_)  para una forma más larga conocida como enlace de rasgos (_trait bound_), la cual es así:
```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```
Este ejemplo es equivalente al previo, pero más verboso.  Se localiza el enlace con la declaración de un tipo genérico después de doble punto (`:`) y dentro de llaves picudas (`<>`).

La sintaxis de `impl Rasgo` es conveniente y sirve para tener un código conciso en casos simples, mientras que la versión completa de enlace de rasgos puede expresar más complejidad en otros casos. Por ejemplo, se puede tener dos parámetros que implementen `Summary`. De est forma con `impl Rasgo` se vería así:
```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```
Este es apropiado si se desea que esta funcion permita `item1` e `item2` tener dos tipos diferentes (siempre y cuando implementen el rasgo `Summary`). Si se quiere forzar ambos parámetros para tener el mismo tipo, es necesario usar un enlace de rasgo, como este:
```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```
El tipo genérico `T` especificado como el tipo de `item1` e `item2` limita la funcion de tal forma que el tipo concreto de valor para los argumentos debe ser el mismo.
## Especificar múltiples enlaces con la sintaxis `+`
También se puede especificar mas de un rasgo enlazado. Digamos que se quiere que `notify` utilice el formateo del rasgo `Display` así como `summarize` en `item`: se especifica en la definición de `notify` que `item` debe implementar ambos rasgos. Para esto se utiliza la sintaxis `+`:
```rust
pub fn notify(item: &(impl Summary + Display)) {
```
Esta sintaxis también es válida con enlaces a rasgos con tipos genéricos:
```rust
pub fn notify<T: Summary + Display>(item: &T) {
```
Con los dos rasgos especificados, el cuerpo de `notify` puede llamar `summarize` y utilizar `{}` para formatear `item`.

## Enlaces más claros con clausulas `where`
Utilizar demasiados enlaces de rasgos tiene sus desventajas. Cada genérico tiene su propio enlace, así que funciones con múltiples tipos genericos pueden contener mucha información de enlaces de rasgos entre el nombre de la función y la lista de parámetros, volviendo dificil la lectura de la firma de la función:
```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```
Por este motivo, Rust tiene una sintaxis alterna para especificar enlaces de rasgos dentro de una cláusula `where` después de la firma de la función:
```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```
# Retornar tipos que implementan rasgos
También podemos utilizar la sintaxis `impl Rasgo` en la posición de retorno para devlover algún tipo que implemente un rasgo:
```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```
Con el, se especifica que `returns_summarizable` devuelve algun tipo que implementa el rasgo `Summary` sin nombrar un tipo concreto. En este caso, devuelve un `Tweet`, pero el código que lo llama no necesita saberlo.

La capacidad de especificar un tipo de retorno solo por el rasgo que implementa es muy util en el contexto de clausulas e iteradores (se cubre en el cap. 13). las clausulas e iteradores crean tipos que solo el compilador conoce o tipos que son muy largos para especificar. La sintaxis de `impl Rasgo` permite especificar concisamente que una funcion retorna algún tipo que implemente el rasgo `Iterator` sin necesidad de escribir un tipo largo.

Sin embargo, solo se puede utilizar `impl Rasgo` si se devuelve un único tipo. Por ejemplo, este código que devuelve o bien un `NewsArticle` o un `Tweet` con el tipo de retorno especificado como `impl Summary` no funcionará:
```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
}
```
En el cap. 17 se explora como escribir una función con este comportamiento.

# Utilizar enlaces para implementar condicionalmente métodos
Al utilizar un enlace de rasgos con un bloque `impl` que utiliza parámetros de tipo genérico, se puede implementar métodos condicionalmente para tipos que implementan los rasgos especificados. Por ejemplo:
```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```
el tipo `Pair<T>` siempre implementa la función `new` para devolver una instancia de `pair<T>`. Pero en el siguiente bloque `impl`, `Pair<T>` solo implementa el método `cmp_display` si su tipo interno `T` implementa el rasgo `PartialOrd` que habilita la comparación y el rasgo `Display` que habilita mostrar por pantalla.

También se puede implementar condicionalmente un rasgo para cualquier tipo que implemente un rasgo. Las implementaciones de un rasgo o cualquier tipo que  satisfaga el enlace de rasgo se llaman implementaciones en blanco (_blank implementations_) y son extensivamente utilizadas en la librería estandar de Rust. Por ejemplo, la librería estandar implementa el rasgo `ToString` en cualquier tipo que implemente el rasgo `Display`. El bloque `impl` en la librería estandar se ve similar a este código:
```rust
impl<T: Display> ToString for T {
    // --snip--
}
```
Dado que la librería estandar cuenta con esta implementación en blanco, se peude llamar al método `to_string` definido en el rasgo `ToString` en cualquier tipo que implemente el rasgo `Display`. Por ejemplo, se puede transformar enteros (_integers_) en sus correspondientes valores `String` porque los enteros implementan `Display`:
```rust
let s = 3.to_string();
```
Las implementaciones en blanco aparecen en la documentación para el rasgo en la sección "Implementors".

# Nota final
Los rasgos y enlaces de rasgos permiten escribir código que utiliza tipos genéricos como parámetros para reducir duplicación, pero tambien especifican al compilador que queremos un tipo genérico que tenga cierto comportamiento. El compilador puede utilizar la información del rasgo enlazado para comprobar que todos los tipos concretos usados con nuestro código provean un comportamiento correcto. En lenguajes dinámicos, se recibiría un error en tiempo de ejecucion si se llama a un método en un tipo que no define el método. Pero Rust mueve estos erroresen tiempo de compilación para forzar a arreglar los problemas antes de que pueda ejecutarse el código. Adicionlamente, no hay que escribir código que comprueba el comportamiento en tiempo de ejecución porque ya se comprueba en tiempo de compilación.