# Backup-DB-Sendgrid


This documentation provides examples for specific use cases from Backups. 
. Thank you!

# Table of Contents

* [Objetivo](#objetivo)
* [Pre-requirements](#requerimientos)
* [Creando carpeta](#creando1)
* [Creando Backup con Cron](#cron1)
* [Creando archivo Logs](#logs1)
* [Configuracion de Php.ini](#phpini)
* [Descargando Libreria Sendgrid](#sendgrid)
* [Codigo para enviar el Backup](#backup1)
* [Cron para enviar Backup](#cron2)
* [Tips](#tips)

<a name="objetivo"></a>
# Objetivo:  
Manual donde se describe las tareas  para realizar backups programados de una Base de Datos, luego  
periodicamente enviarlos a destinatarios via Email, para esto se utilizara la libreria "Sendgrid"

<a name="requerimientos"></a>
# Pre-requirements:  
Servidor Linux  Ubuntu 16.04.  
Servidor Web Apache.  
PHP Version 7.0.  
Servidor de Base de Datos "Mysql".  
Un buen manual para instalar estos pre-requisitos en:  
https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04



<a name="creando1"></a>
# Creando Carpeta para guardar las BD y scripts:
En: /var/backup-bd/


<a name="cron1"></a>
# Cron para ejecutar el Backup. 
Corre cada cuatro horas.  
0 0-23/4 * * * /usr/bin/mysqldump -u root -p****** BDname > /var/backup-bd/BDname.sql
 
<a name="logs1"></a>
# Creando el archivo log de los cron en: /var/log/backup-bd.log.  
Cambiando permisos  
root@aguila2-OEM:/var/backup-bd# chmod 777 /var/log/backup-bd.log



<a name="phpini"></a>
## Configuracion de Php.ini:  
- Ruta: /etc/php/7.0/apache2. 
- Habilitaciones que debemos hacer en el archivo php.ini para poder enviar emails con un archivo  
  adjunto via la libreria sendgrid:  
  


;Windows Extensions  
;Note that ODBC support is built in, so no dll is needed for it.  
;Note that many DLL files are located in the extensions/ (PHP 4) ext/ (PHP 5+)  
;extension folders as well as the separate PECL DLL download (PHP 5+)  
;Be sure to appropriately set the extension_dir directive.  
extension=php_curl.dll  
max_execution_time = 180  

El script de sendgrid tardara mucho en caso de que el tamaño del Backup sea bastante grande, por eso se  
modifica a 180 para que no de error en retardo en tiempo de ejecucion. 

-Instalar Libreria php-curl:  
sudo apt-get install php-curl


<a name="sendgrid"></a>
# Descargando Libreria Sendgrid:

Descargar libreria sendgrid-php desde:  
https://github.com/sendgrid/sendgrid-php.  
Colocarla en directorio de la app, 


<a name="backup1"></a>
# Codigo PHP para enviar el mail con adjunto:  
Nombre de Archivo sendmail.php
Ejemplo tomado de: https://github.com/sendgrid/sendgrid-php/blob/master/USE_CASES.md#attachments

```php
<?php
	
	require("sendgrid-php/sendgrid-php.php");
	$from = new SendGrid\Email("Arquimedes I", "fromfrom@gmail.com");

	$subject = "Backup send";
	
	$to = new SendGrid\Email("Backup enviado", "tototototo@gmail.com");

	$content = new SendGrid\Content("text/plain", "Backup Send");
	
	$file = '/var/backup-bd/xxxxz.sql';   //From Cron
	$file_encoded = base64_encode(file_get_contents($file));
	$attachment = new SendGrid\Attachment();
	$attachment->setContent($file_encoded);
	$attachment->setType("application/text");
	$attachment->setDisposition("attachment");
	$attachment->setFilename("xxxxxz.sql");

	$mail = new SendGrid\Mail($from, $subject, $to, $content);
	$mail->addAttachment($attachment);

	//$apiKey = getenv('SENDGRID_API_KEY');
	$apiKey = 'SG.xVtTDBR_111111111111111111.sZJYxXg3XSX0INGfJLox4-pZZ-25DranfBUtbu4_gqQ';

	$sg = new \SendGrid($apiKey);

	$response = $sg->client->mail()->send()->post($mail);
	echo $response->statusCode();
	print_r($response->headers());
	echo $response->body();

?>

```


 
<a name="cron2"></a>
# Cron para enviar el backup via Sendgrid como archivo adjunto: 
Corren cada cuatro horas desde la 01:00 hasta la 21:00  
0 1,5,9,13,17,21 * * * php /var/backup-bd/sendmail.php >> /var/log/backup-bd.log


<a name="tips"></a>
# Tips Varios: 
Correr un Cron cada minuto  
0 1,5,9,13,17,21 * * * php /var/backup-bd/sendmail.php >> /var/log/backup-bd.log
