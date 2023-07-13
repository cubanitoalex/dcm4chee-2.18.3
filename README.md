# Instalar DCM4CHEE 2.18.3 en Debian 12.

**Empezamos con actualizar el sistema.**

1- apt update && apt upgrade -y <br>
2- Descargar estos fiheros:<br>

DCM4CHEE-2.18.3 <br>
https://sourceforge.net/projects/dcm4che/files/dcm4chee/2.18.3/dcm4chee-2.18.3-mysql.zip/download<br>

JBOS-4.2.3<br>
https://sourceforge.net/projects/jboss/files/JBoss/JBoss-4.2.3.GA/jboss-4.2.3.GA-jdk6.zip/download<br>

De aqui descargar jdk-7u80-linux-x64.tar.gz<br>
https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html<br>

3- mkdir -p /usr/local/java<br>
4- cp -r jdk-7u80-linux-x64.tar.gz /usr/local/java/<br>
5- cd /usr/local/java<br>
6- tar xvzf jdk-7u80-linux-x64.tar.gz<br>
7- nano /etc/profile vas hasta el final del archivo /etc/profile y agregas las siguientes lineas:<br>
JAVA_HOME=/usr/local/java/jdk1.7.0_80<br>
JRE_HOME=/usr/local/java/jdk1.7.0_80 <br>
PATH=$PATH:$JRE_HOME/bin:$JAVA_HOME/bin<br>
export JAVA_HOME<br>
export JRE_HOME<br>
export PATH<br>
8-Actualizas las alternativas:<br>
update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/jdk1.7.0_80/bin/java" 1<br>
update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.7.0_80/bin/javac" 1<br>
update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/local/java/jdk1.7.0_80/bin/javaws" 1<br>
update-alternatives --set java /usr/local/java/jdk1.7.0_80/bin/java<br>
update-alternatives --set javac /usr/local/java/jdk1.7.0_80/bin/javac<br>
update-alternatives --set javaws /usr/local/java/jdk1.7.0_80/bin/javaws<br>
9-Recargas el profile:<br>
source /etc/profile<br>
10-Verificas la instalacion del java con el comando:<br>
java -version<br>
Te debe devolver esto:<br>
java version "1.7.0_80"<br>
Java(TM) SE Runtime Environment (build 1.7.0_80-b15)<br>
Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)<br>
11-Instalamos Mysql Server pero aca necesitamos una version antigua por que las actuales dan error con los indices al importar las bd.<br>
wget https://dev.mysql.com/get/mysql-apt-config_0.8.18-1_all.deb<br>
dpkg -i mysql-apt-config_0.8.18-1_all.deb<br>
Tambien pueden agregar manual en /etc/apt/sources.list<br>
deb http://repo.mysql.com/apt/debian/ buster mysql-apt-config<br>
deb http://repo.mysql.com/apt/debian/ buster mysql-5.7<br>
deb http://repo.mysql.com/apt/debian/ buster mysql-tools<br>
deb-src http://repo.mysql.com/apt/debian/ buster mysql-5.7<br>
apt update<br>
Les dara este error no se asusten<br>
The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 467B942D3A79BD29<br>
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 467B942D3A79BD29<br>
apt update<br>
apt install -y mysql-community-server<br>
![Screenshot](1.webp)<br>
![Screenshot](2.webp)<br>
![Screenshot](3.webp)<br>
![Screenshot](4.webp)<br>
sudo systemctl restart mysql<br>
sudo systemctl enable mysql<br>
escribimos <br>
mysql_secure_installation<br>
y van respondiendo las preguntas segun le interese por mi parte lo deje todo por defecto.<br>
12- Escribimos mysql -u root -p<br>
Creas la siguiente estructura:<br>
mysql> create schema pacsdb;  <br>
mysql> grant all on pacsdb.* to pacs@localhost identified by 'pacs';  <br>
mysql> flush privileges;  <br>
mysql> \q<br>
12-Extraes dcm4chee-2.18.3-mysql.zip y jboss-4.2.3.GA-jdk6.zip en mi caso lo hice en /opt/<br>
13-Como usamos un sistema de 64 bits, y la estructura del dcm4chee es de 32 bits tenemos que parcharlo:<br>
wget http://download.java.net/media/jai-imageio/builds/release/1.1/jai_imageio-1_1-lib-linux-amd64.tar.gz<br>
tar xzvf jai_imageio-1_1-lib-linux-amd64.tar.gz<br>
cp jai_imageio-1_1/lib/libclib_jiio.so /opt/dcm4chee-2.18.3-mysql/bin/native/libclib_jiio.so<br>
14-Ahora instalamos jboss:<br>
cd /opt/dcm4chee-2.18.3-mysql/bin/  <br>
./install_jboss.sh /opt/jboss-4.2.3.GA<br>
15-Importamos los indices:<br>
cd /opt/dcm4chee-2.18.3-mysql/sql/ <br>
mysql -upacs -p pacsdb < create.mysql<br>
Les pidira la clave del usuario pacs que es la misma en la contraseña "pacs", esperen que importe.<br>
16-Para instalar oviyam2 y el ioviyam2<br>
copiamos el oviyam2.war y ioviyam2.war dentro de la carpeta /opt/dcm4chee/server/default/deploy<br>
copiamos la configuracion "iOviyam.properties" /opt/dcm4chee-2.18.3-mysql/server/default<br>
copiamos la configuracion "oviyam2-7-config.xml" /opt/dcm4chee-2.18.3-mysql/server/default/work/jboss.web/localhost<br>
y ejecutamos<br>
/opt/dcm4chee/bin/run.sh &<br>
17-Para entrar al dcm4chee seria http://ip:8080/dcm4chee-web3/ , para configurar el jbos http://ip:8080/jmx-console/, para consultar el oviyam version escritorio<br> http://ip:8080/oviyam2, para consultar ioviyan version movil http://ip:8080/ioviyam2.<br>
18-Tambien le agregue nginx con ssl para darle seguridad al login quedaria de esta forma la configuracion:<br>
upstream dcm4chee {
   server 127.0.0.1:8080;
}
server {
 listen 80;
 charset UTF-8;
 server_name dcm4hlucia.hlg.sld.cu;
 rewrite ^ https://$server_name$request_uri? permanent;      
}
server {
   listen 443;
   server_name dcm4hlucia.hlg.sld.cu;
   charset utf-8;
    ssl on;
   ssl_certificate /etc/apache2/ssl/fullchain.pem;
   ssl_certificate_key /etc/apache2/ssl/privkey.pem;
    ssl_session_timeout  5m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
   ssl_ciphers         HIGH:!aNULL:!MD5;
   ssl_prefer_server_ciphers   on;
   location / {
       return 301 /dcm4chee-web3;
   }
   location /wado {
       proxy_buffering off;
       proxy_pass  http://dcm4chee/wado;
       proxy_set_header Host $host;
   }
   location /dcm4chee-web3 {
       proxy_buffering off;
       proxy_redirect off;
       proxy_pass  http://dcm4chee/dcm4chee-web3;
       proxy_set_header Host $host;
   }    
}
