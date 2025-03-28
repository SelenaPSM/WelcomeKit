# Arq. híbrida: Replicando a HeaWave utilizando GTIDs

## Introducción

Esta arquitectura proporciona una réplica de un ambiente MySQL on-Prem a un DBsystem en OCI, utilizando Inbound Replication y un Canal de Replcación, para más información, consulte: https://docs.oracle.com/es-ww/iaas/mysql-database/doc/overview-inbound-replication.html

# Pre-requisitos: 

* Cuenta de OCI habilitada
* Active una mayor retención de Binlogs en su ambiente onPrem a 7 días (recomendado)
* La versión de MySQL on Prem debe ser mínimo 5.7 para establecer una replicación con la versión Bug Fix 8.0, si necesita replicar a una versión posterior (8.4, 9.0) su ambiente onPrem debe ser 8.0.X 


## Pre-requisitos de configuración

1. Verifique las siguientes variables globales y su valor: 


    binlog_expire_logs_seconds=691200 #7 días sugerido
Activar GTID
gtid_mode=ON
enforce_gtid_consistency=ON
Setear binlog format ROW
binlog_format=ROW

1.	Da click en el menú de hamburguesa, después en Databases y Configurations 
 
2.	De click en Create Configuration
 
3.	Asigna un nombre, selecciona un compartimento y el shape deseado
NOTA: El shape selecconado debe ser igual al shape actual de la base origen
 
4.	Añade la variable  binlog_expire_logs_seconds, cambia el valor a 604800 y al terminar, da click en Create
 
5.	Da click en el menú de hamburguesa, después en Databases y en DB System 
 
6.	Selecciona Edit en el apartado de Shape
 
7.	Seleccione “Cambiar confguración” y seleccione la configuración que creamos
 
8.	Verifica que el mensaje mostrado NO mencione que la base de datos se reiniciará, ya que la variable binlog_expire_logs_seconds es dinámica
9.	Da click en Save Changes, la base se mostrará en estado “Updating” o “Actualizando”, sin embargo, al haber realizado un cambo de una variable dinámica, la disponibildad del servicio no se verá afectado. 

  > **Note:**  Repite este proceso en AMBAS regiones





## Documentación y links importantes

*(optional - include links to docs, white papers, blogs, etc)*

* [URL text 1](http://docs.oracle.com)
* [URL text 2](http://docs.oracle.com)

## Feedback

Si tienes dudas, o deseas darnos retroalimentación de esta guía, porfavor envía un correo a:

* selena.sanchez@oracle.com 

IMPORTANTE: En el Subject del correo coloca al inicio [WELCOME KIT EMAIL], de forma que tu correo sea filtrado y atendido correctamente. 

## Acknowledgements
* **Author** - Selena Sánchez, MySQL Staff Solutions Engineer; Cristian Aguilar, MySQL Staff Solutions Engineer; Oscar Cárdenas, MySQL Staff Solutions Engineer.
* **Contributors** -  MySQL LAD Solutions Engineer Team
* **Last Updated By/Date** - Selena Sánchez, MySQL Staff Solutions Engineer, Marzo 2025
