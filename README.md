# Implementação do OTRS 5

Esse manual foi criado com base na documentação original do OTRS5 para ser utilizado no CTEI-MS.


## Etapa 1: Download do source code via wget

```
http://ftp.otrs.org/pub/otrs/otrs-6.0.8.tar.gz
```

Unpack o .tar.gz e mova para /opt/otrs

```
shell> tar xzf /tmp/otrs-x.x.x.tar.gz
shell> mv otrs-x.x.x /opt/otrs
```

## Etapa 2: Instalação dos módulos do Perl

Para verificar se o container ou VM contém todos os módulos necessários execute o seguinte comando:

**Instale os módulos necessários!!!**

```
shell> perl /opt/otrs/bin/otrs.CheckModules.pl
```


## Etapa 3: Crie um usuario no linux

```
shell> useradd -d /opt/otrs -c 'OTRS user' otrs
```

Adicionar usuário ao grupo de servidores web (se o servidor web não estiver sendo executado como usuário OTRS):

```
shell> usermod -G www otrs
(SUSE=www, Red Hat/CentOS/Fedora=apache, Debian/Ubuntu=www-data)
```

## Etapa 4: Ativar o arquivo de configuração padrão

Há um arquivo de configuração do OTRS incluído $OTRS_HOME/Kernel/Config.pm.dist. Você deve ativá-lo copiando-o sem a extensão de arquivo ".dist".

```
shell> cp /opt/otrs/Kernel/Config.pm.dist /opt/otrs/Kernel/Config.pm
```

### Etapa 5: Verifique se todos os módulos necessários estão instalados

```
shell> perl -cw /opt/otrs/bin/cgi-bin/index.pl
/opt/otrs/bin/cgi-bin/index.pl Sintaxe OK

shell> perl -cw /opt/otrs/bin/cgi-bin/customer.pl
/opt/otrs/bin/cgi-bin/customer.pl sintaxe OK

shell> perl -cw /opt/otrs/bin/otrs.Console.pl
/opt/otrs/bin/otrs.Console.pl sintaxe Está bem
```
### Etapa 7: Instale o apache2

```
apt update && apt install apache2 libapache2-mod-perl2 -y
```

### Etapa 8: Configure o apache2

```
shell> ln -s /opt/otrs/scripts/apache2-httpd.include.conf /etc/apache2/sites-enabled/zzz_otrs.conf
```

OTRS requer que alguns módulos do Apache estejam ativos para uma operação ideal. Na maioria das plataformas, você pode garantir que elas estejam ativas através da ferramenta a2enmod.

```
shell> a2enmod perl
shell> a2enmod version
shell> a2enmod deflate
shell> a2enmod filter
shell> a2enmod headers
```

### Etapa 9: Configure as permissões dos arquivos dentro do diretório otrs

```
shell> cd /opt/otrs/
shell> bin/otrs.SetPermissions.pl
```

### Etapa 10: Instale o MySQL

Use o instalador da web em "http: //localhost/otrs/installer.pl" (substitua "localhost" pelo seu nome de host OTRS) para configurar seu banco de dados e configurações básicas do sistema, como contas de e-mail.

```
shell> apt update && apt install mysql-server -y
```

### Etapa 11: Configure o MySQL

Insira essas seguintes linhas no arquivo /etc/my.cnf

```
[mysqld]
max_allowed_packet   = 128M
query_cache_size     = 32M
innodb_log_file_size = 256M
```

## Configuração do OTRS

Este container esta configurado o básico da implementação do OTRS. Por esse motivo é necessário entrar nesse link para finalizar a configuração:

### Etapa 1: Configuração do email

```
http://ip_service/otrs/installer.pl
```

### Etapa 2: Inicie o Daemon OTRS
O novo daemon OTRS é responsável por lidar com quaisquer tarefas assíncronas e recorrentes no OTRS. O que têm sido anteriormente nas definições do arquivo cron agora é tratado pelo daemon OTRS, que agora é necessário para operar o OTRS. O daemon também lida com todos os jobs do GenericAgent e deve ser iniciado a partir do otrsusuário.

```
shell> /opt/otrs/bin/otrs.Daemon.pl start
```

###  Etapa 3: Tarefas Cron para o Usuário OTRS
Existem dois arquivos cron OTRS padrão /opt/otrs/var/cron/*.dist, e sua finalidade é garantir que o Daemon OTRS esteja em execução. Eles precisam ser ativados copiando-os sem a extensão de arquivo ".dist".

``` 
shell> cd /opt/otrs/var/cron
shell> for foo in *.dist; do cp $foo `basename $foo .dist`; done
```  

Para agendar essas tarefas agendadas no seu sistema, você pode usar o script Cron.sh com o otrsusuário.

``` 
shell> /opt/otrs/bin/Cron.sh start
```

Parar as tarefas agendadas também é possível (útil para manutenção):

```
shell> /opt/otrs/bin/Cron.sh stop
```
