
# RFC: BME-changes The use of Angular 18 Frontend

La intencion de esta RFC es la proponer el uso de angular 18 en BME platform como forma de escalamiento horizontal y separacion funcional de codigo fuente del Frontend y Backend

**Autor**: Jorge Lopez

**Posted**: 26/07/2024

**Status**: Open 

## Contexto

Actualmente BME platform es un aplicacion realizada en los stack de PHP, JavaScript, CSS y HTML en la parte web; y angular 11 y 12 en la parte mobile. La aplicacion es monolitica por cuanto contiene en su codigo fuente las dos instancias, ademas de la interfaz de comunicacion con la base de datos y la parte que se renderiza en el cliente. 

Con este RFC busco proponer impactar la separacion del alcance del codigo de los desarrollos futuros que se realicen sobre BME platform, de tal forma que se pueda realizar un Frontend mas estandar, de forma mas rapida y segura; ademas de que el codigo back diverga sin la necesidad de impactar sobre lo que se renderiza en la parte del cliente.


## Metas

Este cambio impactaria la forma como realizamos las expansiones de los futuros modulos de BME platform, dado que en este momento el front se realiza encapsulado en PHP templates, con algunos llamados recursivos a funciones internas y a consultas a tablas. 

La idea es utilizar un monorepositorio con [Nx](https://nx.dev/concepts/decisions/why-monorepos), que contenga dentro de si el frontend que se realizaria en Angular 18 de los futuros modulos de la aplicación. 

Esta idea se encuentra fundamentada en que, separando el codigo front de esta forma podriamos aprovechar para utilizar las ultimas versiones de angular y de typescript, haciendo el codigo mas consistente y mantenible, ya que esta version estaria dentro de LTS (long term support) de [versiones y releases oficiales del framework](https://angular.dev/reference/releases) del equipo oficial de angular; lo anterior es importante ya que nos cubririamos en terminos de parches y bugfix en temas de seguridad y optimizacion; y de poder usar de las librerias actuales y de desechar third party libraries deprecated y obsoletas que agregan tamaño innecesario y ruido a nuestro codigo. Por otra parte, podriamos beneficiarnos de la ventaja de crear librerias de consumo interno para reutilizar componentes, servicios y directivas Front que sean de común aprovechamiento entre los modulos, esto evita la repeticion de codigo en el front y acceleraria los desarrollos.

La separacion del Frontend y del Backend evita la sobreexposicion de nuestro codigo de consulta a base datos al cliente, reduciendo las probabilidades en muchas ocasiones la inyeccion de codigo PHP y de SQL; dejando separado y aislado el codigo core del negocio, y el codigo del lado del cliente, permitiendo usar estrategias para la minimización de riesgos XSS y de seguridad de contenidos mediante la sanitanización de las entradas y consultas al backend. Adicionalmente, esto nos permitira la especialización del equipo, entre aquellos que manejen lógica back, o lógica front, permitiendonos aumentar la calidad del código y la eficiencia en las tareas del equipo.

Por último, este nueva implementación abarcaria la posibilidad de realizar un unico desarrollo responsive para web y dispositvos mobile, ya que la capa de Ionic no es necesaria al no tener necesidad el proyecto de usar features nativas. 


## Features

- Monorepositorio NX
- Angular 18
- SCSS
- RxJs
- Azure devOps


## Propuesta

Existe un repositorio de prueba en azure Repos del proyecto que sintetiza la propuesta: 

![image](https://github.com/user-attachments/assets/0e2923cf-3fec-47b0-8632-a7be53c826f6)

Se configura un Monorepo con una apps donde se crean los modulos (fronts), y una carpeta libs donde se ubican las distintas librerias de creacion propia que se quieran utilizar. Agregado a esto se encuentran los archivos de configuracion del monorepositorio, de lint, de preset para pruebas unitarias en Jest, de config de typescript, y el archivo de historial de paquetes y de personalizacion de scripts package.json

Aclarar que Nx, es una capa contenedora, y las aplicaciones contenidas en la carpeta apps, son aplicaciones front separadas e independientes, que pueden ser desplegadas, desarrolladas y separadas cada una. Para manejar una consistencia en el desarrollo se utiliza una sola configuracion de paquetes para todas las apps, de tal manera que no haya apps desarrolladas por fuera del framework comun, y con consumo de paquetes y librerias no autorizadas ni relevantes para el proyecto.

Para desarrollar primeramente se debe [instalar de forma global Nx](https://nx.dev/getting-started/installation) compatible con la version apropiada con node JS, y tener igualmente de forma global instalado angular en el sistema, se puede utilizar y se recomienda npm, pero se puede utilizar yarn y pnpm. 

La creacion de una aplicacion (de ahora en adelante moduloBME) comienza con los comandos ejecutores de Nx CLI (command line interface), `nx g @nx/angular:app nombre-del-app`, a partir de allí se genera una estructura tradicional de una app comun de angular 18: 

![image](https://github.com/user-attachments/assets/4b4f7e5c-f8b8-4ce6-bcc8-55aab5ae9238)

Esta estructura tiene su carpeta src, donde se crean componentes, vistas, interfaces y servicios; y archivos de personalizacion de eslint, Jest y de typescript que son versiones extendidas de las configuradas en el repositorio; esto, nuevamente para que haya una unificación en el codigo fuente y pruebas unitarias en los modulosBME. 

Nx tambien tiene documentacion oficial donde se puede consultar [otros comandos ejecutores](https://nx.dev/nx-api/nx), igualmente existe una extension para visual studio code de Nx para utilizar schematics llamado [Nx console](https://marketplace.visualstudio.com/items?itemName=nrwl.angular-console) que permite de forma visual crear recursos ubicando simplemente el espacio de trabajo que es la carpeta contenedora del proyecto en si:

![image](https://github.com/user-attachments/assets/1ec1ead4-28a8-4f21-ab9d-6954afccf5c2)

Tambien se pueden crear librerias que contengan logica de servicios y de reutilizacion de componentes y directivas mediante el comando `nx g @nx/angular:library nombre-de-la-libreria`, este comando crea una carpeta src y archivos simplificados de configuracion desde donde se puede comenzar a escalar la libreria a necesidad. 

![image](https://github.com/user-attachments/assets/249c468e-e586-4117-8ad0-4d0b7a223d98)

Una de las cosas mas importantes del comando de creacion de la libreria es la asignacion de un path que simplifica la importacion y exportacion de librerias de consumo interno. Recordemos que aunque estamos en un monorepositorio, el moduloBME no va a conocer nada por fuera de sus carpetas internas, 

![image](https://github.com/user-attachments/assets/0b436d69-d182-4b6e-a173-aa36d94e87b3)

De esta forma podemos consumir la libreria en cualquier moduloBME que lo requiera, como en el siguiente ejemplo que se consumio la libreria ui-lib en el componente table del moduloBME downtime_configuration para utilizar una etiqueta de un componente de angular material,

![image](https://github.com/user-attachments/assets/5993633d-4cac-4f93-a43d-5b444dae9c88)

Y en otro moduloBME (test-app) que se consumio la misma libreria en un componente de ejemplo,

![image](https://github.com/user-attachments/assets/c467af2f-f6c6-41da-866f-a6d4742f4884)


Así pues, cuando estemos ampliando y expandiendo podemos ver que podemos reutilizar componentes a traves de nuestros distintos modulosBME, y nuestro Monorepo empezara a notarse un poco mas de esta manera: 

![image](https://github.com/user-attachments/assets/f8fa9de3-010a-4820-9a7c-f8fdadffa93b)


Para servir en local un moduloBME se ejecuta el comando `nx serve nombre-del-moduloBME` esto compila el codigo del moduloBME y lo sirve en local para debugeo y desarrollo. Para generar la compilacion de distribucion, se accionara el comando `nx build nombre-del-moduloBME` que creara una carpeta _dist_ con los chunks y archivos que iran en el servidor monolitico de BME. Se debera crear una ruta de acceso en el menu lateral de BME para acceder este modulo

![image](https://github.com/user-attachments/assets/0211a85c-c041-482c-b7bc-73815e98794b)

Y por ultimo asegurarse que el archivo index compilado tenga la etiqueta base cambiada de tal forma que se relativicen todas las llamadas de los recursos accionados por el index.html: en este caso en la carpeta _/buildapp/src/test_front_app/_ que es la carpeta creada para contener el resultado compilatorio del codigo del moduloBME que esta en el monorepositorio. De esta forma, en el repo de BME solo estara la compilacion del codigo typescript, scss y hmtl, ademas de las librerias que se requieren, de forma optimizada, comprimida, minimizadas, compatible con los navegadores de la version de angular utilizada, y eficiente; de dificil lectura para humanos, pero ordenada y organizada para nosotros como mantenedores del proyecto 

![image](https://github.com/user-attachments/assets/27ac44a6-16bf-4946-a382-8a0396de619d)
