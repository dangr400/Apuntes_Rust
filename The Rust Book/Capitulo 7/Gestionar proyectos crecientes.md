# Motivo
A medida que se escriben programas más grandes, organizar el código se vuelve más importante. Al agrupar funcionalidad relacionada y separar código con distintas capacidades, se clarifica donde se puede encontrar código que implementa una capacidad en particular y donde ir para cambiar como funciona una característica.

# Paquetes
Un paquete permite contener múltiples cajas (_crates_) y, opcionalmente, una caja de librerías (_library crate_). A medida que crece el paquet, se puede extraer partes en distintas cajas que se conviertan en dependencias externas. Este capítulo cubre estas técnicas.

Para proyectos muy grandes compuestos de un se de paquetes interrelacionados que evolucionan juntos, Cargo ofrece espacios de trabajo ([[Areas de trabajo en Cargo]]).


Rust cuenta con una serie de características que permiten gestionar la organización del código, incluyendo los detalles que se exponen, que detalles son privados, y que nombres están en cada alcance (_scope_) en los programas. Estas características, llamadas _sistema de módulos_ incluye:

- **[[paquetes]]** : Característica de Cargo que permite crear, testear y compartir cajas (_crates_)
- **[[cajas]]** (**_crates_**) : Arbol de módulos que producen una librería o ejecutable
- **Módulos** y **uso** (_**use**_) : Permite controlar la organización, alcance y privacidad de los caminos (_paths_) 
- **Caminos** (**_paths_**) : Una forma de nombrar un objeto, tales como structs, funciones o módulos
