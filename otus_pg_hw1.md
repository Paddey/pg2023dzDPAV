## OTUS PostgreSQL Cloud Solutions

### Домашняя работа 1

**Cоздать новый проект**

![image](https://user-images.githubusercontent.com/36312830/235984473-8be06319-4ec8-4762-a403-30a8235d8178.png)

**Зайти удаленным ssh (первая сессия)**

**Инсталлировать PostgreSQL**

_sudo zypper in postgresql postgresql-server postgresql-contrib_

**Инсталлировать расширения**

_sudo zypper in postgresql-plperl postgresql-plpython postgresql-pltcl_

**Запустить Postgres**

_sudo systemctl enable postgresql_
_sudo systemctl start postgresql_

**Переключиться на пользователя postgres. Используем подключение через Unix-domain сокет**

_sudo su postgres_

**Зайти вторым ssh (вторая сессия) и запустить везде psql из под пользователя postgres и выключить auto commit**

![image](https://user-images.githubusercontent.com/36312830/235975643-d79bc5db-9444-46c8-a1af-070c1d3260f7.png)

**Сделать в первой сессии новую таблицу и наполнить ее данными**

**Посмотреть текущий уровень изоляции: show transaction isolation level**

![image](https://user-images.githubusercontent.com/36312830/235979216-30484c85-fd4f-400b-bf9d-5ce952e1a8e6.png)

**Начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции и в первой сессии добавить новую запись**

_insert into persons(first_name, second_name) values('sergey', 'sergeev');_

![image](https://user-images.githubusercontent.com/36312830/235984837-69dbe52c-e2f6-47ba-8e20-9043fa9e23f8.png)

**сделать select * from persons во второй сессии. Видите ли вы новую запись и если да то почему?**

Новая запись не видна тк изменение первой транзакции не зафиксировано.

![image](https://user-images.githubusercontent.com/36312830/235984997-f1941691-23b7-4fec-aec5-2ce37e47b207.png)

**Завершить первую транзакцию - commit;**
**Сделать select * from persons во второй сессии. Видите ли вы новую запись и если да то почему?**

Новая запись видна т.к в режиме Read Committed операторы одной транзакции видят зафиксированные изменения других транзакций при повторном вызове оператора.

![image](https://user-images.githubusercontent.com/36312830/235985112-ff3aaf20-3849-49be-9ca1-86e9b32ea0f7.png)

**Завершить транзакцию во второй сессии**

![image](https://user-images.githubusercontent.com/36312830/235985526-f74c6b27-22a7-4987-8334-b22ba9061021.png)

**Начать новые но уже repeatable read транзакции**
**В первой сессии добавить новую запись**

_insert into persons(first_name, second_name) values('sveta', 'svetova');_

![image](https://user-images.githubusercontent.com/36312830/235985722-37c7fcde-ceec-4239-b9b6-3d13eff04bce.png)

**Сделать select * from persons во второй сессии. Видите ли вы новую запись и если да то почему?** 

Результат работы первой транзакции недоступен т.к первая транзакция ещё не завершилась.

![image](https://user-images.githubusercontent.com/36312830/235986920-d5de624b-8d33-47f3-8760-7a8d81dfc0b1.png)

**Завершить первую транзакцию - commit;**

**Сделать select * from persons во второй сессии Видите ли вы новую запись и если да то почему?**

Результат первой транзакции во второй сессии не виден тк транзакция во *второй* сессии не завершилась.
Запрос в транзакции данного уровня видит снимок данных на момент начала первого оператора в транзакции (не считая команд управления транзакциями), 
а не начала текущего оператора. Таким образом, последовательные команды SELECT в одной транзакции видят одни и те же данные; 
они не видят изменений, внесённых и зафиксированных другими транзакциями после начала их текущей транзакции.

![image](https://user-images.githubusercontent.com/36312830/235987000-98e953d6-91ae-4139-b768-201d5b8ad3f9.png)

**Завершить вторую транзакцию**
**Сделать select * from persons во второй сессии видите ли вы новую запись и если да то почему?**  

Результат первой транзакции во второй сессии виден
Запрос в транзакции данного уровня видит снимок данных на момент начала первого оператора в транзакции (не считая команд управления транзакциями), 
а не начала текущего оператора. Таким образом, последовательные команды SELECT в одной транзакции видят одни и те же данные; 
они не видят изменений, внесённых и зафиксированных другими транзакциями после начала их текущей транзакции.

![image](https://user-images.githubusercontent.com/36312830/235987112-1de8592d-112e-4964-b4da-4ca54583d683.png)

