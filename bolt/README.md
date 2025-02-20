# Guía de Explotación - TryHackMe Bolt

## Reconocimiento

Iniciamos con un escaneo Nmap para identificar los puertos abiertos, servicios, versiones de estos, y demás información relevante:

`nmap -T4 --min-rate=10000 -p- -A IP_MAQUINA_ATACADA`

Explicación de los parámetros:
1. `-T4` -> Define la velocidad del escaneo.
2. `--min-rate=10000` -> Envía al menos 10,000 paquetes por segundo.
3. `-p-` -> Escanea todos los puertos disponibles.
4. `-A` -> Activa un escaneo agresivo para obtener más información.

![alt text](<./assets/2025-02-07 16_35_21-KaliLinux-Hacking - VMware Workstation.png>)

## Explotación de la Web

El escaneo revelo que hay dos puertos abirtos dando sevicio `http`, el puerto `80` y el `8000`, por lo que vamos a entrar y mirar a ver que hay.

Entramos en el puerto 80, y vemos la pagina predeterminada de Apache.

![alt text](<./assets/2025-02-07 16_35_54-KaliLinux-Hacking - VMware Workstation.png>)

No había tanto contenido en ``robots.txt`` ni en la visualización de la fuente de la página. Así que saltamos al siguiente puerto.

En el puerto ``8000`` encontramos un sitio de Bolt.

![alt text](<./assets/2025-02-07 16_36_14-KaliLinux-Hacking - VMware Workstation.png>)

Si revisamos el sitio, encontraremos un nombre de usuario y una contraseña.

Lo primero que encontramos es la contraseña la cual es `boltadmin123`.

![alt text](<./assets/2025-02-07 16_36_59-KaliLinux-Hacking - VMware Workstation.png>)

Mas abjo tambien encotraremos el nombre del usuario, el cual es `bolt`.

![alt text](<./assets/2025-02-07 16_37_27-KaliLinux-Hacking - VMware Workstation.png>)

Tambien buscando entre los directorios hay un login al cual se accede poniendo `/bolt`.

![alt text](<./assets/2025-02-07 16_38_11-KaliLinux-Hacking - VMware Workstation.png>)

Ponemos el usuario y contraseña encontrado anterirormente (`bolt`:`boltadmin123`), y nos loguearemos correctamente en la web.

Buscando un poco en ella pdemos encontrar la version de bolt que esta instalada, la cual es la `3.7.1`.

![alt text](<./assets/2025-02-07 16_40_09-KaliLinux-Hacking - VMware Workstation.png>)

## Comprometiendo la Máquina

Con esto podemos usar Searchsploit para buscar los exploits disponibles y ver si hay alguno para esta version.

![alt text](<./assets/2025-02-07 16_40_46-KaliLinux-Hacking - VMware Workstation.png>)

Vemos que si hay un exploit para la version 3.7 por lo que usaremos Metasploit para ejecutarlo.

`msfconsole` Con este comando ejecutamos Metasploit.

`use 0` Par seleccionar el exploit que queremos ejecutar.

![alt text](<./assets/2025-02-07 16_42_45-KaliLinux-Hacking - VMware Workstation.png>)

Ahora ejecutaremos el comando `show options` para ver los parametros del exploit que debemos de rellenar.

![alt text](<./assets/2025-02-07 16_46_57-KaliLinux-Hacking - VMware Workstation.png>)

Y usando el comando `set PARAMETRO VALOR`, rellenamos todos los parametros que nos pide el exploit para funcionar.

![alt text](<./assets/2025-02-07 16_50_54-KaliLinux-Hacking - VMware Workstation.png>)

Una vez tengamos todo listo, podremos el comando `run` para ejecutar el exploit.

![alt text](<./assets/2025-02-07 16_51_41-KaliLinux-Hacking - VMware Workstation.png>)

Cuando alla cargado correctamente, ya estaremos en la maquina de la victima, y podremos ejecutar comandos como por ejemplo `whoami` el cual nos sirve para identificar en que usuario estamos, y en este caso estamos en el usuario `root`.

![alt text](<./assets/2025-02-07 16_52_46-KaliLinux-Hacking - VMware Workstation.png>)

Por lo que nos podremos en la busqueda de la flag, que en este caso se encuentra en el `/home` del usuario, por lo que con un `cat flag.txt`, podremos ver la flag perfectamente.

![alt text](<./assets/2025-02-07 16_55_30-KaliLinux-Hacking - VMware Workstation.png>)

¡Máquina comprometida con éxito!
