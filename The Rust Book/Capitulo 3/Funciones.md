# Convenio de declaración
- Se utiliza _snake case_ como estilo para el nombre de las funciones (y de variables). Todas las letras son minúsculas y se separan las palabras con barra baja.
	- Ejemplo de snake case: 
```rust
fn another_function() {
    println!("Another function.");
}
```

# Cómo se definen
Para definir una funcion se escribe la palabra reservada `fn` seguido del nombre de la función y un par de paréntesis. Las llaves indican al compilador donde empieza el cuerpo de la función y donde termina.

# Llamadas
Una funcion puede llamarse en cualquier punto, siempre y cuando esté definida en el ámbito de donde se llame (es decir, que esté o definida o referenciada en el _scope_).

# Parámetros
Las funciones pueden recibir parámetros, que son variables especiales que forman parte de la firma de una función.
Cuando una función tiene parámetros, se puede ofrecer valores concretos a dichos parámetros. Tecnicamente, estos valores se llaman **argumentos**, pero normalmente se intercambia las palabras parámetro y argumento para, o bien variables en la definición de una funcion, o para los valores concretos pasados a la función.

# Declaración de tipos
Es **obligatorio declarar el tipo de cada parámetro** en la definición de la función.
# Cuerpo de una funcion
## Estamentos
Son instrucciones que realizan una acción y no devuelven un valor

## Expresiones
Las expresiones son instrucciones que realizan una acción y devuelven un valor resultante. Llamar a una **función**, o a una **macro** es una expresión, así como también lo es un nuevo bloque creado con llaves ("**{ }**"). Un ejemplo de este último caso:
```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {y}");
}
```
La expresión:
```rust
{
    let x = 3;
    x + 1
}
```
Es un bloque que evalua a 4 (x= 3; x+1=4). Este valor se enlaza a **y** como parte del estamento **let**. Notese que la linea x + 1 no cuenta con punto y coma al final.

## Punto y coma final
Si se agrega un punto y coma al final de una expresión, se transforma en un **estamento**, es decir, no devuelve ningún valor. En el caso anterior, si se agregase punto y coma al final de la linea **x + 1** , **"y"** no tendría ningún valor asignado.

# Funciones con valor de retorno
Una función puede devolver valores al código que las llama. No se nombrar los valores de retorno, pero hay que declarar su tipo seguido de una **flecha** ("->", guión y mayor-que). En Rust, el valor de retorno de una función es sinónimo con el **valor de la expresión final en el bloque del cuerpo de una funcion**.

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {x}");
}
```

También se puede devolver el valor antes de la última expresión con la palabra clave **return** y especificando el valor, pero la mayoría de funciones devuelven la última expresión implicitamente.

```rust
fn five() -> i32 {
    return 5;
}

fn main() {
    let x = five();

    println!("The value of x is: {x}");
}
```
