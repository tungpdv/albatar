FROM ubuntu:18.04

MAINTAINER Sebastien Macke <lanjelot@gmail.com>

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND=noninteractive 

RUN MYSRP=Password1; \
 { echo "mysql-server mysql-server/root_password password $MYSRP"; \
   echo "mysql-server mysql-server/root_password_again password $MYSRP"; \
 } | debconf-set-selections \
 && apt-get update && apt-get install -y mysql-server \
 && sed -i -e "s/bind-address.*/bind-address = 0.0.0.0/" -e 's,^#general_log,general_log,' /etc/mysql/mysql.conf.d/mysqld.cnf \
 && echo secure_file_priv= >> /etc/mysql/mysql.conf.d/mysqld.cnf \
 && Q1="GRANT ALL ON *.* TO 'root'@'%' IDENTIFIED BY '$MYSRP' WITH GRANT OPTION;" \
 && Q2="FLUSH PRIVILEGES;" \
 && SQL="${Q1}${Q2}" \
 && rm -f /etc/apparmor.d/usr.sbin.mysqld \
 && service mysql start \
 && mysql -uroot -p"$MYSRP" -e "$SQL"

RUN apt-get update && apt-get install -y php libapache2-mod-php php-mysql apache2 \
 && echo 'ServerName localhost' >> /etc/apache2/apache2.conf

RUN apt-get update && apt-get install -y iproute2 lsof vim wget curl gnupg2 unzip \
 && echo 'set bg=dark' > /root/.vimrc

RUN curl -s https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
RUN curl -s -o /etc/apt/sources.list.d/msprod.list https://packages.microsoft.com/config/ubuntu/18.04/prod.list 
RUN apt-get update && ACCEPT_EULA=Y apt-get install -y msodbcsql17 mssql-tools unixodbc-dev php-pear php7.2-dev \
 && pecl install sqlsrv \
 && echo extension=sqlsrv.so > /etc/php/7.2/apache2/conf.d/20-sqlsrv.ini

WORKDIR /opt/oracle
RUN wget https://download.oracle.com/otn_software/linux/instantclient/instantclient-basiclite-linuxx64.zip \
 && wget https://download.oracle.com/otn_software/linux/instantclient/instantclient-sdk-linuxx64.zip \
 && wget https://download.oracle.com/otn_software/linux/instantclient/instantclient-sqlplus-linuxx64.zip \
 && unzip instantclient-basiclite-linuxx64.zip \
 && rm -f instantclient-basiclite-linuxx64.zip \
 && unzip instantclient-sdk-linuxx64.zip \
 && rm -f instantclient-sdk-linuxx64.zip \
 && unzip instantclient-sqlplus-linuxx64.zip \
 && rm -f instantclient-sqlplus-linuxx64.zip \
 && mv /opt/oracle/instantclient_* /opt/oracle/instantclient \
 && echo /opt/oracle/instantclient > /etc/ld.so.conf.d/oracle-instantclient.conf \
 && ldconfig

RUN echo instantclient,/opt/oracle/instantclient | pecl install oci8 \
 && echo extension=oci8.so > /etc/php/7.2/apache2/conf.d/20-oci8.ini

RUN echo -e "echo Starting services\n\
service mysql start\n\
service apache2 start\n\
bash /var/www/html/demo/setup-mysql.sh\n\
#tail -f /var/log/apache2/error.log\n\
tail -f /dev/null\n" > /usr/local/sbin/start-all-services.sh

CMD ["bash", "/usr/local/sbin/start-all-services.sh"]
