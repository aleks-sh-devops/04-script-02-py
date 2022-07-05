# 04-script-02-py

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:

Какое значение будет присвоено переменной `c`?  
```
TypeError: unsupported operand type(s) for +: 'int' and 'str' ошибка, поскольку тип данных строка складывается с типом данных integer
```

Как получить для переменной `c` значение 12?  
```
#!/usr/bin/env python3
a = '1'
b = '2'
c = a + b
print (c)
```

Как получить для переменной `c` значение 3?  
```
#!/usr/bin/env python3
a = 1
b = 2
c = a + b
print (c)
```

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
#print (result_os)
#print (os.popen ('pwd'))
#is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
#        print ('1: ',prepare_result)
        path_modified_files = os.path.abspath(prepare_result)
#        print('modif: ', path_modified_files)
        print('modif: ', path_modified_files)
#        break
```

### Вывод скрипта при запуске при тестировании:
```
vagrant@vagrant:~/netology/sysadm-homeworks$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   FOLD/456.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   file.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        789.txt

vagrant@vagrant:~/netology/sysadm-homeworks$  ./1.py
modif:  /home/vagrant/netology/sysadm-homeworks/file.txt

```
Обращаю внимание, что отображаются только модифицированные файлы и скрипт должен лежать в папке репы!  

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```
#!/usr/bin/env python3

import os

path_to_git_rep = input("Enter path: ")

os.chdir(path_to_git_rep)
bash_command = ["git status"]
result_os = os.popen(' && '.join(bash_command)).read()
#print (result_os)
#is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print('modif:', path_to_git_rep+prepare_result)
#        break
```

### Вывод скрипта при запуске при тестировании:
```
Enter path: /home/vagrant/netology/sysadm-homeworks
modif: /home/vagrant/netology/sysadm-homeworksfile.txt

Если несуществующи путь:
vagrant@vagrant:~$ ./2.py
Enter path: /home/vagrant/netology/1
Traceback (most recent call last):
  File "./2.py", line 7, in <module>
    os.chdir(path)
FileNotFoundError: [Errno 2] No such file or directory: '/home/vagrant/netology/1'

Если существует путь, но нет гита:
vagrant@vagrant:~$ ./2.py
Enter path: /home/vagrant/netology
fatal: not a git repository (or any of the parent directories): .git

```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```
#!/usr/bin/env python3

import time
import socket

dns_names = ['drive.google.com', 'mail.google.com', 'google.com', 'ya.ru']
old_ip = []

i = 0

while i < len(dns_names):
#    print (dns_names[i])
#    print (socket.gethostbyname(dns_names[i]))
    old_ip.append(socket.gethostbyname(dns_names[i]))
    i += 1
#print ('old_ip: ', old_ip)

while True:
    for name in dns_names:
        if socket.gethostbyname(name) == old_ip[dns_names.index(name)]:
            print ('DNS Name = ', name, 'IP Address = ', socket.gethostbyname(name))
        else:
            print ('[ERROR] ', name, 'IP mismatch! Old IP:', old_ip[dns_names.index(name)], 'New IP:', socket.gethostbyname(name))
            old_ip.pop(dns_names.index(name))
            old_ip.insert(dns_names.index(name), socket.gethostbyname(name))
        time.sleep(2)

```

### Вывод скрипта при запуске при тестировании:
```
DNS Name =  drive.google.com IP Address =  142.251.1.194
[ERROR]  mail.google.com IP mismatch! Old IP: 74.125.131.19 New IP: 74.125.131.18
[ERROR]  google.com IP mismatch! Old IP: 64.233.164.113 New IP: 64.233.164.100
DNS Name =  ya.ru IP Address =  87.250.250.242
DNS Name =  drive.google.com IP Address =  142.251.1.194
DNS Name =  mail.google.com IP Address =  74.125.131.18
DNS Name =  google.com IP Address =  64.233.164.100
DNS Name =  ya.ru IP Address =  87.250.250.242
DNS Name =  drive.google.com IP Address =  142.251.1.194
DNS Name =  mail.google.com IP Address =  74.125.131.18
DNS Name =  google.com IP Address =  64.233.164.100
DNS Name =  ya.ru IP Address =  87.250.250.242
```
