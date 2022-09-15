# Definición
El tipo de dato `HashMap<K, V>` almacena un mapeo de claves de tipo `K` con valores de tipo `V`, utilizando una funcion de fragmentación (_hashing function_), la cual determina como almacenar estas clave y valores en memoria.

Son útiles para cuando se quiere buscar información sin un índice, sino con una clave que puede ser de cualquier tipo.

Por ejemplo, en un juego se peude mantener la puntuación de cada equipo en un hash map en el cual cada clave es el nombre del equipo y los valores son los valores de puntuación de cada equipo

# Crear un nuevo Hash Map
Se crea un hash map con `new` y se agregan nuevos valores con `insert`:
```rust
	use std::collections::HashMap;    // necesario incluir paquete

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
```
Los hash maps almacenan su información en el [[Heap]]. Este hash map `scores` tiene claves de tipo `String` y valores de tipo `i32` (por defecto). Como los vectores, los has maps son homogéneos: todas las claves deben tener el mismo tipo que las demás, y todos los valores deben ser del mismo tipo entre ellos.

# Acceder a valores en el mapa
Se puede acceder al valor de un mapa hash proveyendo la clave con el método `get`:
```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name);
```
El método `get` devuelve un `Option<&V>`; si no hay ningún valor con esa clave en el mapa, `get` devuelve `None`.

También se puede iterar sobre cada par clave/valor de forma similar a un vector con el `for` loop:
```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
```
El acceso a los valores del mapa es arbitrario
# Hash maps y pertenencia
Para los tipos que implementan el rasgo `Copy`, como los `i32`, los valores se copian en el hash map. Para los valores con pertenencia como `String`, los valores se mueven y el hash map será el dueño de dichos valores:

```rust
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
```

Si se introducen referencias a los valores en el hash map, los valores no serán movidos al hash map. Los valores que las referencias apuntan deben ser válidos por lo menos hasta que el hashmap es válido.

# Actualizar un Hash map
Cada clave solo puede tener un valor asociado a esta clave de cada vez. Si se pretende cambiar los datos en el hash map, debes decidir como manejar el caso de que ya hay una clave con dicho valor asignado. Se puede reemplazar el valor antiguo con el nuevo valor, descartando el viejo. O se podría mantener el valor antiguo y desechar el nuevo, agregando solo el nuevo valor si no existe la clave.

## Sobreescribir un valor
Si insertamos una clave y valor en el mapa, y después se inserta la misma clave con un valor distinto, el valor asociado con esa clave es reemplazado.
```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{:?}", scores); // muestra {"Blue": 25}
```

## Añadir clave-valor solo si no existe la clave
Es común comprobar si una clave en particular existe en el hash map con el valor: si la clave existe en el hash map, el valor existente debe permanecer en el mapa; si no existe, insertarla con el nuevo valor.

Los Hash Maps cuentan con una API especial llamada `entry` que toma el valor de la clave que se quiere comprobar como parámetro. El valor de retorno de `entry` es un enum llamado `Entry` que representa un valor que puede o no existir:
```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{:?}", scores);
```

El método `or_insert` en `Entry` retorna una referencia mutable al valor para la correspondiente clave `Entry` si esa clave existe, y si no, inserta el parámetro como un nuevo par clave-valor, y devuelve una referencia mutable de dicho valor.

## Actualizar un valor basado en uno antiguo
Otro caso común en los hash maps es mirar el valor de una clave y después actualizarlo basado en el valor antiguo:
```rust
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map);
```

# Funciones hashing
Por defecto, `HashMap` utiliza una funcion de hash llamada _SipHash_ la cual provée resistencia a ataques de DoS en tablas hash. No es el algoritmo más rapido, pero la seguridad ofrecida merece la pena el coste en rendimiento.

En caso de que esta función de hash sea demasiado lenta para los propósitos del programa, se puede cambiar a otra función especificando el hasher. El Hasher es un tipo que implementa el rasgo `BuildHasher`. Se hablará mas detalladamente de los rasgos en el Capítulo 10.