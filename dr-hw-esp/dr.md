# Arq. de tolerancia a fallas: DR entre regiones de OCI

## Introduction

Esta arquitectura proporciona una réplica de un DBSystem en una región secundaria, utilizando Inbound Replication y un Canal de Replcación, para más información, consulte: https://docs.oracle.com/es-ww/iaas/mysql-database/doc/overview-inbound-replication.html


### Prerequisites

This lab assumes you have:
* Región destino habilitada en su tenant
* VCN Peering entre ambas regiones
* Backup policy activa en base origen
* Retención de binlogs de 7 días (recomendado) 



## Prerequisito: Activa una mayor retención de bonlogs

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

  > **Note:** Repite este proceso en AMBAS regiones

	![Image alt text](images/sample1.png)


## Configurando la replicación

1.	Seleccione el último backup automático del DBSystem  origen y de click en Copy to another region
 

  > **Note:**  Este backup debe haberse realizado DESPUES del cambio de variable binlog_expire_logs_seconds, de lo contrario, realice un backup manual después de realizar este cambio. 

2.	Seleccione la región destino y copia el backup. 
3.	Cuando la copia termine en la región destino, de click en Restore to New DBSystem
4.	Seleccione Restore from backup, configure el DBSystem
 

5.	En la sección de Configuración, asegurese de seleccionar la configuración creada anteriormente con la variable binlog_expire_logs_seconds modificada
 

6.	De click en Restore
7.	En el Dbsystem origen ejecute las siguientes instrucciones:
CREATE USER 'repl'@'%' IDENTIFIED BY '<password>';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

  > **Note:**  Asegurese de reemplazar <password> por una contraseña definida por usted
8.	Una vez que termine la creación del Dbsystem, de click en Menu principal > Databases> Channels
 

9.	De click en create Channel
 
10.	En el campo hostname, ingrese la IP del DBSystem origen
11.	Posteriormente ingrese la información del usuario creado anteriormente y seleccione la opción DISABLED
 
12.	Seleccione la opción Recomendada de auto-posicionamiento de GTIDs
 
13.	Seleccione el Dbsystem destino 


14.	De click en Create Chanel 
15.	Verifique el funcionamiento del canal, ejecute la siguiente instrucción en el DBSystem destino con un usuario administrador: 
Show replica status\G
16.	La replicación es funcional si la salida del comando anterior es similar a la imagen: 
 

17.	También puede probar realizando actualizaciones de datos en el Dbsystem origen y verificando el cambio en el destino.

    ```
    Adding code examples
  	Indentation is important for the code example to appear inside the step
    Multiple lines of code
  	<copy>Enclose the text you want to copy in <copy></copy>.</copy>
    ```

4. Code examples that include variables

	```
  <copy>ssh -i <ssh-key-file></copy>
  ```

## Simulando un error de replicación (con GTIDs)

1.	Ejecute la siguiente instrucción en la el DBSystem origen para crear una base de datos de prueba
create database test;
2.	Elimine esta base de datos en el DBSystem destino 
drop database test;
3.	Elimine esta base de datos en el DBSystem origen   
   drop database test;
4.	Esta operación provocará un error en la replicación, ya que la instrucción de borrar se replicará pero esta base ya no existe en la base destino.
5.	Ejecute la siguiente instrucción en en el DBSystem destino
   Show replica status\G

6.	La replicación presenta un error. Si tiene configurada una alarma, la notificación puede tardar un par de minutos en llegar.


## Reparando la replicación (con GTIDs)

7.	Ejecute las siguientes instruccione en el DBSystem destino 
Show replica status\G
       select * from performance_schema.error_log
       where SUBSYSTEM = 'Repl' and PRIO = 'Error'
       order by 1 desc limit 2\G

8.	Al consultar el error log (segunda instrucción del paso anterior), obtendremos más información del error que provocó que la replicación se rompiera: 
9.	Copie la transacción que causó el error, por ejemplo: “c92bbb50-59a4-11ef-ba1b-02001723c256:4” 
10.	Ejecute las siguientes instruccione en el DBSystem destino:
SET GTID_NEXT='aaa-bbb-ccc-ddd:N';
BEGIN;
COMMIT;
SET GTID_NEXT='AUTOMATIC';

Asegurese de reemplazar 'aaa-bbb-ccc-ddd:N' por la transacción que copió en el paso anterior
11.	Vaya al Canal de Replicación, vaya a menu principal > Databases > Channels e ingrese al canal afectado
 

12.	Haga click en Deshabiltar o Disable
 

13.	Cuando termine, de click en “Reset” 
 

14.	Finalmente, de click en Enable o Habilitar
 
15.	Cuando la operación termine, ejecute la siguiente instruccion en el DBSystem destino:
Show replica status\G
16.	La replicación debe ser funcional nuevamente: 



## Regresando a la región primaria

1.	Selecciona el último backup automático del DBSystem que se convertirá en el DR y da click en Copy to another region
 
2.	Selecciona la región destino y copia el backup. 
3.	Cuando la copia termine, da click en Restore to New DBSystem
4.	Selecciona Restore from backup, configura el DBSystem
 

5.	En la sección de Confguración, asegurese de seleccionar la configuración creada anteriormente con la variable binlog_expire_logs_seconds modificada
 
6.	De click en Restore
7.	Una vez que termine la creación del Dbsystem, de click en Menu principal > Databases> Channels
 

8.	De click en create Channel, este proceso se realizará en la región primaria
 
9.	En el campo hostname, ingrese la IP del DBSystem origen (que actualmente está en región secundaria)
10.	Posteriormente ingrese la información del usuario repl y seleccione la opción DISABLED
 
11.	Seleccione la opción Recomendada de auto-posicionamiento de GTIDs
 
12.	Seleccione el Dbsystem destino (ubicado en la región primaria)


13.	De click en Create Chanel 
14.	Verifique el funcionamiento del canal, ejecute la siguiente instrucción en el DBSystem destino (ubicado en la región primaria) con un usuario administrador: 
Show replica status\G
15.	La replicación debe es funcional si la salida del comando anterior es similar a la imagen: 
 

16.	También puede probar realizando actualizaciones de datos en el Dbsystem origen y verificando el cambio en el destino.
17.	Importante: Cuando la variable de la imagen anterior “Seconds_behind_Source” Se encuentra en 0 constantemente, significa que en este punto ya tenemos un disaster recovery activo, se tiene una replicación “inversa”, es decir, Región Secundaria -> Región Primaria. Se necesitará una ventana de mantenimento para su aplicativo para realizar el cambio: dejar de apuntar a la región secundaria y apuntar a la región primaria, cuando este listo para tener esta ventana, continue: 
18.	Desconecte su aplicativo del DBSystem ubicado en la región secundaria
19.	Verifique que la variable Seconds_behind_Source  se encuentre en CERO:
Show replica status\G
 

20.	Detenga el canal de replicación ubicado en la región primaria, acceda a su canal de replicación ubicado en la región secundaria 
21.	De click en “Enable” y posteriormente en Editar
22.	Cambie el valor de Hostname a la IP del DBSystem recién creado (nuevo origen)
 


23.	Verifique que la replicación está funcionando, ejecute la siguiente instruccion en el DBSystem destino:
Show replica status\G
24.	El resultado debe ser similar a la siguiente imagen


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
