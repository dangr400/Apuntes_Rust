# Definición
Se refiere a ejecutar ciertos segmentos de código en base a una condición, o ejecutar código repetitivamente mientras se cumpla cierta condición. En esta definición se incluyen:
## Condicionales
### If - Else
Permite enramar el código dependiendo de cierta condición. La condición debe ser de tipo `bool`, en caso contrario el código no compilará correctamente.

```rust
if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
```


- If - Else if
Se puede encadenar múltiples condiciones combinando `else if`:
```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

### Utilizando `if` en una sentencia `let`
Dado que if es una expresión, se puede utilizar a la derecha de una sentencia `let` para asignar la salida de una variable:
```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}");
}
```
Para este caso, **ambos valores deben ser del mismo tipo de dato**.
## Repeticiones
### Loop
Esta palabra clave informa a Rust para ejecutar un bloque de código infinitamente, o hasta que se cancele explicitamente con la palabra clave `break`. También se puede utilizar la palabra clave `continue` para saltarse todo el código restante del ciclo del loop y continuar de nuevo en la siguiente iteración.

#### Devolver un valor en un ciclo loop
Se puede devolver un valor en la salida de un bloque repetitivo añadiendo el valor de retorno después de la palabra clave `break`:
```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}
```

#### Etiquetas para loops
Se puede nombrar con una etiqueta un ciclo para especificar que loop cerrar, en caso de tener loops anidados:

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```

Por norma, `break` y `continue` se aplican al loop más interno. En el caso de ejemplo, el primer break afecta al loop sin etiqueta, mientras que el segundo afecta al loop más externo.

### Loops condicionales
#### While
A veces es necesario evaluar una condición para continuar con un loop. Para esto, se utiliza la palabra clave `while`, la cual continuará ejecutando el código mientras se cumpla la condición que evalúa. Este objetivo se puede lograr utilizando `loop, if, else y break`, pero dado que es un patrón muy común se incorpora esta palabra clave.

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

#### For
Para acceder a los valores de una colección, se puede acceder con un `while` y comprobando que no supere el índice de elementos. Pero por comodidad, existe la palabra clave `for`, la cual permite iterar sobre los elementos de una colección:
```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}
```
