# Backup-DB-Sendgrid


This documentation provides examples for specific use cases from Backups. 
. Thank you!

# Table of Contents

* [Objetivo](#objetivo)
* [Creando carpeta](#creando1)
* [Creando Backup con Cron](#cron1)
* [Creando archivo Logs](#logs1)
* [Codigo para enviar el Backup](#backup1)
* [Cron para enviar Backup](#cron2)

<a name="objetivo"></a>
# Objetivo:  
Manual donde se describe las tareas  para realizar backups automaticos de una Base de 
de Datos.


<a name="creando1"></a>
# Creando Carpeta para guardar las BD y scripts:
/var/backup-bd/


<a name="cron1"></a>
# Creando Backup con un Cron. 
Corre cada cuatro horas.
0 0-23/4 * * * /usr/bin/mysqldump -u root -p****** BDname > /var/backup-bd/BDname.sql >> /var/log/backup-bd.log.
 
<a name="logs1"></a>
# creando el archivo log de los cron en /var/log/backup-bd.log.
Cambiando permisos
root@aguila2-OEM:/var/backup-bd# chmod 777 /var/log/backup-bd.log

<a name="backup1"></a>
# Codigo PHP para enviar el mail con adjunto:  
Nombre de Archivo envia.php
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
0 0,4,8,12,16,20 * * * php /var/backup-bd/envia.php >> /var/log/backup-bd.log

