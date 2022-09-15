# Tipos de error
Existe dos tipos de errores en Rust:
1. [[Errores recuperables]]
2. [[Errores inrecuperables]]

# Escoger el tipo de error
## `panic!` o no `panic!`
¿Cómo se decide cuando llamar a `panic!` y cuando retornar un `Result`?

Cuando un código paniquea, no hay forma de recuperarlo. se podría llamar a `panic!` para cualquier situación de error, ya bien haya posibilidad de recuperar o no, pero entonces se está tomando la decision de que la situación es irrecuperaeble de parte del código que llama.

Cuando se escoge retornar un `Result`, le das al código que llama opciones. El código que ejecuta puede escoger tomar la oportunidad de recuperarse de forma apropiada a la situación, o peude decidir que un valor `Err` es en este caso irrecuperable, por lo que puede llamar a `panic!` y tornar el error recuperable en uno irrecuperable. Por lo tanto, es mejor retornar `Result` como opción primaria al definir una función que puede fallar.

En situaciones como código prototipo, tests o ejemplos, es mas apropiado un código que devuelva panics en lugar de retornar un `Result`.

## Ejemplos, código prototipo y Tests
Cuando se escribe un ejemplo para ilustrar algún concepto, incluir codigo robusto para manejo de errores puede hacer el ejemplo menos claro. En ejemplos, se entiende que una llamada al método como por ejemplo `unwrap` puede paniquear.

También, los métodos `unwrap` y `expect` son muy útiles cuando se anda con prototipos, antes de escoger como manejar errores. Dejan marcadores claros en el código de cuando estás listo para hacer el programa más robusto.

Si la llamada a un método falla en un test, es preferible que el conjunto del test falle, incluso si ese método no es parte del test de funcionalidad. Dado que `panic!` es como un test marca como fallo, llamar `unwrap` o `expect` es exactamente lo que debería pasar.

## Casos en los que se posee más información que el compilador
También seria apropiado llamar a `unwrap` o `expect` cuando hay otra lógica que asegure que el `Result` tenga un valor de `Ok`, pero la lógica no es algo que el compilador comprenda. Seguirías teniendo un valor `Result` que necesitas manejar: cualquier operación que llames todavía tiene la posibildad de fallar en general, aunque es lógicamente imposible en la situacion particular. Si se puede asegurar que, mediante una inspección manual, es imposible recibir una variante `Err`, es perfectamente válido llamar `unwrap`, e incluso mejor documentar la razon por la que nunca se va a recibir una variante en el texto del `expect`, por ejemplo:
```rust
    use std::net::IpAddr;

    let home: IpAddr = "127.0.0.1"
        .parse()
        .expect("Hardcoded IP address should be valid");
```
Creamos una instancia de `IpAddr` mendainte parsear una cadena en el código. Podemos ver que `127.0.0.1` es una dirección IP válida, por lo que es aceptable usar `expect` aquí. Sin embargo, teniendo una cadena válida codificada no cambia el tipo de retorno del método `parse`: seguimos teniendo un valor `Result`, y el compilador nos hará manejar el resultado como si la variante `Err` fuese una posibilidad, ya que el compilador no es lo suficientemente inteligente como para ver que la cadena es siempre una dirección IP valida. En caso de que la dirección IP fuese introducida por el usuario, si que sería necesario controlar el resultado de `Result`. Mencionar la asumición de que esta dirección IP esta codificada nos sugiere cambiar el `expect` para mejorar el manejo de errores en un futuro, por si se recibe la dirección IP de otra fuente.

# Pautas para manejo de errores
Es recomendable que el codigo entre en panic cuando es posible que el codigo acabe en un masl estado. En este contexto, un mal estado es cuando alguna asumición, garantia, contrato o invarainte ha sido rota, tales como valores inválidos, valores contradictorios, o valores que faltan son pasados al codigo, además de alguno de los siguientes puntos:
- El mal estado es algo inesperado, como un usasrio introduciendo datos en el formato incorrecto
- El código que continua necesita que no este en este mal estado, en vez de comprobar el problema en cada paso.
- No existe una buena forma de codificar esta información en los tipos que se usan (mejor explicacion en capitulo 17).

Si alguien trata de llamar al código y pasa un valor que no tiene sentido, es mejor devolver un error si se peude, para informar al usuario de la libreria, y que este decida que hacer en dicho caso. 

Sin embargo, en casos donde continuar peude ser inseguro o peligroso, la mejor opción es llamar a `panic!` y avisar de que la persona usando la libreria el bug en su código para que puedan repararlo durante el desarrollo. Similarmente, `panic!` es apropiado si se llama a codigo externo fuera del control y retorna un estado invalido que se es incapaz de solucionar.

Cuando un fallo es esperado, es mejor retornar un `Result`, porque indica que el fallo es una posibilidad esperada la cual el codigo que llama a dicha función decida como manejarlo.

Si el código realiza una operación que puede poner en riesgo al usuario si se llama usando valores inválidos, el código debería verificar que los valores son validos de primera y paniquear si los valores son inválidos. Esto se debe por motivos de seguridad: intentar operear sobre valores inválidos puede exponer el programa a vulnerabilidades. Esta es la razón por la que, por ejemplo, la libreria estandar llama a `panic!` si se trata de acceder a un indice fuera de memoria en un array.

Las funciones cuentan con contratos (_contracts_): su comporatmiento solo se garantiza si la entrada cumple ciertos requerimientos. Paniquear cuando se rompe este contrato tiene sentido porque una violación de contrato saiempre indica un bug del lado de la llamada y no es un tipo de error que quieres que el codigo de llamada deba manejar explicitamente. Los contratos para una función, especialmente cuando una violación causa un panic, deben ser explicados en la documentación de la API para dicha función.

Sin embargo, tener multiples comprobaciones de erro en todas las funciones puede ser muy verboso y engorroso. Por suerte, se puede utilizar el sistema de Rust de tipos (y por ende el check de tipos realizado por el compilador) para realizar múltiples comprobaciones. Por ejemplo, si tienes un tipo en vez de una `Option`, el programa espera recibir algo en vez de nada. Así, el código se salva de controlar ambos casos para las variantes `Some` y `None`: solo tendrá que controlar el caso en el que haya un valor. Código que implemente dicha función solo puede pasar valores en vez de `Option`, asegurandose en tiempo de compilación que va recibir un valor.

# Crear tipos customizables para validaciones
Vamos a recoger la idea de usar el sistema de tipos de Rust para asegurar que hay un valor valido y mirar como crear un tipo customizable para validar. Retomando el ejemplo en el capitulo 2 de adivinar el número, nunca se valido que el numero estuviese entre los valores 1 y 100, solo validamos que el valor fuese positivo.

Una mejora al código sería guiar al usuario hacia un valor valido y tener distintos comportamientos cuando un usuario adivine un numero fuera del rango a cuando el usuario introduce, por ejemplo, letras.

Una forma de hacerlo sería analizar (_parse_) la entrada como un `i32` en vez de solo un `u32` para permitir numeros negativos potenciales, y luego comprobar que está dentro del rango, tal que así:
```rust
    loop {
        // --snip--

        let guess: i32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        if guess < 1 || guess > 100 {
            println!("The secret number will be between 1 and 100.");
            continue;
        }

        match guess.cmp(&secret_number) {
            // --snip--
    }
```
La expresión `if` comprueba si el valor introducido está fuera del rango válido, avisa al usuario sobre el problema, y llama a `continue` para empezar una nueva iteracion del loop para que el usuario introduzca otro valor para adivinar. Después de la expresión `if`, se puede proceder com comparaciones entre `guess` y el numero secreto sabiendo que `guess` está entre 1 y 100.

SIn embargo, esta solución no es ideal: si fuerea absolutamente crítico que el programa solo operase con valores entre 1 y 100, y tuviese muchas funciones con dicho requerimiento, tener una comprobación como esta en cada fucnión sería tedioso, y puede afectar al rendimiento.

En su lugar, vamos a definir un nuevo tipo e introducir las validaciones en una funcion para crear una instancia del tipo en vesz de repetir las validaciones en todos lados. De esta forma, es mas seguro par las funciones de usar el nuevo tipo y sus firmas y usar los valores que recibe de forma segura:
```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```
En este caso, se define una estructura `Guess` con un valor `value` que continene un `i32`, donde se va a almacenar el numero.

Después implementamos una fucnion asociada a `Guess` llamada `new` que crea una instancia de `Guess` con un valor. La función está definida para recibir un parametro llamado `value` de tipo `i32` y devolver un `Guess`. El código dentro de `new` comprueba que el valor esté comprendido entre 1 y 100. Si el valor no pasa esta comprobación, se realiza una llamada a `panic!` la cual alertará al programador que esta escribiendo el código de llamada que tiene un bug que necesita solucionar, porque crear un `Guess` con un valor fuera de este rango es una violación del contrato que `Guess::new` está utilizando. Si pasa el test, se crea una nueva instancia de `Guess` con `value` y se devuelve.

Lo siguiente es una implementación del método llamado `value` que toma prestado `self`, no tiene parámetros, y devuelve un `i32`. Este tipo de método se le conoce como _getter_, ya que su propósito es recoger (_get_) alguna infomación sobre sus campos y retornarlos. Esta función es necesaria, ya que el valor `value` es privado. Es importante que sea privado: de esta forma, no se puede crear una instancia de `Guess` sin utilizar la función `new` para que asegure que el valor es válido.