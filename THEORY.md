# Ansible Lint: проверка playbooks

В прошлом блоке вы разобрались с хранением секретных данных. Теперь можно использовать Ansible, не боясь
скомпрометировать секретные данные продуктовой среды.

Однако, прежде, чем запускать плейбук в продакшене, хорошей практикой является проверка кода статическим анализатором (
`линтером`) на возможность его улучшения.

Одним из самых популярных линтеров для Ansible является `Ansible-Lint`, созданный Уиллом Темзом.

Ansible-lint (далее «линт») использует каталог с правилами, реализованными в виде сценариев на Python. При желании вы
можете создать дополнительный каталог со своими правилами.

В этом блоке мы:

- разберёмся, как установить линт;
- улучшим с помощью линта **написанную ранее роль;
- поговорим об интеграции линта с CI инструментом;
- узнаем, как игнорировать замечания линта.

## Установка ansible-lint

Рекомендуемый способ установки `ansible-lint` с помощью пакетного менеджера Python, потому что он в большинстве случаев
содержит более свежие версии пакетов, является кросс-платформенным, содержит широкий выбор пакетов и пр.:

- Установка с помощью pip — pip install `ansible-lint`.

Однако вы можете установить его через пакетный менеджер соответствующего дистрибутива:

Ubuntu/Debian — `apt update && apt install ansible-lint`;
Fedora/RedHat/CentOS — `dnf install ansible-lint`.
Проверить версию линта можно с помощью команды:

```bash
ansible-lint --version
```

output:

```text
ansible-lint 24.5.0 using ansible-core:2.16.6 ansible-compat:24.6.0
ruamel-yaml:0.18.6 ruamel-yaml-clib:0.2.8
```

## Проверка линтом написанных ролей и плейбуков

Давайте вернёмся к приведённому ранее плейбуку, который использует роль установки веб-сервера на базе nginx на ОС
семейства CentOS/RHEL. И попробуем его «скормить» линту.

```bash
ansible-lint install_nginx.yml
```

output:

```text
WARNING  Listing 21 violation(s) that are fatal
name[play]: All plays should be named.
install_nginx.yml:2

yaml[truthy]: Truthy value should be one of [false, true]
install_nginx.yml:3

yaml[trailing-spaces]: Trailing spaces
roles/nginx/tasks/main.yml:4

fqcn[action-core]: Use FQCN for builtin module actions (yum_repository).
roles/nginx/tasks/main.yml:6 Use `ansible.builtin.yum_repository` or `ansible.legacy.yum_repository` instead.

yaml[indentation]: Wrong indentation: expected at least 3
roles/nginx/tasks/main.yml:6

yaml[truthy]: Truthy value should be one of [false, true]
roles/nginx/tasks/main.yml:7

yaml[trailing-spaces]: Trailing spaces
roles/nginx/tasks/main.yml:12

fqcn[action-core]: Use FQCN for builtin module actions (yum).
roles/nginx/tasks/main.yml:15 Use `ansible.builtin.yum` or `ansible.legacy.yum` instead.

package-latest: Package installs should not use latest.
roles/nginx/tasks/main.yml:15 Task/Handler: Install Nginx on CentOS/RHEL latest version

yaml[truthy]: Truthy value should be one of [false, true]
roles/nginx/tasks/main.yml:16

fqcn[action-core]: Use FQCN for builtin module actions (yum).
roles/nginx/tasks/main.yml:22 Use `ansible.builtin.yum` or `ansible.legacy.yum` instead.

yaml[truthy]: Truthy value should be one of [false, true]
roles/nginx/tasks/main.yml:23

fqcn[action-core]: Use FQCN for builtin module actions (service).
roles/nginx/tasks/main.yml:30 Use `ansible.builtin.service` or `ansible.legacy.service` instead.

yaml[truthy]: Truthy value should be one of [false, true]
roles/nginx/tasks/main.yml:31

yaml[truthy]: Truthy value should be one of [false, true]
roles/nginx/tasks/main.yml:35

fqcn[action-core]: Use FQCN for builtin module actions (setup).
roles/nginx/tasks/main.yml:39 Use `ansible.builtin.setup` or `ansible.legacy.setup` instead.

yaml[indentation]: Wrong indentation: expected 4 but found 2
roles/nginx/tasks/main.yml:39

fqcn[action-core]: Use FQCN for builtin module actions (set_fact).
roles/nginx/tasks/main.yml:47 Use `ansible.builtin.set_fact` or `ansible.legacy.set_fact` instead.

fqcn[action-core]: Use FQCN for builtin module actions (template).
roles/nginx/tasks/main.yml:50 Use `ansible.builtin.template` or `ansible.legacy.template` instead.

risky-file-permissions: File permissions unset or incorrect.
roles/nginx/tasks/main.yml:50 Task/Handler: Copy the new welcome web page

yaml[truthy]: Truthy value should be one of [false, true]
roles/nginx/tasks/main.yml:51

Read documentation for instructions on how to ignore specific rule violations.

                    Rule Violation Summary
 count tag                    profile    rule associated tags
     1 name[play]             basic      idiom
     2 yaml[indentation]      basic      formatting, yaml
     2 yaml[trailing-spaces]  basic      formatting, yaml
     7 yaml[truthy]           basic      formatting, yaml
     1 package-latest         safety     idempotency
     1 risky-file-permissions safety     unpredictability
     7 fqcn[action-core]      production formatting

Failed: 21 failure(s), 0 warning(s) on 3 files. Last profile that met the validation criteria was 'min'.
```

Ого! Кажется, наша роль далека от совершенства. Но не стоит пугаться этого: со временем у вас появится навык написания
более чистых плейбуков.

Кроме того, если вы будете следовать рекомендациям хорошо оформленного кода, вашим коллегам будет проще разобраться в
ваших ролях и плейбуках.

Как мы уже знаем, сила Ansible заключается в большом количестве готовых ролей. И хорошая новость в том, что большинство
опубликованных ролей учитывают рекомендации линта при загрузке их в репозиторий.

## Приведение роли к рекомендованному виду

Давайте разберём на категории замечания линта и попытаемся их исправить.

### Плей в плейбуке: именование

Первое замечание линта, которое мы встретили, выглядело так:

```text
name[play]: All plays should be named.
install_nginx.yml:2
```

Вспоминаем, что каждый плейбук направлен на выполнение какой-то цели. Например, установка приложения. Но часто бывает
так, что для приложения нужны дополнительные компоненты, например, база данных. То есть наш плейбук должен для одних
типов хостов установить приложение, а для других — БД. Эти логические части называются плей. И хорошим тоном является
дать каждому плею имя, даже если он единственный в плейбуке. В нашем случае линт обнаружил два замечания данного типа (
install_nginx.yml:2).

Далее, для наглядности, мы будем приводить разницу до правки и после исправления:

```bash
git diff install_nginx.yml
diff --git a/install_nginx.yml b/install_nginx.yml
```

output:

```text
index 75f4d52..ea39b4a 100644
--- a/install_nginx.yml
+++ b/install_nginx.yml
@@ -1,5 +1,6 @@
 ---
-- hosts: web_servers
+- name: Install web-server
+  hosts: web_servers
   gather_facts: no
   roles:
     - role: nginx
```

После изменений замечание линта пропало: 1 name[play].

### Унифицированные значения для логических переменных

Ансибл своей гибкостью иногда усложняет жизнь разработчиков. И «фича» превращается в «баг», который очень сложно в
будущем найти. Одной из этих «фич» является то, что ансибл может в качестве логического аргумента правды принимать
`true,1,yes`, а в качестве лжи принимать `false,0,no`. Лучшей практикой является использование только `true` и `false`.

Давайте заменим значения 7 yaml[truthy] на рекомендованные с помощью следующих команд:

```bash
find . -type f | xargs sed -i 's/: no/: false/g'
find . -type f | xargs sed -i 's/: yes/: true/g'
```

💡 С осторожностью применяйте такие команды, т.к. заменяемое слово может использоваться в другом контексте. Поэтому не
забывайте делать коммиты перед изменениями, пользоваться командой `git diff` перед следующим коммитом.
Опустим в этом блоке вывод гита для краткости.

Избегание пробелов в конце строки
Иногда лишние пробелы в конце строки (trailing-spaces) могут сыграть с вами злую шутку. Особенно это неприятно тем, что
они не видны человеческому взгляду. Поэтому в хорошем коде они должны отсутствовать:

```bash
git diff
diff --git a/roles/nginx/tasks/main.yml b/roles/nginx/tasks/main.yml
```

output:

```text
index 6f51ed4..2abc048 100644
--- a/roles/nginx/tasks/main.yml
+++ b/roles/nginx/tasks/main.yml
@@ -1,7 +1,7 @@
 ---
 # roles/nginx/tasks/main.yml
 - name: Nginx installation
-  block:
+  block:

   - name: Add Nginx repository on CentOS/RHEL
     become: true
@@ -9,7 +9,7 @@
       name: nginx
       description: Official Nginx repository
       baseurl: http://nginx.org/packages/centos/$releasever/$basearch/
-      gpgcheck: true
+      gpgcheck: true
       gpgkey: https://nginx.org/keys/nginx_signing.key
       enabled: true
   - name: Install Nginx on CentOS/RHEL latest version
```

После изменений замечания 2 yaml[trailing-spaces] пропали.

## Избегание сокращённого названия модулей

Как мы уже говорили раньше, хорошей практикой использовать полное название модуля в коллекции FQCN (Fully Qualified
Collection Name).

Например, вместо модуля `yum_repository` стоит указать `ansible.builtin.yum_repository`.

Давайте исправим замечания 7 fqcn[action-core]:

```bash
git diff
diff --git a/roles/nginx/tasks/main.yml b/roles/nginx/tasks/main.yml
```

output:

```text
index 2abc048..205af04 100644
--- a/roles/nginx/tasks/main.yml
+++ b/roles/nginx/tasks/main.yml
@@ -5,7 +5,7 @@

   - name: Add Nginx repository on CentOS/RHEL
     become: true
-    yum_repository:
+    ansible.builtin.yum_repository:
       name: nginx
       description: Official Nginx repository
       baseurl: http://nginx.org/packages/centos/$releasever/$basearch/
@@ -14,14 +14,14 @@
       enabled: true
   - name: Install Nginx on CentOS/RHEL latest version
     become: true
-    yum:
+    ansible.builtin.yum:
       name: nginx
       state: latest
       update_cache: true
     when: nginx_redhat_nginx_version == ""
   - name: Install Nginx on CentOS/RHEL sepsific version {{ nginx_redhat_nginx_version }}
     become: true
-    yum:
+    ansible.builtin.yum:
       name: "nginx-{{ nginx_redhat_nginx_version }}"
       state: present
       allow_downgrade: true # если мы захотим переустановить пакет меньшей версии
@@ -29,7 +29,7 @@
     when: nginx_redhat_nginx_version != ""
   - name: Start Nginx service
     become: true
-    service:
+    ansible.builtin.service:
       name: nginx
       state: started     # переводим в состояние запущено
       enabled: true      # добавляем в автозагрузку
@@ -37,7 +37,7 @@
 - name: Get ansible_facts and publish them to the default page
   block:
   - name: Сollecting only the necessary facts
-    setup:
+    ansible.builtin.setup:
       filter:
         - ansible_distribution
         - ansible_distribution_version
@@ -45,10 +45,10 @@
     ansible.builtin.package_facts:
       manager: auto
   - name: Add fact about nginx version
-    set_fact:
+    ansible.builtin.set_fact:
       nginx_installed_version: "{{ ansible_facts.packages.nginx[0].version }}"
   - name: Copy the new welcome web page
     become: true
-    template:
+    ansible.builtin.template:
       src: index.html.j2
       dest: "{{ nginx_redhat_welcome_page_path }}/index.html"   
```

## Унификация отступов

Ансибл использует YAML, а значит важно правильно выравнивать отступы для корректной интерпретации кода. Не забывайте
следовать следующим правилам:

Используйте пробелы, вместо табуляции;
Выровняйте отступы на одинаковом уровне для всех вложенных элементов;
Обычно отступы составляют 2 или 4 пробела.
💡 Чтобы избежать подобных ошибок, используйте текстовый редактор или IDE с поддержкой YAML и функциями проверки
синтаксиса. Например, VS Code
Давайте исправим замечания `2 yaml[indentation]`:

Было:

```yaml
- name: Nginx installation
  block:
    - name: Add Nginx repository on CentOS/RHEL
      become: true
      ansible.builtin.yum_repository:
        ...
- name: Get ansible_facts and publish them to the default page
  block:
- name: Сollecting only the necessary facts
  ansible.builtin.setup:
    filter:
      ...
```

Стало:

```yaml
- name: Nginx installation
  block:
    - name: Add Nginx repository on CentOS/RHEL
      become: true
      ansible.builtin.yum_repository:
        ...
    - name: Get ansible_facts and publish them to the default page
      block:
        - name: Сollecting only the necessary facts
          ansible.builtin.setup:
            filter:
              ...
```

### Задавать явно разрешения для файлов

В роли nginx мы копируем с помощью модуля template веб-страницу приветствия:

```yaml
  ansible.builtin.template:
    src: index.html.j2
    dest: "{{ nginx_redhat_welcome_page_path }}/index.html"
```

Линт «обратил внимание» на то, что у данного файла не выставлены разрешения. Как следствие, это может привести к
проблемам в безопасности. Давайте явно зададим резрешения:

```yaml
  ansible.builtin.template:
    src: index.html.j2
    dest: "{{ nginx_redhat_welcome_page_path }}/index.html"
    mode: "0644"
```

После изменений замечание `1 risky-file-permissions` устранено.

## Избегайте установки приложения с тегом Latest

Линт «обращает внимание» на попытку установить приложение с тегом `Latest`.

Идея очень простая: ваш плейбук может повести себя непредсказуемо при установке приложения вновь вышедшей версии.
Поэтому лучшей практикой является явно задавать версию устанавливаемого приложения.

Однако, если мы вспомним, то мы сделали такое поведение умышленным. И мы устанавливаем последнюю версию веб-сервера
только в том случае, когда в файле переменных не задана версия явно.

А значит, нам можно проигнорировать данное замечание на данном этапе.

## Интеграция Ansible-Lint c CI

Перед запуском стадии (stage) развёртывания (deploy) вашего плейбука обычно делают предшествующую стадию с запуском
линта.

На примере GitLab CI приведён фрагмент файла `.gitlab-ci.yml`:

```yaml
...
stages:
  - lint
  - deploy

lint:
  stage: lint
  image: python:3.9
  before_script:
    - pip install ansible-lint
  script:
    - ansible-lint install_nginx.yml
...
```

Обратите внимание на то, что если мы запустим наш исправленный плейбук, то наш пайплайн завершится с ошибкой на стадии
`lint`. Дело в том, что мы проигнорировали одно из замечаний линта. А значит, команда `ansible-lint install_nginx.yml`
завершится отличным от нулевого кода возврата (0 код возврата означает успешное выполнение скрипта), что укажет
gitlab-ci инструменту, что стадия завершилась с ошибкой.

На этот случай в линте предусмотрены правила игнорирования ошибок.

## Использование правил игнорирования линта

### С помощью комментариев в плейбуке

Вы можете игнорировать ошибки непосредственно в плейбуке, используя комментарий:
`# noqa <tag>.`
Для нашего примера изменение в плейбуке:

```bash
git diff
diff --git a/roles/nginx/tasks/main.yml b/roles/nginx/tasks/main.yml
```

output:

```text
index 005699c..5352d8a 100644
--- a/roles/nginx/tasks/main.yml
+++ b/roles/nginx/tasks/main.yml
@@ -13,6 +13,7 @@
         enabled: true
     - name: Install Nginx on CentOS/RHEL latest version
       become: true
+      # noqa package-latest
       ansible.builtin.yum:
         name: nginx
         state: latest
```

Теперь при запуске линта мы не видим ошибок, и код оценён как готовность к продакшену:

```bash
ansible-lint install_nginx.yml
```

output:

```text
Passed: 0 failure(s), 0 warning(s) on 3 files. Last profile that met the validation criteria was 'production'.
```

### С помощью файла конфигурации

Если вы хотите игнорировать все ошибки определённого типа, то можно создать файл `.ansible-lint` в корне вашего проекта
такого вида:

```yaml
ignore:
  - package-latest
```

В нашем случае мы выбрали способ игнорирования с помощью комментариев.

## Итоги

В этом блоке вы:

- познакомились с инструментом Ansible-Lint;
- оптимизировали ранее написанную роль;
- посмотрели на примере инструмента GitlabCI, как можно интегрировать линт c CI-процессом и даже игнорировать замечания
  линта.
  В практическом задании вам предстоит оптимизировать кросс-платформенную роль с помощью линта.

В следующем блоке мы поговорим подробнее о тестировании плейбуков и ролей.