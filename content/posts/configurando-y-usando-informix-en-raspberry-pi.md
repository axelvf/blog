---
Description: "Sysarmy - Comunidad de sistemas"
Keywords:
- sysadmin 
- sistemas
- desarrollo
- developer
- administración de sistemas
- administrador de sistemas
- linux
- cloud
- docker
- kubernetes
Section: 
Tags:
- sysarmy
- raspberry
- informix
Thumbnail: assets/raspberrypi4.jpg
socialImage: assets/raspberrypi4.jpg
featuredImage: assets/raspberrypi4.jpg
Title: Configurando y usando Informix en Raspberry PI
Topics:
- Development
- Leadership
- Systems
date: 2015-09-16
markup: html
---

<p>Una vez instalado todo, ahora tenemos que configurar el motor de base de datos y levantarlo.</p>
<p><strong><em>Configuración y primeros pasos para levantar el motor de base de datos</em></strong>.<br />
<strong><em>Como usuario Informix</em></strong></p>
<p>Primero, exportar estas tres variables de ambiente:</p>
<pre><strong>export INFORMIXDIR=/opt/IBM/informix</strong>
<strong>export PATH=$PATH:$INFORMIXDIR/bin</strong>
<strong>export INFORMIXSERVER=ol_informix_raspi</strong>
</pre>
<p><em>Ponerlas en el .bashrc para dejarlas por defecto a nuestro usuario informix.</em></p>
<pre>informix@raspberrypi ~ $ <strong>tail .bashrc</strong>
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).

if [ -f /etc/bash_completion ] &amp;&amp; ! shopt -oq posix; then
. /etc/bash_completion
fi

<em>export INFORMIXDIR=/opt/IBM/informix</em>
<em>export PATH=$PATH:$INFORMIXDI/bin</em>
<em>export INFORMIXSERVER=ol_informix_raspi</em>
</pre>
<p>Copiamos los dos archivos de configuración de ejemplo (.std) por sus homónimos sin extensión.</p>
<pre>informix@raspberrypi ~ $ <strong>cp $INFORMIXDIR/etc/onconfig.std $INFORMIXDIR/etc/onconfig</strong>
informix@raspberrypi ~ $ <strong>cp $INFORMIXDIR/etc/sqlhosts.demo $INFORMIXDIR/etc/sqlhosts</strong>
</pre>
<p>Vamos al directorio de instalación del Informix,</p>
<pre>informix@raspberrypi ~ $ <strong>cd /opt/IBM/informix</strong></pre>
<p>Generamos el directorio chunks (que va a tener los enlaces simbólicos a nuestros raw devices ya generados)</p>
<pre>informix@raspberrypi /opt/IBM/informix $ <strong>mkdir chunks</strong></pre>
<p>Vamos al directorio chunks y generamos los enlaces simbólicos a los <strong>raw devices</strong></p>
<pre>informix@raspberrypi /opt/IBM/informix $ <strong>cd chunks/</strong>
informix@raspberrypi /opt/IBM/informix/chunks $ <strong>ln -s /dev/raw/raw1 rootdbs</strong>
informix@raspberrypi /opt/IBM/informix/chunks $<strong> ln -s /dev/raw/raw2 datadbs00</strong>
informix@raspberrypi /opt/IBM/informix/chunks $ <strong>ln -s /dev/raw/raw3 datadbs01</strong>
informix@raspberrypi /opt/IBM/informix/chunks $ <strong>ln -s /dev/raw/raw4 tempdbs00</strong>
informix@raspberrypi /opt/IBM/informix/chunks $ <strong>ln -s /dev/raw/raw5 logsdbs</strong>
</pre>
<p><em>Verificamos que los enlaces hayan quedado bien.</em></p>
<pre>informix@raspberrypi /opt/IBM/informix/chunks $ <strong>ls -la</strong>
total 8
drwxr-xr-x  2 informix informix 4096 Jun 20 12:47 .
drwxr-xr-x 19 informix informix 4096 Jun 20 11:02 ..
lrwxrwxrwx  1 informix informix   13 Jun 20 11:05 datadbs00 -&gt; /dev/raw/raw2
lrwxrwxrwx  1 informix informix   13 Jun 20 11:06 datadbs01 -&gt; /dev/raw/raw3
lrwxrwxrwx  1 informix informix   13 Jun 20 12:47 logsdbs -&gt; /dev/raw/raw5
lrwxrwxrwx  1 informix informix   13 Jun 20 11:05 rootdbs -&gt; /dev/raw/raw1
lrwxrwxrwx  1 informix informix   13 Jun 20 11:06 tempdbs00 -&gt; /dev/raw/raw4
</pre>
<p>Editamos el archivo $INFORMIXDIR/etc/onconfig</p>
<pre><strong>vi $INFORMIXDIR/etc/onconfig</strong></pre>
<p><em>Buscar las siguientes variables y poner como valor:</em></p>
<pre><strong>ROOTPATH /opt/IBM/informix/chunks/rootdbs</strong>
<strong>DBSERVERNAME ol_informix_raspi</strong>
<strong>TAPEDEV /dev/null</strong>
<strong>LTAPEDEV /dev/null</strong>
<strong>LOGFILES 10</strong>
</pre>
<p>Editamos el archivo $INFORMIXDIR/etc/sqlhosts</p>
<pre><strong>vi $INFORMIXDIR/etc/sqlhosts</strong></pre>
<p>Borramos todo el contenido y agregamos la siguiente línea</p>
<pre><strong>ol_informix_raspi       onsoctcp        localhost       9125</strong></pre>
<p>Una vez hecho esto, inicializamos (algo asi como formatear el disco) la instancia de Informix.</p>
<pre>informix@raspberrypi /opt/IBM/informix/etc $ <strong>oninit -iv</strong>

This action will initialize IBM Informix Dynamic Server;
any existing IBM Informix Dynamic Server databases will NOT be accessible -
Do you wish to continue (y/n)? y   <strong>&lt;--- confirmamos</strong>

Reading configuration file '/opt/IBM/informix/etc/onconfig'...succeeded
Creating /INFORMIXTMP/.infxdirs...succeeded
Allocating and attaching to shared memory...succeeded
Creating resident pool 3238 kbytes...succeeded
Creating infos file "/opt/IBM/informix/etc/.infos.ol_informix_raspi"...succeeded
Linking conf file "/opt/IBM/informix/etc/.conf.ol_informix_raspi"...succeeded
Initializing rhead structure...rhlock_t 16384 (512K)... rlock_t (1875K)... Writing to infos file...succeeded
Initialization of Encryption...succeeded
Initializing ASF...succeeded
Initializing Dictionary Cache and SPL Routine Cache...succeeded
Bringing up ADM VP...succeeded
Creating VP classes...succeeded
Forking main_loop thread...succeeded
Initializing DR structures...succeeded
Forking 1 'soctcp' listener threads...succeeded
Starting tracing...succeeded
Initializing 8 flushers...succeeded
Initializing log/checkpoint information...succeeded
Initializing dbspaces...succeeded
Opening primary chunks...succeeded
Validating chunks...succeeded
Creating database partition...succeeded
Initialize Async Log Flusher...succeeded
Starting B-tree Scanner...succeeded
Init ReadAhead Daemon...succeeded
Init DB Util Daemon...succeeded
Initializing DBSPACETEMP list...succeeded
Init Auto Tuning Daemon...succeeded
Checking database partition index...succeeded
Initializing dataskip structure...succeeded
Checking for temporary tables to drop...succeeded
Updating Global Row Counter...succeeded
Forking onmode_mon thread...succeeded
Creating periodic thread...succeeded
Creating periodic thread...succeeded
Starting scheduling system...succeeded
Verbose output complete: mode = 5
informix@raspberrypi /opt/IBM/informix/etc $
</pre>
<p>Termina de levantar con la linea: <strong>Verbose output complete: mode = 5</strong><br />
Esto significa que el motor arrancó bien. Y, nos devuelve el shell.</p>
<p><em>Ejecutamos</em>: <strong>onstat -</strong></p>
<pre>informix@raspberrypi onstat - 
IBM Informix Dynamic Server Version 12.10.UC4DE -- On-Line -- Up 00:00:39 -- 52656 Kbytes
</pre>
<p>Lo que significa que, afirmativamente tenemos nuestro motor de base de datos andando</p>
<p><em>Ejecutamos</em>: <strong>onstat -m</strong><br />
Y tomamos nota del message log file (<strong>/opt/IBM/informix/tmp/online.log</strong> (lo vamos a necesitar))</p>
<pre>informix@raspberrypi onstat -m
IBM Informix Dynamic Server Version 12.10.UC4DE -- On-Line -- Up 00:00:43 -- 52656 Kbytes

Message Log File: /opt/IBM/informix/tmp/online.log
11:46:07  Loading Module &lt;BUILTINNULL&gt;
11:46:12  DR: DRAUTO is 0 (Off)
11:46:12  DR: ENCRYPT_HDR is 0 (HDR encryption Disabled)
11:46:12  Event notification facility epoll enabled.
11:46:12  IBM Informix Dynamic Server Version 12.10.UC4DE Software Serial Number AAA#B000000
11:46:35  IBM Informix Dynamic Server Initialized -- Complete Disk Initialized.
11:46:35  Started 1 B-tree scanners.
11:46:35  B-tree scanner threshold set at 5000.
11:46:35  B-tree scanner range scan size set to -1.
11:46:35  B-tree scanner ALICE mode set to 6.
11:46:35  B-tree scanner index compression level set to med.
11:46:35  Dataskip is now OFF for all dbspaces
11:46:35  Building 'sysmaster' database ...
11:46:35  Checkpoint Completed:  duration was 0 seconds.
11:46:35  Sat Jun 20 - loguniq 1, logpos 0x3a8, timestamp: 0xae Interval: 2

11:46:35  Maximum server connections 0
11:46:35  Checkpoint Statistics - Avg. Txn Block Time 0.033, # Txns blocked 1, Plog used 4, Llog used 1

11:46:35  On-Line Mode
</pre>
<p>Después le hacemos un tail -f al archivo de mensajes (<strong>/opt/IBM/informix/tmp/online.log</strong>), y, esperamos a que nos aparezca la leyenda: <strong>'sysadmin' database built successfully</strong>. Una vez aparecido ese mensaje, significa que, nuestra base esta 100% lista para usarse.</p>
<pre>informix@raspberrypi /opt/IBM/informix/etc $ <strong>tail -f /opt/IBM/informix/tmp/online.log</strong>

11:46:35  B-tree scanner index compression level set to med.
11:46:35  Dataskip is now OFF for all dbspaces
11:46:35  Building 'sysmaster' database ...
11:46:35  Checkpoint Completed:  duration was 0 seconds.
11:46:35  Sat Jun 20 - loguniq 1, logpos 0x3a8, timestamp: 0xae Interval: 2
11:46:35  Maximum server connections 0
11:46:35  Checkpoint Statistics - Avg. Txn Block Time 0.033, # Txns blocked 1, Plog used 4, Llog used 1

11:46:35  On-Line Mode
11:46:58  Dynamically allocated new virtual shared memory segment (size 16384KB)
11:46:58  Memory sizes:resident:3504 KB, virtual:65536 KB, SHMTOTAL:1048576 KB
11:47:14  Booting Language &lt;spl&gt; from module &lt;&gt;
11:47:14  Loading Module &lt;SPLNULL&gt;
11:47:20  Unloading Module &lt;SPLNULL&gt;
11:47:26  Loading Module &lt;SPLNULL&gt;
11:47:28  Checkpoint Completed:  duration was 0 seconds.
11:47:29  Sat Jun 20 - loguniq 1, logpos 0x122e018, timestamp: 0x1396f Interval: 3

11:47:29  Maximum server connections 1
11:47:29  Checkpoint Statistics - Avg. Txn Block Time 0.000, # Txns blocked 0, Plog used 11, Llog used 4654
11:47:33  Logical Log 1 Complete, timestamp: 0x15bea.
11:47:33  Process exited with return code 127: /bin/sh /bin/sh -c /opt/IBM/informix/etc/alarmprogram.sh 2 23 "Logical Log 1 Complete, timestamp: 0x15bea." "Logical Log 1 Complete, timestamp: 0x15bea." "" 23001
11:47:43  Checkpoint Completed:  duration was 0 seconds.
11:47:43  Sat Jun 20 - loguniq 2, logpos 0x53c018, timestamp: 0x1a7df Interval: 4

11:47:43  Maximum server connections 1
11:47:43  Checkpoint Statistics - Avg. Txn Block Time 0.000, # Txns blocked 0, Plog used 54, Llog used 1686

11:47:59  Checkpoint Completed:  duration was 1 seconds.
11:48:03  Sat Jun 20 - loguniq 2, logpos 0xbd2018, timestamp: 0x215d4 Interval: 5

11:48:03  Maximum server connections 1
11:48:03  Checkpoint Statistics - Avg. Txn Block Time 0.000, # Txns blocked 0, Plog used 58, Llog used 1686

11:48:04  'sysmaster' database built successfully.
11:48:05  'sysutils' database built successfully.
11:48:05  'sysuser' database built successfully.
11:48:11  Building 'sysadmin' database ...
11:48:26  Logical Log 2 Complete, timestamp: 0x297d2.
11:48:26  Process exited with return code 127: /bin/sh /bin/sh -c /opt/IBM/informix/etc/alarmprogram.sh 2 23 "Logical Log 2 Complete, timestamp: 0x297d2." "Logical Log 2 Complete, timestamp: 0x297d2." "" 23001
<strong><em>11:48:43  'sysadmin' database built successfully.</em></strong>
11:48:43  SCHAPI: Started dbScheduler thread.
11:48:43  Auto Registration is synced
11:48:44  Updating Low Memory Manager to version 11
11:48:45  Installing patch to upgrade ph_task code. version(13.04)
11:48:45  SCHAPI: Started 2 dbWorker threads.
11:48:45  SCHAPI: Started Low Memory Manager thread.
11:48:45  Low Memory Manager is ON - Memory Limit 1048576 KB Thread ID 0x47594cd8
11:48:46  Low Memory manager Started
Total(Cap) 1048576 KB
Start Threshold 5120 KB
Stop Threshold 10240 KB
Idle Time 300 Sec
Call Internal Yes
Task Name Low Memory Manager
11:48:48  Checkpoint Completed:  duration was 6 seconds.
11:48:48  Sat Jun 20 - loguniq 3, logpos 0x823018, timestamp: 0x30154 Interval: 6

11:48:48  Maximum server connections 1
11:48:48  Checkpoint Statistics - Avg. Txn Block Time 0.000, # Txns blocked 1, Plog used 153, Llog used 4307
</pre>
<p>Bien tenemos nuestro motor andando. Ahora a darle el disco (raw devices) y hacer unos últimos ajustes. Falta poco. Vamos !</p>
<p><strong>Creando los dbspaces</strong> (un dbspace es un agrupamiento lógico de disco (raw device o file (en este caso raw device)))</p>
<p>Creamos nuestro primer dbspace con uno de los raw devices que hicimos en la entrega previa.</p>
<pre>informix@raspberrypi /opt/IBM/informix/chunks $ <strong>onspaces -c -d datadbs -p /opt/IBM/informix/chunks/datadbs00 -o 0 -s 2097152 </strong>

Verifying physical disk space, please wait ...
Space successfully added.

** WARNING **  A level 0 archive of Root DBSpace will need to be done.
</pre>
<p>Como creamos un <strong>nuevo</strong> dbspace nuevo nos pide un backup full de sistema, todavia no lo vamos a hacer</p>
<p>Agregamos un segundo raw device al dbspace</p>
<pre>informix@raspberrypi /opt/IBM/informix/chunks $ <strong>onspaces -a datadbs -p /opt/IBM/informix/chunks/datadbs01 -o 0 -s 2097152</strong>

Verifying physical disk space, please wait ...
Chunk successfully added.
</pre>
<p>Creamos un dbspace llamado logsdbs de 500Mb</p>
<pre>informix@raspberrypi /opt/IBM/informix/chunks $ <strong>onspaces -c -d logsdbs -p /opt/IBM/informix/chunks/logsdbs -o 0 -s 512000</strong>

Verifying physical disk space, please wait ...
Space successfully added.

** WARNING **  A level 0 archive of Root DBSpace will need to be done.
</pre>
<p>Generamos un dbspace temporal de 500 Mb tambien</p>
<pre>informix@raspberrypi /opt/IBM/informix/chunks $ <strong>onspaces -c -d tempdbs -p /opt/IBM/informix/chunks/tempdbs00 -o 0 -s 512000</strong>

Verifying physical disk space, please wait ...
Space successfully added.

** WARNING **  A level 0 archive of Root DBSpace will need to be done.
</pre>
<p>Nos fijamos el archivo de mensajes:</p>
<pre><em>12:50:13  Space 'datadbs' added.</em>
<em>12:50:29  Chunk '/opt/IBM/informix/chunks/datadbs01' added to space 'datadbs'.</em>
<em>12:51:40  Space 'logsdbs' added.</em>
<em>12:52:04  Space 'tempdbs' added.</em>
</pre>
<p>Ejecutamos el backup completo del sistema</p>
<pre>informix@raspberrypi /opt/IBM/informix/chunks $ <strong>ontape -s -L 0</strong>

Archive to tape device '/dev/null' is complete.
Program over.
</pre>
<p>Por último revisamos que nuestros chunks, estén en buen estado revisando los flags <strong><em>PO</em></strong> (que significan (Primary - Online) ) ejecutando <strong>onstat -d</strong>. </p>
<pre>informix@raspberrypi /opt/IBM/informix/chunks $ <strong>onstat -d</strong>

IBM Informix Dynamic Server Version 12.10.UC4DE -- On-Line -- Up 01:08:01 -- 69040 Kbytes

Dbspaces
address  number   flags      fchunk   nchunks  pgsize   flags    owner    name
4451e8b0 1        0x60001    1        1        2048     N  BA    informix rootdbs
47999b80 2        0x60001    2        2        2048     N  BA    informix datadbs
45161978 3        0x60001    4        1        2048     N  BA    informix logsdbs
47445d98 4        0x60001    5        1        2048     N  BA    informix tempdbs

4 active, 2047 maximum

Chunks
address  chunk/dbs     offset     size       free       bpages     flags pathname
445db018 1      1      0          150000     65432                 <strong><em>PO</em></strong>-B-- /opt/IBM/informix/chunks/rootdbs
44de3018 2      2      0          1048576    1048523               <strong><em>PO</em></strong>-B-- /opt/IBM/informix/chunks/datadbs00
45161018 3      2      0          1048576    1048573               <strong><em>PO</em></strong>-B-- /opt/IBM/informix/chunks/datadbs01
44fc3018 4      3      0          256000     255947                <strong><em>PO</em></strong>-B-- /opt/IBM/informix/chunks/logsdbs
45088018 5      4      0          256000     255947                <strong><em>PO</em></strong>-B-- /opt/IBM/informix/chunks/tempdbs00

5 active, 32766 maximum

NOTE: The values in the "size" and "free" columns for DBspace chunks are
displayed in terms of "pgsize" of the DBspace to which they belong.

Expanded chunk capacity mode: always

informix@raspberrypi /opt/IBM/informix/chunks $
</pre>
<p>Ultimos ajustes: en el  onconfig - Cambiamos:</p>
<p>DBSPACETEMP <strong>tempdbs  </strong>&lt;-- Agregamos tempdbs<br />
DEF_TABLE_LOCKMODE <strong>row</strong> &lt;-- Cambiamos page x row</p>
<p>Bajamos la base</p>
<pre>informix@raspberrypi ~ $ <strong>onmode -yuck</strong></pre>
<p>Chequeamos que haya bajado</p>
<pre>informix@raspberrypi ~ $ <strong>onstat -</strong>

<em>shared memory not initialized for INFORMIXSERVER 'ol_informix_raspi'</em>
</pre>
<p>Levantamos la base para que tome los cambios del default table lock mode y del dbspace temporal. Ejecutamos: <strong>oninit -v</strong></p>
<pre>informix@raspberrypi ~ $ <strong>oninit -v</strong>
Reading configuration file '/opt/IBM/informix/etc/onconfig'...succeeded
Creating /INFORMIXTMP/.infxdirs...succeeded
Allocating and attaching to shared memory...succeeded
Creating resident pool 3238 kbytes...succeeded
Creating infos file "/opt/IBM/informix/etc/.infos.ol_informix_raspi"...succeeded
Linking conf file "/opt/IBM/informix/etc/.conf.ol_informix_raspi"...succeeded
Initializing rhead structure...rhlock_t 16384 (512K)... rlock_t (1875K)... Writing to infos file...succeeded
Initialization of Encryption...succeeded
Initializing ASF...succeeded
Initializing Dictionary Cache and SPL Routine Cache...succeeded
Bringing up ADM VP...succeeded
Creating VP classes...succeeded
Forking main_loop thread...succeeded
Initializing DR structures...succeeded
Forking 1 'soctcp' listener threads...succeeded
Starting tracing...succeeded
Initializing 8 flushers...succeeded
Initializing SDS Server network connections...succeeded
Initializing log/checkpoint information...succeeded
Initializing dbspaces...succeeded
Opening primary chunks...succeeded
Validating chunks...succeeded
Initialize Async Log Flusher...succeeded
Starting B-tree Scanner...succeeded
Init ReadAhead Daemon...succeeded
Init DB Util Daemon...succeeded
Initializing DBSPACETEMP list...succeeded
Init Auto Tuning Daemon...succeeded
Checking database partition index...succeeded
Initializing dataskip structure...succeeded
Checking for temporary tables to drop...succeeded
Updating Global Row Counter...succeeded
Forking onmode_mon thread...succeeded
Creating periodic thread...succeeded
Creating periodic thread...succeeded
Starting scheduling system...succeeded
Verbose output complete: mode = 5
</pre>
<p>Chequeamos que haya levantado</p>
<pre>informix@raspberrypi ~ $ <strong>onstat -</strong>

IBM Informix Dynamic Server Version 12.10.UC4DE -- On-Line -- Up 00:00:14 -- 69040 Kbytes
</pre>
<p>Y, por ultimo, generamos la base de datos de ejemplo (notar que instruimos que use el dbspace datadbs para la base)</p>
<pre>informix@raspberrypi ~ $ <strong>dbaccessdemo -log -dbspace datadbs</strong>

DBACCESS  Demonstration Database Installation Script
Dropping existing stores_demo database ....
Creating stores_demo database ....
Lockmode set.
Database created.
Database closed.
Database selected.

BLA BLA BLA BLA

Database closed.
The creation of the demonstration database is now complete.  The remainder
of this script copies the examples into your current directory.
Press "Y" to continue, or "N" to abort. &lt;-- Es conveniente apretar Y para después chusmear los scripts de creación.

Now copying SQL command files ....
End of DBACCESSDEMO script.
</pre>
<p>Y ahora nuestra primera consulta:</p>
<pre>informix@raspberrypi ~ $<strong> time dbaccess stores_demo &lt;&lt; EOM</strong>
<strong> &gt; select count(*) from customer;</strong>
<strong> &gt; EOM</strong>

Database selected.
(count(*))
28

1 row(s) retrieved.

Database closed.
real 0m0.139s
user 0m0.060s
sys 0m0.020s
</pre>
<p>Mas adelante iré explicando, muy de a poco algunos aspectos internos de Informix, como asi tambien la interfaz de consultas dbaccess y onstat para monitorizarlo.</p>
<p>Un enlace para ir leyendo un poco</p>
<p><a href="http://www-01.ibm.com/support/knowledgecenter/SSGU8G_12.1.0/com.ibm.welcome.doc/welcome.htm" target="_blank">http://www-01.ibm.com/support/knowledgecenter/SSGU8G_12.1.0/com.ibm.welcome.doc/welcome.htm</a></p>
<p>Que es el sitio de documentacion oficial de IBM</p>
<p>Dudas? consultas? sugerencias ?</p>
<p>Hasta el próximo capítulo !!!</p>
