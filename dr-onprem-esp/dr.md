# Arq. híbrida: Replicando a HeaWave utilizando binlogs 

## Introducción

Esta arquitectura proporciona una réplica de un ambiente MySQL on-Prem a un DBsystem en OCI, utilizando Inbound Replication y un Canal de Replcación, para más información, consulte: https://docs.oracle.com/es-ww/iaas/mysql-database/doc/overview-inbound-replication.html

# Pre-requisitos: 

* Cuenta de OCI habilitada
* Active una mayor retención de Binlogs en su ambiente onPrem a 7 días (recomendado)
* La versión de MySQL on Prem es importante, verifique los requisitos mencionados más adelante


## Pre-requisitos de configuración y versionamiento

1. Verifique las siguientes variables globales y su valor para versión onPrem 5.7.X

    binlog_expire_logs_days=7 días sugerido
    binlog format ROW
    binlog_format=ROW

    HeatWave soporta la replicación únicamente en formato ROW 
    > **Note:**  Con una versión del servidor source 5.7.X únicamente es posible replicar a un DBSystem con versión 8.0.X 


2. Verifique las siguientes variables globales y su valor para versión onPrem 8.0.X 
    binlog_expire_logs_seconds=691200 #7 días sugerido
    binlog format ROW
    binlog_format=ROW

    HeatWave soporta la replicación únicamente en formato ROW 
    > **Note:**  Con una versión del servidor source 8.0.X es posible replicar a un DBSystem con versión 8.0.X y 8.4.X 

## Realice un dump con MySQL Shell 

1. Si desea replicar todo su ambiente local:

2. Si desea replicar únicamente ciertos schemas: 

## Cree un DBsystem 

1. 

2. 

    Seleccione el versionamiento adecuado, según la versión de su ambiente onPrem

## Restaure el dump en HeatWave MySQL

## Estableciendo el canal de replicación 

## Simulando un error de replicación

## Reparando la replicación


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