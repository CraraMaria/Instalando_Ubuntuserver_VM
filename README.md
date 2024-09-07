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

############### PARTE 2  ##############################

1. No Virtualbox redirecione a porta 3306 do hospedeiro para 3306 da máquina convidada.(agora vc pode conectar via putty)

2. No cliente é necessário definir a variável para aceitar o certificado digital

allowPublicKeyRetrieval=true

    For DBeaver users:

    Right-click your connection, choose "Edit Connection"

    On the "Connection settings" screen (main screen), click on "Edit Driver Settings"

    Click on "Driver properties"

    Change two properties: "useSSL" and "allowPublicKeyRetrieval"

    Set their values to "false" and "true" by double-clicking on the "value" column

3. Conectar com o cliente DBeaver

4. Crie seu primeiro banco de dados
CREATE DATABASE HelloMysql;

5. Adeus primeiro BD
DROP DATABASE HelloMysql;


####Reiniciar o serviço do MySQL####
$ sudo service mysql restart

####troubleshooting####
$ sudo journalctl -u mysql
#Crie o banco de dados
mysql> CREATE DATABASE university;


#Selecione o banco de dados a ser manipulado ou definido
mysql> use university;


#Execute o seguinte script de criação do banco de dados
create table classroom
    (building        varchar(15),
     room_number        varchar(7),
     capacity        numeric(4,0),
     primary key (building, room_number)
    );

create table department
    (dept_name        varchar(20),
     building        varchar(15),
     budget                numeric(12,2) check (budget > 0),
     primary key (dept_name)
    );

create table course
    (course_id        varchar(8),
     title            varchar(50),
     dept_name        varchar(20),
     credits        numeric(2,0) check (credits > 0),
     primary key (course_id),
     foreign key (dept_name) references department (dept_name)
        on delete set null
    );

create table instructor
    (ID            varchar(5),
     name            varchar(20) not null,
     dept_name        varchar(20),
     salary            numeric(8,2) check (salary > 29000),
     primary key (ID),
     foreign key (dept_name) references department (dept_name)
        on delete set null
    );

create table section
    (course_id        varchar(8),
         sec_id            varchar(8),
     semester        varchar(6)
        check (semester in ('Fall', 'Winter', 'Spring', 'Summer')),
     year            numeric(4,0) check (year > 1701 and year < 2100),
     building        varchar(15),
     room_number        varchar(7),
     time_slot_id        varchar(4),
     primary key (course_id, sec_id, semester, year),
     foreign key (course_id) references course (course_id)
        on delete cascade,
     foreign key (building, room_number) references classroom (building, room_number)
        on delete set null
    );

create table teaches
    (ID            varchar(5),
     course_id        varchar(8),
     sec_id            varchar(8),
     semester        varchar(6),
     year            numeric(4,0),
     primary key (ID, course_id, sec_id, semester, year),
     foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        on delete cascade,
     foreign key (ID) references instructor (ID)
        on delete cascade
    );

create table student
    (ID            varchar(5),
     name            varchar(20) not null,
     dept_name        varchar(20),
     tot_cred        numeric(3,0) check (tot_cred >= 0),
     primary key (ID),
     foreign key (dept_name) references department (dept_name)
        on delete set null
    );

create table takes
    (ID            varchar(5),
     course_id        varchar(8),
     sec_id            varchar(8),
     semester        varchar(6),
     year            numeric(4,0),
     grade                varchar(2),
     primary key (ID, course_id, sec_id, semester, year),
     foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        on delete cascade,
     foreign key (ID) references student (ID)
        on delete cascade
    );

create table advisor
    (s_ID            varchar(5),
     i_ID            varchar(5),
     primary key (s_ID),
     foreign key (i_ID) references instructor (ID)
        on delete set null,
     foreign key (s_ID) references student (ID)
        on delete cascade
    );

create table time_slot
    (time_slot_id        varchar(4),
     day            varchar(1),
     start_hr        numeric(2) check (start_hr >= 0 and start_hr < 24),
     start_min        numeric(2) check (start_min >= 0 and start_min < 60),
     end_hr            numeric(2) check (end_hr >= 0 and end_hr < 24),
     end_min        numeric(2) check (end_min >= 0 and end_min < 60),
     primary key (time_slot_id, day, start_hr, start_min)
    );

create table prereq
    (course_id        varchar(8),
     prereq_id        varchar(8),
     primary key (course_id, prereq_id),
     foreign key (course_id) references course (course_id)
        on delete cascade,
     foreign key (prereq_id) references course (course_id)
    );

#vamos trabalhar algumas deleções
drop table student;
drop table teaches;
drop table section;
drop table instructor;
drop table course;
drop table department;
drop table classroom;

ALTER TABLE course ADD description VARCHAR(100);
ALTER TABLE course ALTER COLUMN credits NUMERIC(3,1);
ALTER TABLE course RENAME COLUMN title TO course_title;
ALTER TABLE course DROP COLUMN description;

*Execute o arquivo smallRelationsInsertFile.sql*/

/*Cria uma trigger verificando se o estudante é calouro*/
CREATE TRIGGER trigger_check_student_freshman
BEFORE INSERT ON takes
FOR EACH ROW
BEGIN
    DECLARE student_count INT;
    DECLARE error_message VARCHAR(255);
   
    SELECT COUNT(*) INTO student_count
    FROM takes
    WHERE ID = NEW.ID;
   
    IF student_count = 0 THEN
        SET error_message = CONCAT('O aluno com ID ', NEW.ID, ' é um calouro');
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = error_message;
    END IF;
END;


SELECT COUNT(*)
    FROM takes
    WHERE ID = '12346';

SELECT ID, course_id, sec_id, semester, `year`, grade
FROM university.takes where ID = '12346';

INSERT INTO student VALUE('12346', 'Ana', 'Comp. Sci.', 0);

SELECT course_id, sec_id, semester, `year`, building, room_number, time_slot_id
FROM university.`section` WHERE course_id = 'CS-101';

INSERT INTO university.`section`
(course_id, sec_id, semester, `year`, building, room_number, time_slot_id)
VALUES('CS-101', '3', 'Summer', 2024, 'Packard', '101', 'F');

SELECT ID, course_id, sec_id, semester, `year`, grade
FROM university.takes where ID = '12346';

INSERT INTO takes
(ID, course_id, sec_id, semester, `year`, grade)
VALUES('12346', '584', '3', 'Summer', 2024, 'B ');


CREATE VIEW student_takes_view AS
SELECT
    s.ID AS student_id,
    s.name AS student_name,
    s.dept_name AS student_department,
    s.tot_cred AS student_total_credits,
    t.course_id,
    t.sec_id,
    t.semester,
    t.year,
    t.grade
FROM
    student s
JOIN
    takes t ON s.ID = t.ID;
   
SELECT * FROM student_takes_view;

CREATE INDEX idx_student_id ON student (ID);
CREATE INDEX idx_takes_id ON takes (ID);

SHOW INDEX FROM student ;

EXPLAIN SELECT * FROM student_takes_view;
