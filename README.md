# PilaLemp tres capas

## Configuración de Red en el Despliegue con Vagrant: Despliegue de Wordpress en Tres Capas con Nginx

En esta práctica, llevaremos a cabo un despliegue en tres capas utilizando Nginx. La primera capa albergará el balanceador de carga de Nginx, la segunda capa contendrá dos servidores web con el motor PHP en una máquina independiente, y la última capa albergará el servidor de base de datos. Durante este despliegue, configuraremos y desplegaremos Wordpress. Ya cuentan con respaldo con script con todos los paquetes necesarios descargados para este despliegue.

## 1. COnfiguración del balanceador de servidores nginx


Dentro de la ruta /etc/nginx/conf.d, se crea un archivo de configuración específico para tel balanceador en Nginx. Este archivo contendrá las directivas y configuraciones necesarias para definir cómo Nginx gestionará la carga entre los servidores. 

Dentro de este archivo de configuración, dse definirá la sección upstream, donde se enumeran los servidores backend a los que se dirigirá el balanceador. Luego, en la sección server, cse configura cómo Nginx interactuará con las solicitudes entrantes y las dirigirá a los servidores backend.

 ![image](https://github.com/RKillerN/PilaLemp/assets/146434664/56060ece-7eab-4e4d-87ca-7867712ea448)

Eliminamos el enlace simbólico del archivo defaults.conf de la ruta /etc/nginx/sites-enabled.
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/611a7c68-a34d-427e-97df-86ed03476fbc)

Después de realizar cambios en la configuración de Nginx, es esencial comprobar la sintaxis para evitar posibles errores antes de reiniciar el servicio mediante el comando "sudo nginx -t". Si la sintaxis es correcta reiniciaremos el servicio.
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/a44da5bd-13bb-450a-a28a-86aadaec28a4)

Para verificar el funcionamiento de nuestro balanceador, accedemos a los servidores web individualmente. En cada servidor añadiendo un index.html en la ruta por defecto especificada en el archivo de configuración por defecto, "defaults" y abrimos un navegador web para visualizar la aplicación o el contenido del sitio.

Este proceso nos permitirá confirmar que cada servidor web está respondiendo correctamente y sirviendo la aplicación o el contenido esperado.

Visualización de servidorweb1 y servidorweb2:
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/119f8bdd-175d-4b04-9fc3-a25bf5cda68d)


## 2. Configuración del NFS y el servidor php
### Motor PHP

La configuración cgi.fix_pathinfo en PHP es un parámetro que afecta la forma en que se manejan las solicitudes de scripts CGI. Cuando se sugiere "para mayor seguridad en la línea cgi.fix_pathinfo=1, la descomentamos y le pondremos el valor de 0".
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/5f1f37d4-252a-4b65-a7de-b6fd1153979b)

El listener en PHP-FPM especifica la dirección IP y el puerto en el cual PHP-FPM escucha las solicitudes entrantes. Esta configuración es esencial para que el servidor web y PHP-FPM puedan comunicarse correctamente. Para modificar el listen hay que editar el archivo www.conf en la ruta /etc/php/7.3/fpm/pool.d/ y le añadiremos la direcciín IP de nuestro servidor PHP.

El siguiente paso consiste en editar el archivo "default" ubicado en la ruta "/etc/nginx/sites-available/". Configuramos el archivo añadiendo "index.php" para permitir que el servidor pueda procesar archivos PHP y especificamos la carpeta donde llevará a cabo su servicio. Descomentamos las líneas indicadas en las imágenes adjuntas. En la directiva "fastcgi_pass", añadimos la dirección IP de nuestro servidor PHP y el puerto asociado que es el '9000'. 
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/abd43f1d-0ce1-4122-93cd-d536e4d6b770)
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/ea532c79-4199-4b7f-9845-0db41e93a9be)

CDespués de realizar las modificaciones en el archivo de configuración, verificamos la sintaxis con el comando "sudo nginx -t" para asegurarnos de que no haya errores. En caso de que no se encuentren fallos, reiniciamos los servicios de nginx y php7.3-fpm utilizando el comando "sudo service (nombre del servicio) restart".
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/d343b6d9-d620-4db8-b4d7-18dcb0651b26)

### Configurar el NFS

El sistema de archivos en red (NFS) para esta práctica se basará en la carpeta /var/www/nginxNereaCM. El script de aprovisionamiento ya ha creado esta ruta, y ahora solo falta realizar el último paso que consiste en editar el archivo "exports" ubicado en "/etc". En este archivo, especificamos la ruta de la carpeta que queremos compartir ("/var/www/nginxNereaCM"), así como las direcciones IP de las máquinas con las que compartiremos la carpeta.

Además de esto, podemos incluir opciones adicionales en el archivo "exports" para tener un mayor control sobre el directorio compartido, como configuraciones de acceso, permisos, y demás. Estas opciones proporcionan una capa adicional de seguridad y control sobre el sistema de archivos compartido mediante NFS.

![image](https://github.com/RKillerN/PilaLemp/assets/146434664/e94ebe8d-a636-4045-9180-17136644f467)
De esta manera solo queda terminar montando las carpetas en los servidores web


## 3. Configuración de los servidores web nginx


El primer paso consiste en montar el directorio compartido en ambos servidores web. Para lograr esto, ejecutamos el siguiente comando en cada servidor web que se visualiza en la imagen, especificando la dirección IP de la máquina donde se encuentra el servidor NFS, la ruta de la carpeta que queremos compartir y el destino en nuestra máquina:
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/5f658ef9-608d-4260-85c2-1bc26e175374)

Después de la configuración inicial, procedemos a ajustar el archivo "defaults" ubicado en la ruta "/etc/nginx/nginx.conf" en ambos servidores web. En este paso, modificaremos la dirección de la directiva "root" para que apunte a nuestro directorio compartido a través de NFS. Para lograr esto, también agregaremos la referencia al archivo "index.php" para permitir el procesamiento de archivos PHP y descomentaremos las líneas según se muestra en la imagen de referencia.

Además, en la directiva "fastcgi_pass", incluiremos la dirección IP de nuestra máquina PHP seguida del puerto "9000". Este paso es crucial para establecer la conexión adecuada con el servidor PHP y garantizar la ejecución correcta de los scripts.

![image](https://github.com/RKillerN/PilaLemp/assets/146434664/725d1377-9b23-4999-83f9-0f58821743f7)
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/f34801d7-cdbe-488e-8c3e-148c9711465e)

## Configuración del servidor MariaDB

El primer paso para la configuración es ajustar el parámetro bind_address en el archivo "50-server.cnf", el cual se encuentra en la ruta "/etc/mysql/mariadb.conf.d/". Este ajuste se realiza para permitir conexiones desde cualquier dirección IP externa al servidor de base de datos MariaDB.

La base de datos y el usuario necesario para la instalación de WordPress ya han sido creados previamente mediante el script de aprovisionamiento. Para permitir que los usuarios desde cualquier dirección IP accedan a las máquinas servidores web mediante el cliente MariaDB, se ha configurado la base de datos de manera que acepte conexiones remotas.

## Instalación de WordPress


El primer paso consiste en descargar el recurso mediante el comando wget. Para hacerlo desde la ruta de nuestro archivo "defaults", añadimos la carpeta "wordpress" (/var/www/nnginxNereaCM/wordpress) a la ruta. De esta manera, al realizar modificaciones desde uno de nuestros servidores, se actualizarán automáticamente en los demás. Después de la descarga, ubicamos el archivo en la carpeta compartida y procedemos a descomprimirlo.

![image](https://github.com/RKillerN/PilaLemp/assets/146434664/44ae8fcd-840f-44d1-b841-4fd3b8a94b11)

A continuación, procedemos a realizar una copia del archivo wp-config-sample.php y lo renombramos como wp-config.php. Posteriormente, editamos dicho archivo, introduciendo la siguiente información, que incluye el nombre de la base de datos creada, el usuario asociado a esa base de datos, la contraseña del usuario, y la dirección IP de nuestro servidor MariaDB, que es la ubicación de la base de datos.

![image](https://github.com/RKillerN/PilaLemp/assets/146434664/8f400d25-1509-49ed-9165-8b43160cdbb4)


Finalmente, desde un navegador web, procedemos a completar la instalación de WordPress. Si la configuración es correcta, el sistema nos mostrará la pantalla siguiente:
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/2e244ab4-b8f5-4c50-95da-e9cd45110424)
Este paso confirma que la configuración de WordPress se ha realizado de manera exitosa y que el sistema está listo para ser utilizado. A partir de este punto, podemos seguir con la configuración específica de WordPress, como la creación de cuentas de usuario y la personalización del sitio según nuestras necesidades.

En esta pantalla, procedemos a configurar los detalles iniciales de WordPress. Nombramos el título de la página, establecemos un nombre de usuario para el administrador, ingresamos una contraseña segura y proporcionamos una dirección de correo electrónico. Finalmente, para completar el proceso, hacemos clic en la pestaña "Install WordPress".

![image](https://github.com/RKillerN/PilaLemp/assets/146434664/2f3be40b-882e-4571-a578-17dc17d375ad)

### Personalicación de MiWordPress

En la barra lateral de la página de inicio de WordPress, ubicamos la sección llamada "Pages" y hacemos clic en ella. A continuación, seleccionamos la opción "Add New Page".

![image](https://github.com/RKillerN/PilaLemp/assets/146434664/d3e12a26-ef61-41ba-a071-b303272d2d5e)
Esta serie de pasos nos dirigirá a la interfaz de creación de nuevas páginas en WordPress. Al seleccionar "Add New Page", estamos listos para comenzar a crear y diseñar una nueva página para nuestro sitio web.
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/11f5c431-8a2f-447f-a26f-9d12f5f5b82d)

Una vez finalizado tendremos Wordpress personalizado. Así se ve una vez después de personalizarlo.
![image](https://github.com/RKillerN/PilaLemp/assets/146434664/8ae1aeb2-5c05-4a0e-b65d-a18bb79e12d5)

# Conclusión

En conclusión en este proyecto, implementamos una arquitectura de tres capas utilizando la pila LEMP con Nginx, PHP y MariaDB. Configuramos Nginx como balanceador de carga y ajustamos la configuración de los servidores web y MariaDB. Desplegamos WordPress, compartiendo archivos a través de NFS para coherencia entre servidores. La personalización de WordPress destacó la flexibilidad de esta plataforma. En resumen, logramos una implementación escalable y robusta de una aplicación web utilizando la pila LEMP.

#### Screencash

https://drive.google.com/file/d/1iXsMHb-9BO-QVK44BdARj6-Dn9mr1Hgl/view?usp=drive_link
