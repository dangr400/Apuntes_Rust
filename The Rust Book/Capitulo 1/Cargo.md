[Documentación oficial ](https://doc.rust-lang.org/cargo/)
# Qué es
Es el gestor de paquetes que está incluido con [[Rust]]

# Qué hace
- realiza la "construcción" el codigo
- descargar las dependencias del código
- construir dichas librerias

# Crear un proyecto con cargo
El comando `cargo new <nombre_proyecto>` crea un directorio con el nombre que se le introduzca. Cargo se encargará de crear los ficheros en la carpeta. La estructura base que crea es:
- Un fichero de configuración en formato **[[TOML]]** (llamado Cargo.tml)
- Una carpeta donde se aalmacena el código, llamada "**src**" 
	(en donde crea el fichero main.rs)
- Un nuevo repositorio Git, así como un fichero .gitignore
	(para verlos, es necesario usar `ls -a`)
	- Estos ficheros no se generan si se ejecuta `cargo new` en un repositorio Git que ya existe. Para sobrreescribir este comportamiento, es necesario usar `cargo new --vcs=git`
	- Tambíen es posible utilizar otro sistema de control de versiones o ninguno utilizando el flag anterior `--vcs`. Para ver las opciones disponibles, ejecutar `cargo new --help`.

# Características
- Cargo espera que todo el código fuente esté contenido en la carpeta "src". 
- El directorio raíz solo existe para almacenar los ficheros README, información de licencia del software, archivos de configuración, y todo aquello que no esta relacionado con el código.
- Si se crea un proyecto sin Cargo, se puede transformar almacenando el código en una carpeta llamada src y creando el archivo Cargo.toml pertinente.

# Compilar y ejecutar un proyecto de Cargo
Desde la carpeta raíz del proyecto, se ejecuta `cargo build` para compilar el proyecto. Una vez ejecutado, se crea un fichero ejecutable en la nueva carpeta "target" (_en target/debug/nombreFichero_)
Al ejecutar por primera vez el comando, se crea un fichero en la raíz llamado Cargo.lock, el cual mantiene un histórico de las versiones exactas de las dependencias del proyecto. Este fichero no debería ser manipulado manualmente, sino que se encarga Cargo de ello.

Además de ejectuar manualmente el programa "buildeado", se puede ejecutar `cargo run` para compilar y ejecutar el código directamente.

También existe el comando `cargo check`. Este comprueba que el código compilará, pero no produce un ejecutable, **Solo lo comprueba**.

# Compilación para lanzamiento
Cuando un proyecto esta finalmente listo para lanzamiento, se puede utilizar el comando `cargo build --release` para compilar con optimizaciones. Crea un ejecutable en _target/release_ en vez de en _target/debug_. Estas optimizaciones hacen que el código de Rust se ejecute más rápido, pero al activarlas hacen que el proceso de compilación tarde más. Esto explica por que hay dos perfiles de compilación:
1. Para desarrollo, donde prima recompilar rapidaamente y reiteradas veces
2. Para salida al mercado, en el cual el cliente no va a recompilar el programa y que hace falta que se ejecute lo más rápido posible.
Para comprobar la velocidad del programa, es mejor probar con un _release_ para tener los tiempos reales de ejecución.

# Cargo como convenio
Para  trabajar con proyectos nuevos o existentes, se recomienda utilizar Cargo. Es más, se puede utilizar la siguiente sucesion de comandos para coger el código de un proyecto y compilarlo para su uso con la siguiente sucesión de comandos:
`$ git clone ejemplo.org/nomeProxecto`
`$ cd nomeProxecto`
`$ cargo build`
