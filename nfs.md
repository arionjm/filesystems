NFS

NFS (sistema de archivos de red: «Network File System») es un protocolo que permite acceso remoto a un sistema de archivos a través de la red. Todos los sistemas Unix pueden trabajar con este protocolo; cuando se involucran sistemas Windows, debe utilizar Samba en su lugar.
NFS es una herramienta muy útil. Si bien anteriormente ha tenido muchas limitaciones, la mayoría ha desaparecido con la versión 4 del protocolo. El inconveniente es que la última versión de NFS es más difícil de configurar cuando se quieren utilizar funciones básicas de seguridad como la autenticación y el cifrado, puesto que se basa en Kerberos para estas funcionalidades. Sin éstas, el protocolo NFS tiene que restringirse a la utilización en una red local de confianza puesto que los datos que circulan por la red no están cifrados (un sniffer los puede interceptar) y los permisos de acceso se conceden en función de la dirección IP del cliente (que puede ser suplantada).

Protección de NFS
Si no se utilizan las características de seguridad basadas en Kerberos, debería asegurarse de que sólo los equipos autorizados a utilizar NFS puedan conectarse a los varios servidores RPC necesarios, porque el protocolo básico confía en la información recibida a través de la red. El firewall debería por tanto prohibir la usurpación de IPs («IP spoofing») para prevenir que una máquina externa se haga pasar por una interna y el acceso a los puertos apropiados debería estar restringido únicamente a los equipos que deban acceder a espacios compartidos por NFS.
Las versiones antiguas del protocolo requerían otros servicios RPC que utilizaban puertos asignados dinámicamente. Afortunadamente, con la versíon 4 de NFS solo son necesarios los puertos 2049 (para el NFS propiamente) y el 111 (para el portmapper), por lo que son fáciles de filtrar mediante un cortafuegos.

Servidor NFS
El servidor NFS es parte del núcleo Linux; en los núcleos que Debian provee está compilado como un módulo de núcleo. Si necesita ejecutar el servidor NFS automáticamente al iniciar, debe instalar el paquete nfs-kernel-server; contiene los scripts de inicio relevantes.
El archivo de configuración del servidor NFS, /etc/exports, enumera los directorios que estarán disponibles en la red (exportados). Para cada espacio compartido NFS, sólo tendrán acceso las máquinas especificadas. Puede obtener un control más detallado con unas pocas opciones. La sintaxis para este archivo es bastante simple:

/directorio/a/compartir maquina1(opcion1,opcion2,...) maquina2(...) ...

Es importante recalcar que con NFSv4 todas las carpetas exportadas deben ser parte de un único árbol de directorios y que el directorio raíz de este árbol debe ser exportado e identificado con la opción fsid=0 o fsid=root.
Puede identificar cada máquina mediante su nombre DNS o su dirección IP. También puede especificar conjuntos completos de máquinas utilizando una sintaxis como *.falcot.com o un rango de direcciones IP 192.168.0.0/255.255.255.0 o 192.168.0.0/24.
De forma predeterminada (o si utiliza la opción ro), los directorios están disponibles sólo para lectura. La opción rw permite acceso de lectura y escritura. Los clientes NFS típicamente se conectan desde un puerto restringido sólo a root (en otras palabras, menor a 1024); puede eliminar esta restricción con la opción insecure (la opción secure es implícita, pero puede hacerla explícita para más claridad).
De forma predeterminada, el servidor sólo responderá a peticiones NFS cuando se complete la operación actual de disco (la opción sync); puede desactivar esta funcionalidad con la opción async. Las escrituras asíncronas aumentarán un poco el rendimiento a cambio de una disminución de la fiabilidad, debido al riesgo de pérdida de datos en caso de un cierre inesperado del servidor durante el tiempo que transcurre entre que se recibe la petición de escritura y cuando los datos hayan sido escritos realmente en el disco. Debido a que el valor predeterminado cambió recientemente (comparado con el valor histórico de NFS), se recomienda configurarlo explícitamente.
Para no proveerle acceso de root al sistema de archivos a ningún cliente NFS, el servidor considerará todas las consultas que parezcan provenir de un usuario root como si provinieran del usuario nobody. Este comportamiento corresponde a la opción root_squash y está activado de forma predeterminada. La opción no_root_squash, que desactiva este comportamiento, es riesgosa y sólo debe ser utilizada en entornos controlados. Las opciones anonuid=uid y anongid=gid permiten especificar otro usuario falso que será utilizado en lugar deñ UID/GID 65534 (que corresponden al usuario nobody y al grupo nogroup).
Con NFSv4 se puede añadir la opción sec para precisar el nivel de seguridad deseado: sec=sys es el valor predeterminado sin ningún tipo de seguridad particular, sec=krb5 habilita únicamente la autenticación, sec=krb5i añade una protección de integridad y sec=krb5p es el nivel más alto, que incluye la protección de la confidencialidad (mediante el cifrado de datos). Para que todo esto pueda funcionar es necesaria una instalación funcional de Kerberos (este libro no se trata en este libro).

Cliente NFS
Como con cualquier otro sistema de archivos, incorporar un espacio compartido NFS en la jerarquía del sistema es necesario montarlo. Debido a que este sistema de archivos tiene sus peculiaridades fueron necesarios unos pocos ajustes en la sintaxis de mount y en el archivo /etc/fstab.

Ejemplo 11.22. Montaje manual con el programa mount

          # mount -t nfs4 -o rw,nosuid arrakis.internal.falcot.com:/shared /srv/shared

Ejemplo 11.23. Elemento NFS en el archivo /etc/fstab

arrakis.internal.falcot.com:/shared /srv/shared nfs4 rw,nosuid 0 0

El elemento descrito monta automáticamente en cada arranque del sistema el directorio NFS /shared/ desde el servidor arrakis en el directorio local /srv/shared/. Se solicita acceso de lectura y escritura (de ahí el parámetro rw). La opción nosuid es una medida de protección que elimina cualquier bit setuid o setgid de los programas almacenados en el espacio compartido. Si el espacio compartido NFS está destinado únicamente a almacenar documentos, también se recomienda utilizar la opción noexec, que evita la ejecución de programas almacenados en el espacio compartido. Es importante tener en cuenta que, en el servidor, el directorio shared se encuentra dentro del directorio exportado como raíz de NFSv4 (por ejemplo /export/shared), no es un directorio de primer nivel.
