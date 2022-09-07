# Definición
Los métodos son similares a las funciones:
- se declaran con la palabra clave `fn` y un nombre
- Pueden tener parámetros y un valor de retorno
- Contienen código que se ejecuta cuando se ejecuta al ser llamado desde otra parte.

A diferencia de una función, los métodos:
- se definen en el contexto de un Struct o de un [[enum]] o un [[trait object]]
- Su primer parámetro es siempre `self`, que representa la instancia del struct al que se llama en el método

# Demostración
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

Se comienza con un bloque `impl` para el Struct, en este caso "Rectangle" (todo lo escrito en el bloque de impl se asociará con el tipo que implementa).

## Elección de parámetros en funcion area
En este caso, solo se escoge el valor `&self` porque solo es necesario leer el valor de los datos contenidos en el struct; si fuese necesario reasignar los valores, el parámetro sería `&mut self` para que pueda tomar [[pertenencia]] del valor.

## Nombres válidos
Se puede dar el mismo nombre de una de las variables del struct, como al método; Rust entiende que si va seguido de paréntesis es un método, y si la referencia es sin paréntesis, está accediendo a un valor del struct:
```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}
```

Normalmente, asignar el nombre de una función el mismo nombre que un dato es para devolver el valor del dato. Estos métodos se llaman _getters_, los cuales son útiles al hacer el dato privado pero el método público, y así activar acceso de solo lectura como parte de la API publica del tipo de dato

# Métodos con más de un parámetro
```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```
En este ejemplo, se crea una nueva función llamada `can_hold` que recibe un parámetro del tipo Referencia de Rectángulo. Esto es para transmitir un préstamo inmutable (ya que **solo es necesario leer los datos, no modificar el rectángulo**) de un rectángulo para comprobar si el primer rectángulo puede contener al segundo rectángulo:
```rust
fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let rect3 = Rectangle {
        width: 60,
        height: 45,
    };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

# Funciones Asociadas
Todas las funciones definidas en un bloque de `impl` se llaman funciones asociadas porque están asociadas con el tipo nombrado en el `impl`.

Se pueden definir funciones asociadas que no tienen `self` como primer parámetro porque no necsitan una instancia del tipo para poder funcionar (`String::from` es una funcion de este estilo). Este tipo de funciones sin self como parámetro **no son métodos**.

Estas funciones asociadas que no son métodos se utilizan normalmente para constructores que devuelven una nueva instasncia del struct:
```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
```
la palabra clave `Self` en el tipo de retorno y en el cuerpo de la función son alias para el tipo que aparece despues de la palabra clave `impl`, en este caso **Rectangle**.

## Llamada de funciones asociadas
Para llamar a la función asociada, se utiliza la sintaxis `::` con el nombre del struct:
``` Rust
 let sq = Rectangle::square(3) // Por ejemplo
```
Esta sintaxis se utiliza tanto para funciones asociadas y espacios de nombre ([[namespace]]) creado por [[módulos]].


# Múltiples bloques `impl`
Cada struct tiene permitido contar con múltiples bloques `impl`. Esto es útil para [[tipos genericos]] y [[traits]] (rasgos):
```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```