

vagrant init bento/ubuntu-20.04


Edita el Vagrantile para hacer el forward del puerto 8080 al puerto 80

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.network "forwarded_port", guest: 80, host: 8080
end


$ vagrant up
$ vagrant status
$ vagrant ssh



Paso 1: Instalar Apache y actualizar el firewall:

Instale Apache usando el administrador de paquetes de Ubuntu, apt:

$ sudo apt update
$ sudo apt install apache2

ajustar la configuración de su firewall para permitir tráfico HTTP y HTTPS. UFW :

$ sudo ufw app list


Output
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH


Para permitir tráfico únicamente en el puerto 80 utilice el perfil Apache:

$ sudo ufw allow in "Apache"


Puede verificar el cambio con lo siguiente:

$ sudo ufw status


Output
 Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                                
Apache                     ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)                    
Apache (v6)                ALLOW       Anywhere (v6)     

Puede realizar una verificación rápida para comprobar:


http://your_server_ip

Cómo averiguar la dirección IP pública de su servidor:

$ ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'




Paso 2: Instalar MySQL

Una vez más, utilice apt para adquirir e instalar este software:

$ sudo apt install mysql-server


Inicie la secuencia de comandos interactiva ejecutando lo siguiente:

$ sudo mysql_secure_installation

Elija Y para indicar que sí, o cualquier otra cosa para continuar sin la habilitación.

VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No:

Si responde “sí”, se le solicitará que seleccione un nivel de validación de contraseña

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary              file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1




Si está conforme con su contraseña actual, ingrese Y para indicar “sí” en la solicitud:

Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y

Cuando termine, compruebe si puede iniciar sesión en la consola de MySQL al escribir lo siguiente:

$ sudo mysql

Debería ver el siguiente resultado:

Output
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 22
Server version: 8.0.19-0ubuntu5 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.



Para salir de la consola de MySQL, escriba lo siguiente:

mysql>exit




Paso 3: Instalar PHP

$ sudo apt install php libapache2-mod-php php-mysql

confirmar su versión de PHP:

php -v

Output
PHP 7.4.3 (cli) (built: Mar 26 2020 20:24:23) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.3, Copyright (c), by Zend Technologies


Paso 4: Crear un host virtual para su sitio web


$ sudo mkdir /var/www/your_domain

$ sudo nano /etc/apache2/sites-available/your_domain.conf

<VirtualHost *:80>
    ServerName your_domain
    ServerAlias www.your_domain
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/your_domain
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

Ahora, puede usar a2ensite para habilitar el nuevo host virtual:

$ sudo a2ensite your_domain

Para deshabilitar el sitio web predeterminado de Apache, escriba lo siguiente:

$ sudo a2dissite 000-default

Para asegurarse de que su archivo de configuración no contenga errores de sintaxis, ejecute lo siguiente:

$ sudo apache2ctl configtest


Por último, vuelva a cargar Apache para que estos cambios surtan efecto:

$ sudo systemctl reload apache2

Cree un archivo index.html en esa ubicación para poder probar que el host virtual funcione según lo previsto:

$ nano /var/www/your_domain/index.html

<h1>It works!</h1>

<p>This is the landing page of <strong>your_domain</strong>.</p>

hora, diríjase a su navegador y acceda al nombre de dominio o la dirección IP de su servidor una vez más:

http://server_domain_or_IP


Con la configuración predeterminada de DirectoryIndex en Apache, un archivo denominado index.html siempre tendrá prioridad sobre un archivo index.php

Si desea cambiar este comportamiento, 

$ sudo nano /etc/apache2/mods-enabled/dir.conf


<IfModule mod_dir.c>
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>

volver a cargar Apache

$ sudo systemctl reload apache2

Paso 4: Probar el procesamiento de PHP en su servidor web

Cree un archivo nuevo llamado info.php dentro de su carpeta root web personalizada:

$ nano /var/www/your_domain/info.php

<?php
phpinfo();


Para probar esta secuencia de comandos,

http://server_domain_or_IP/info.php
