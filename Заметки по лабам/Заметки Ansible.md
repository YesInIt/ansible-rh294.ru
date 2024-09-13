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


### Ansible-vault

ansible-playbook --syntax-check --ask-vault-pass playbook.yml


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


### Использование regex
это в условиях
when: item is regex('^SSLFile.*')

это фактах:
- name: Извлечение строк, соответствующих регулярному выражению
      set_fact:
        matched_lines: "{{ file_content.stdout_lines | regex_search('^SSLFile.*') }}
## Файлы и темплейты
при обработке конфигов, грепаем строки без # и вносим переменные для .j2


отображается до входа в систему
  src: /etc/issue
        dest: /etc/issue.net
Отображает текст, после ввода логина
 path: /etc/motd

может выполнить скрипт при входе пользователя
/etc/bashrc
~/.bashrc

скелет для новых пользователей хранится в 
ls -lah /etc/skel/


## Ansible день 4
### Модуль 7

Шаблоны хостов - это использование подстановочных знаков в плеях в разделе 

---
- name: шаблооны хостов
  hosts: webservers,$msk,!serverb

  найди все сервера в группе wbservers И входящие в группу msk кроме сервера serverb


  >**inf** в ini при создании вложений групп при перекрестных ссылках на разные инвентори файлы , в каждом инвентори должна быть хотя бы пустые группы


[qa]

[test1]
test1.ru
test2.ru

[test2]

[test:children]
test1
test2
qa

### Динамический инвентори

например можно брать плагина
https://github.com/ansible/ansible/tree/release1.5.0/plugins

посмотреть дерево инвентори:
ansible-inventory -i ./lab3/inventory/inventory --graph

посмотреть json инвентори:
ansible-inventory -i ./lab3/inventory/inventory --list


### Форки и сериал
одновременный запуск 
ansible-config dump |grep -i forks

править в конфиге
[student@demo ~]$ grep forks ansible.cfg forks = 5

serial жостко указывает сколько должно быть выполнено хостов успешно в плее. Если неуспешно хотя бы один, плей останавливается

весь плей выполняется только на 2 хоста, если успешно, то следующие 2 хоста 
```yaml
---
- name: шаблооны хостов
  hosts: webservers
  serial:2
```

форки - это вилки
сериал - это серии

### Импорт плейбуков и тасков
уровень плеев

```yaml
-name: import plybook
 import_playbook: web.yml
```

уровень тасков, импорт тасков/ в yml таски прописаны подряд с нулевого уровня. файл только с тасками

```yaml
-name: import tasks
 hosts: webservers
 tasks:
 - import_tasks: webserver_tasks.yml
```
импорт полностью формирует плейбук и после выполняет его. этостаическое созание плейбука.


Инклюд подключает таски только в момент выполнения  инклуд таска,
в начале работы плейбука про инклуд таски нкто незнает.

```yaml
-name: include tasks
 hosts: webservers
 tasks:
 - include_tasks: webserver_tasks.yml
```

Re-using files and roles
Ansible offers two ways to reuse files and roles in a playbook: dynamic and static.

For dynamic reuse, add an include_* task in the tasks section of a play:

include_role

include_tasks

include_vars

For static reuse, add an import_* task in the tasks section of a play:

import_role

import_tasks


### Модуль 8 Упрощение наборов сценариев с помощью ролей и коллекций.

```yaml
---
- name: Example Playbook
  hosts: all
  gather_facts: yes

  pre_tasks:
    - name: Check for required packages
      package:
        name:
          - git
          - curl
        state: present
  roles:
    - role: common
    - role: webserver
    - role: database
  tasks:
    - name: Clone repository
      git:
        repo: 'https://github.com/example/repo.git'
        dest: '/opt/repo'
      notify: Restart application

    - name: Install dependencies
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - python3
        - python3-pip

    - name: Run application
      command: python3 /opt/repo/app.py
      async: 10
      poll: 0

  post_tasks:
    - name: Clean up temporary files
      file:
        path: /tmp/some_temp_file
        state: absent
  handlers:
    - name: Restart application
      service:
        name: my_application
        state: restarted
```
можно инклудить роль внутрь таска
```yaml
---
- name: Install and configure Nginx
  hosts: webservers
  become: yes

  tasks:
    - name: Include Nginx role
      include_role:
        name: nginx
```


вывести список ролей
ansible-galaxy role list


включить запись логов можно в ansible.cfg
log_path = /var/log/ansible.log

сделать права на это файл заранее

ложить роли можно в 
Ansible ищет роли в следующих местах:
В каталоге roles/: Относительно файла плейбука.
В /etc/ansible/roles: Это стандартный путь для хранения ролей.
В пользовательском каталоге: Обычно это ~/.ansible/roles.
В каталогах, указанных в конфигурации: Вы можете настроить дополнительные пути для поиска ролей в файле ansible.cfg с помощью параметра roles_path.
Полный путь к роли: Вы можете указать полный путь к роли при вызове в плейбуке.

Если я использую зависимости роли из ансибле гелекси, то роль подтянется сама при запуске моей роли, если в ./meta/main.yml
прописан параметр
```yaml
---
dependencies:

```


установка роли
ansible-galaxy install geerlingguy.apache -p roles/

Установка ролей из requirements.yml
Чтобы установить роли, указанные в requirements.yml, выполните следующую команду:
bash
ansible-galaxy install -r requirements.yml

Эта команда установит все роли, перечисленные в файле requirements.yml, в стандартный каталог ролей (обычно /etc/ansible/roles или ~/.ansible/roles).

```yaml
---
roles:
  - src: geerlingguy.apache
    version: 1.0.0
  - src: geerlingguy.mysql
    version: 3.0.0
  - src: git+https://github.com/username/my_custom_role.git

```
установка роли в папку
ansible-galaxy install -r roles/requirements.yml -p roles

устновка коллекции 
```yaml
#install a collection hosted in Galaxy
ansible-galaxy collection install my_namespace.my_collection
#upgrade a collection to the latest available version
ansible-galaxy collection install my_namespace.my_collection –upgrade
#directly use the tarball from your build
ansible-galaxy collection install my_namespace-my_collection-1.0.0.tar.gz \ -p ./collections
#build and install a collection from a local source directory
ansible-galaxy collection install /path/to/collection -p ./collections

#удаление колекции
rm -rf ~/.ansible/collections/ansible_collections/community/general
```

список колекций
ansible-galaxy collection list


/usr/lib/python3.9/site-packages/ansible_collections

```yaml
- name: Reference collections contents using their FQCNs
hosts: all tasks:
- name: Import a role ansible.builtin.import_role:
name: my_namespace.my_collection.role1
- name: Call a module
my_namespace.mycollection.my_module: option1: value
- name: Call a debug task
ansible.builtin.debug:
msg: '{{ lookup("my_namespace.my_collection.lookup1", 'param1')| my_namespace.my_collection.filter1 }}'
```

зависимости от коллекции можно указать внутри роли
```yaml
# myrole/meta/main.yml
collections:
- my_namespace.first_collection
- my_namespace.second_collection
- other_namespace.other_collection
```

создание скелета роли
ansible-galaxy init <имя роли>



## Ansible день 5 Устранение проблем в Ansible.

Логи писать в /var/log
Ротация логов
 /etc/logrotate.d/
создается отдельный файлик, ожно копировать с имеющихся или дописывать в существующие правила

лог ротейт прописан в crontab
/etc/chrontab

в модуле дебаг можно менять режим verbisity
```yaml
- name: Example of debug with verbosity
  debug:
    var: my_variable
    verbosity: 2
```



Проверить синтакс
[student@demo ~]$ ansible-playbook play.yml --syntax-check
Подверждения каждого шага
[student@demo ~]$ ansible-playbook play.yml --step

[student@demo ~]$ ansible-playbook play.yml --start-at-task="start httpd service"


список тасков с тегами
ansible-playbook example_playbook.yml --list-tasks
запустить все таски с тегом web
ansible-playbook example_playbook.yml --tags web
запустить все таски кроме cleanup
ansible-playbook example_playbook.yml --skip-tags cleanup

проверка плейбука
ansible-playbook --check example_playbook.yml
или
ansible-playbook -С example_playbook.yml

в тасках есть параметр :
check_mode: yes # таска всегда всгде только в режиме check
check_mode: no # таска всегда будет выполняться, даже в режиме check


