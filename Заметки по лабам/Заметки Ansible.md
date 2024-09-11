https://192.168.225.50/ui/
Ansible@vmtraining.local
Ansible!09_20$24!


sa / pass

https://www.perplexity.ai/


машина тренера:
DNS Name:	ws.lab.example.com
IP Addresses:	172.16.195.142

мой сервер управления
DNS Name:	ws.lab.example.com
IP Addresses:	172.16.195.155

serverA
DNS Name:	servera.lab.example.com
IP Addresses:	172.16.195.163




## Ansible день 1


Инвентори по умолчанию лежит
/etc/ansible/hosts



Настройка Ansible
/etc/ansible/ansible.cfg`-низкий приоритет
~/.ansible.cfg - средний приоритет
./ansible.cfg - высокий приоритет





Плагины для VSCode для удаленного подключения:
Remote -ssh
Remote Explorer
Remote tunels

список управляемых хостов
ansible all -i inventory.cfg --list-hosts

пинг всех хостов
ansible all -m ping

Выполнить неидемпотентую команду(BAD)
ansible all -m command -a /usr/bin/hostname -o

Идемпотентные коанды через модули -m :
ansible all -m command -a 'id'
ansible all -m copy  -a 'content="This server is managed by Ansible.\n" dest=/etc/motd' --become
ansible all -m command -a 'cat /etc/motd'
ansible -m user -a 'name=newbie uid=4000 state=present' -b


## Ansible день 2

Проверка синтаксиса плейбука
ansible-playbook --syntax-check webserver.yml

Проверка выполнения(сухой прогон)
ansible-playbook -C webserver.yml


Просмотр документации по модулям
ansible-doc -l #список модулей
ansible-doc user #инфа по конкретному модулю
ansible-doc -s yum #инфа по конкретному модулю сокращенно

Расположение модулей bultin
/usr/lib/python3.9/site-packages/ansible/modules/


### Модуль 4 Переменные

!!нельзя называть пременные как имя модуля

приоритет перменных:
https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#where-to-set-variables


Если переменная указана в начале ключа, тоее нужно помещать в кавычки

- name: Creates the user {{ user }}
  user:  
    name: "{{ users }}"

Переменные в плейбук можно определеить прямо или через файл:

- hosts: all
  vars:
    users: joe

- hosts: all
  vars_files:
    - vars/users.yml

### Вместо переменных в инвентори использовать каталоги group_vars и host_vars

в group_vars создается файл по имени группы
в host_vars создается файл с именем хоста

принудителный сбор фактов
- name: get manual facts
  setup:

для записи своих ansible_facts на хосте ложим в папку с расширением .fact в формате ini
/etc/ansible/facts.d/users.fact

обращзаться к ней можно так:

-name: ansible facts local
  debag:
    var: ansible facts.ansible_local.users

можно выключить сбор фактов
gather_facts: no


Магические переменные
Самые полезные из них:
hostvars – содержит переменные для управляемых хостов и может быть использован для получения значений переменных другого переменных другого управляемого узла. Не включает факты управляемого узла, если они еще не были собраны для этого узла;
group_names – перечисляет все группы, в которых находится текущий управляемый хост;
groups – перечисляет все группы и хосты в инвентаре;
inventory_hostname – содержит имя хоста для текущего управляемого хоста, настроенного в инвентаризации. Может отличаться от имени хоста, сообщаемого фактами, по различным причинам.


## Ansible день 3

### Циклы

  vars:
    web_service: httpd
    fw_service: firewalld
    services:
     - "{{ web_service }}"
     - "{{ fw_service }}"
  #Enable and start services
  - name: Ensure services are started and enabled
      service:
        name: "{{ item }}"
          state: started
          enabled: yes
        loop: "{{ services }}"
### when

Вот таблица в формате Markdown, созданная на основе предоставленного текста:

#### Условные обозначения

| Условие                               | Пример                          |
|---------------------------------------|---------------------------------|
| Equal (value is a string)            | `ansible_machine == "x86_64"`  |
| Equal (value is numeric)             | `max_memory == 512`            |
| Less than                             | `min_memory < 128`             |
| Greater than                          | `min_memory > 256`             |
| Less than or equal to                | `min_memory <= 256`            |
| Greater than or equal to             | `min_memory >= 512`            |
| Not equal to                          | `min_memory != 512`            |
| Variable exists                       | `min_memory is defined`        |
| Variable does not exist               | `min_memory is not defined`|


#### Операции

| Операция                                                                 | Пример                          |
|--------------------------------------------------------------------------|---------------------------------|
| Булева переменная - это истина (true). Значения 1, True или yes оцениваются как true. | `memory_available`              |
| Булева переменная равна false. Значения 0, False или no оцениваются как false. | `not memory_available`          |
| Значение первой переменной присутствует как значение во второй список переменных | `ansible_distribution in supported_distros` |

Вы можете использовать этот Markdown-код в любом редакторе, поддерживающем форматирование Markdown, чтобы отобразить таблицы.


#### Синтаксис сложного условия
```yaml
when: >
  - ( ansible_distribution == "RedHat" and
      ansible_distribution_major_version == "7" )
    or
    ( ansible_distribution == "Fedora" and
      ansible_distribution_major_version == "28" )
```

> **Информация:** Переменная {{ ansible_mounts }} в Ansible содержит информацию о всех смонтированных файловых системах на целевой машине.

> **Информация:** параметр игнорирования ошибок в таске
ignore_errors: yes


#### handlers

* хендлеры при упоминании несколько раз выполняются тольок 1 раз
* Хендлеры записываются в конце плейбука
* хенджлеры выполняются только если статус таска changed
* Хендлеры выполняются по порядку записи в плейбуке

синтаксис:

```yaml
  tasks:
    - name: Copy Config Files
        copy:
          src: "{{ item.src }}"
          dest: "{{ item.dest }}"
        loop: "{{ web_config_files }}"
        notify: restart web service
#Add handlers
  handlers:
    - name: Restart web service
      service:
        name: "{{ web_service }}"
        state: restarted
```
> **инф** force_handlers: yes - парметр на уровне плея, хендлеры выполнятся даже если плей завершился fail , после последнего неудавшегося таска плей переходит к хендлерам 

#### обраотка отказов
через 
failed_when: "'Erorr' in comand_result.stdout"

или через модуль fail, он завершается всегда ошибкой и выходом:

tasks:
    - name: Проверка наличия обязательных переменных
      fail:
        msg: "Переменные Directory и SearchString обязательны для выполнения плейбука."
      when: Directory is not defined or SearchString is not defined

можно поменять статус таски на changed через команду
changed_when : false
или
changed_when: "'Success' in comand_result.stdout"

#### пример использования block: это что то типа if else
```yaml
---
- name: Пример использования block, rescue и always
  hosts: all
  become: yes
  tasks:
    - name: Основной блок задач
      block:
        - name: Установка пакета
          yum:
            name: httpd
            state: present

        - name: Запуск службы httpd
          systemd:
            name: httpd
            state: started

    - name: Обработка ошибок
      rescue:
        - name: Вывод сообщения об ошибке
          debug:
            msg: "Произошла ошибка при установке или запуске httpd."

        - name: Завершение работы службы httpd
          systemd:
            name: httpd
            state: stopped

    - name: Задачи, которые всегда выполняются
      always:
        - name: Сообщение о завершении выполнения
          debug:
            msg: "Завершение выполнения плейбука, независимо от успеха или неудачи."
```
