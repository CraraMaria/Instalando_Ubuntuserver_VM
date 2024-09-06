# Instalando_Ubuntuserver_VM
como baixar o ubuntu e usando MYSQL

###################################################
Aqui encontram-se todos os comando que serão utilizados nos LABS.
Procure entender os comandos.
Execute e vá tirando os prints de tela 
para facilitar na hora da produção do artigo.
##################################################

teste o Open SSH com o comando:

$ssh localhost

caso conection refused executar:
$sudo apt install openssh-server

Construir ambiente MySQL e conectar com o Cliente DBeaver

1. instalar ubuntu server no Virtualbox ativando o openSSH

2. No Virtualbox redirecione a porta 2222 do hospedeiro para 22 da máquina convidada.(agora vc pode conectar via putty)

3. comando para atualizar o linux e instalar o Mysql
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt install mysql-server

4. verificar a instalação do MySQL
$ sudo service mysql status

5. verificar o status de rede do MySQL
$ sudo ss -tap | grep mysql

6. Para configurar o MySQL para escutar conexões de hosts de rede, no arquivo /etc/mysql/mysql.conf.d/mysqld.cnf, altere a diretiva de endereço de ligação para o endereço IP do servidor: bind-address  = 0.0.0.0

$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

Depois de fazer uma alteração na configuração, o daemon do MySQL precisará ser reiniciado com o seguinte comando:

$ sudo systemctl restart mysql.service

7. criar um usuário para acesso externo e defina seus privilégios

$ sudo mysql

mysql> use mysql;

mysql> select user, host from mysql.user;

mysql> CREATE USER 'db_user'@'%' IDENTIFIED BY '123456';

mysql> GRANT ALL PRIVILEGES on *.* TO 'db_user'@'%' WITH GRANT OPTION;

mysql> FLUSH PRIVILEGES;

mysql> exit

####Para Dev vc cria com menos privilégios####
GRANT  INSERT, UPDATE, DELETE, SELECT on <nome_do_objeto> TO 'dev'@'%' WITH GRANT OPTION;
########

8. testar usuário db_user

$sudo mysql -u db_user -p
