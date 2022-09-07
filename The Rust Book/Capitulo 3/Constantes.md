# Cómo son en Rust
- Las constantes se declaran con la palabra clave `const`.
- Es obligatorio especificar el tipo de valor que va a almacenar. Pueden ser declaradas a cualquier nivel, incluido a nivel global.
- Solo se pueden settear con una expresión constante, no de un resultado que solo puede ser computado en tiempo de ejecución (para más info, [evaluacion de constantes](https://doc.rust-lang.org/reference/const_eval.html))
- Son validas durante todo el tiempo de ejecución del programa en el scope que fueron declaradas.