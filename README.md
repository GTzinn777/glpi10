# Atualiza Lista de Pacotes
apt update

# Removendo pacotes NTP
apt purge ntp
 
# Instalar pacotes OpenNTPD
apt install -y openntpd
 
# Parando Serviço OpenNTPD
service openntpd stop
 
# Configurar Timezone padrão do Servidor
dpkg-reconfigure tzdata
 
 
# Adicionar servidor NTP.BR
echo "servers pool.ntp.br" > /etc/openntpd/ntpd.conf
 
# Habilitar e Iniciar Serviço OpenNTPD
systemctl enable openntpd
systemctl start openntpd

# Pacotes de manipulação de arquivo
apt install -y xz-utils bzip2 unzip curl

# Instalar dependências no sistema
apt install -y apache2 libapache2-mod-php php-soap php-cas php php-{apcu,cli,common,curl,gd,imap,ldap,mysql,xmlrpc,xml,mbstring,bcmath,intl,zip,redis,bz2}

# Criar arquivo com conteúdo
echo -e "<Directory \"/var/www/html/glpi\">\nAllowOverride All\n</Directory>" > /etc/apache2/conf-available/glpi.conf
 
# Habilita a configuração criada
a2enconf glpi.conf
 
# Reinicia o servidor web considerando a nova configuração
systemctl restart apache2

# Baixar pacote glpi
wget -O- https://github.com/glpi-project/glpi/releases/download/10.0.6/glpi-10.0.6.tgz | tar -zxv -C /var/www/html/

# Ajustar permissões de arquivos
chown www-data. /var/www/html/glpi -Rf
find /var/www/html/glpi -type d -exec chmod 755 {} \;
find /var/www/html/glpi -type f -exec chmod 644 {} \;

# Instalando o Serviço MySQL
apt install -y mariadb-server

# Criando base de dados
mysql -e "create database glpidb character set utf8"
 
# Criando usuário
mysql -e "create user 'YourName'@'localhost' identified by 'YourPassword'"
 
# Dando privilégios ao usuário
mysql -e "grant all privileges on glpidb.* to 'YourName'@'localhost' with grant option";

# Habilitando suporte ao timezone no MySQL/Mariadb
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -p -u root mysql
 
# Permitindo acesso do usuário ao TimeZone
mysql -e "GRANT SELECT ON mysql.time_zone_name TO 'verdanatech'@'localhost';"
 
# Forçando aplicação dos privilégios
mysql -e "FLUSH PRIVILEGES;"

# Remover o arquivo de instalação do sistema
rm -Rf /var/www/html/glpi/install/install.php

# Criar entrada no agendador de tarefas do Linux
echo -e "* *\t* * *\troot\tphp /var/www/html/glpi/front/cron.php" >> /etc/crontab

# Reiniciar agendador de tarefas para ler as novas configurações
service cron restart
