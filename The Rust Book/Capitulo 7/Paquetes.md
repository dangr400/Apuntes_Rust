# Definición
Un paquete (_package_) es un conjunto de una o más cajas que provée una serie de funcionalidades. Un paquete ci¡ontiene un fichero _Cargo.toml_ que describe como buildear esas cajas. Cargo es en realidad un paquete que contiene la caja binaria para las herramientas de la linea de comandos utilizada hasta ahora para buildear el código.

El paquete de Cargo tambien contiene una caja librería de la cual depende la caja binaria. Otros proyectos pueden depender de la librería de Cargo para usar la misma lógica que utiliza las herramientas de linea de comandos de Cargo.

# Contenido
Un paquete puede contener tantas cajas binarias como se desee, pero como máximo solo una libreria. Un paquete debe contener al menos una caja, bien sea librería o binaria.
```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```
Al crear un nuevo proyecto, se crea un fichero Cargo.toml, por lo que tenemos un paquete. En el directorio del proyecto también hay una carpeta `src` donde, por convenio, es la caja raiz de la caja binaria con el mismo nombre que el paquete. Así mismo, Cargo sabe que si el paquete contiene un fichero llamado lib.rs en la carpeta `src`, eel paquete contiene una libreria con el mismo nombre que el paquete. Cargo pasa los ficheros de la caja raiz a `rustc` para bildear la libreria o el binario, respectivamente.