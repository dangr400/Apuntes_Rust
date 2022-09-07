# En este documento...
En este documento se hablará de módulos y otras partes del sistema de módulos, principalmente caminos (_paths_) que permiten nombrar objetos; la palabra clave `use` que trae un camino al alcance (_scope_); y la palabra clave `pub` para hacer objetos públicos.
# Definición
Los módulos permiten organizar código en una caja para facilitar su lectura y fácil reusabilidad. También permiten controlar la privacidad de los objetos, porque código en un módulo es privado por defecto. Se puede escoger hacer público el módulo y los objetos dentro de este, exponiéndolos para hacer que código externo lo use y dependan de el.
# Chuleta de módulos
- **Empezar desde la caja raíz**: Cuando se compila una caja, el compilador primero mira en el fichero de código de la caja raíz (normalmente src/lib.rs para cajas de librería o src/main.rs para binarias), buscando código que compilar.
- **Declarar módulos**: En el documento de la caja raíz, se puede declarar nuevos módulos; por ejemplo, se declara un módulo "jardín" con `mod garden;`. El compilador buscará el código del modulo en estos lugares:
	- En la misma linea, seguido de llaves que reemplazan el punto y coma seguido de `mod garden`
	- En el fichero _src/garden.rs_
	- En el fichero _src/garden/mod.rs_
- **Declarar submódulos**: En cualquier fichero distinto al de la caja raíz, se pueden declarar submódulos. Por ejemplo, se puede declarar `mod vegetables;` en _src/garden.rs_. El compilador buscará el código del submódulo en el directorio nombrado por el módulo padre en estos lugares:
	- En la misma linea, seguido de llaves que reemplazan el punto y coma seguido de `mod vegetables`
	- En el fichero _src/vegetables.rs_
	- En el fichero _src/vegetables/mod.rs_
- **Caminos del código en módulos**: Una vez un módulo forma parte de la caja, se puede referir a ese código del módulo desde cualquier lugar en la misma caja, siempre y cuando las reglas de privacidad lo permitan, utilizando el path al código. Por ejemplo, un tipo de dato `Esparrago`  en el módulo de vegetales del jardín se encontraría en `crate::garden::vegetables::Esparrago`
- **Privado vs Público**: El codigo en un módulo es privado desde los módulos de su padre por defecto. Para hacer un módulo público, es necesario declararlo como `pub mod` en lugar de `mod`. Para hacer objetos públicos en un módulo público, es necesario utilizar la palabra clave `pub` antes de su declaración.
- **La palabra clave `use`**: en un alcance (_scope_), la palabra clave `use` crea acceso directo a los items para reducir repetición de largos caminos (_paths_). Para cualquier alcance que pueda referirse a `crate::garden::vegetables::Esparrago`, se puede crear un acceso directo con `use crate::garden::vegetables::Esparrago` para que después solo sea necesario escribir `Esparrago` para utilizar ese tipo en el alcance actual.

# Ejemplo: restaurante
Vamos escribir una caja librería que provea la funcionalidad de un restaurante. Definiremos las funciones pero dejaremos el interior vacío para concentrarse en la organización del código en vez de su funcionamiento.

En la industria de los restaurantes, algunas partes se consideran _front of the house_ y otras _back of the house_.
- El frente se refiere a donde están los clientes; encompasa donde los hosts sientan a los clientes en la mesa, los servidores reciben los pedidos y pagos, y los baristas hacen bebidas.
- El back se refiere donde los chefs cocinan en cocina, los lavaplatos lavan, y los mánagers realizan el trabajo administrativo.
Para estructurar la caja de esta forma, podemos organizar las funciones en módulos anidados. Creamos una nueva librería llamada restasurante con `cargo new --lib restaurant`; despues introducimo el código en _src/lib.rs_ para definir algunos módulos y funciones clave. Aquí esta la sección de front:
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```
Definimos el módulo con la palabra clave `mod` seguido del nombre del módulo, en este caso `front_of_the_house`. El cuerpo del módulo va entre llaves (`{}`). Dentro de lo s módulos, podemos poner otros módulos, en este caso `hosting` y `serving`. Los módulos tambien pueden contener definiciones de otros objetos, tales como Structs, enums, constantes, rasgos y funciones.

Utilizando módulos podemos agrupar definiciones relacionadas junstas y nombrar por que están relacionadas. Los programadores que utilicen este código pueden navegar el código basado en los grupos en lugar de tener que leer todas las definiciones, haciéndolo más sencillo encontrar las definiciones relevantes para ellos. Los programadores que fuesen a añadir funcionalidad al código sabrían donde localizar el código para mantener el programa organizado.

Antes se mencionó que _src/main.rs_ y _src/lib.rs_ se llaman raices de cajas. El motivo de su nombre se debe a que los contenidos de cualquiera de estos dos ficheros forman un módulo llamado `crate` en la raíz de la estructura de módulos de la caja, conocido como _**árbol de módulos**_.

A continuación se muestra el árbol de módulos para el ejemplo anterior:
```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

Este árbol de módulos es similar a un arbol de directorios en un sistema de ficheros. En este arbol, `hosting` y `serving` serian hermanos, y `front_of_the_hosue` sería el padre de ambos.

Para poder referirse a uno de estos es necesario un sistema de [[Paths]].
