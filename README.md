## Домашнее задание к занятию "08.02 Работа с Playbook"

### Подготовка к выполнению

1. Создайте свой собственный (или используйте старый) публичный репозиторий на `github` с произвольным именем.
2. Скачайте `playbook` из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соотвтествии с группами из предподготовленного [playbook](https://github.com/netology-code/mnt-homeworks/tree/MNT-7/08-ansible-02-playbook/playbook).
4. Скачайте дистрибутив [java](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) и положите его в директорию `playbook/files/`.

### Основная часть

1. Приготовьте свой собственный inventory файл `prod.yml`.
2. Допишите `playbook`: нужно сделать ещё один play, который устанавливает и настраивает `kibana`.
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
7. Запустите playbook на prod.yml окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что `playbook` идемпотентен.
9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.


***Ответ:***

![альт](https://i.ibb.co/88vbBDM/Screenshot-1.jpg)

> Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.

Дописал task для установки `Kibana` по аналогии с `Elasticsearch` в ` playbook/site.yml`

```
- name: Install kibana
    hosts: kibana
    tasks:
      - name: Upload tar.gz kibana from remote URL
        get_url:
          url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
          dest: "/tmp/kibana-{{ elastic_version }}-linux-x86_64.tar.gz"
          mode: 0755
          timeout: 60
          force: true
          validate_certs: false
        register: get_kibana
        until: get_kibana is succeeded
        tags: kibana
      - name: Create directrory for kibana
        file:
          state: directory
          path: "{{ kibana_home }}"
        tags: kibana
      - name: Extract Kibana in the installation directory
        become: true
        unarchive:
          copy: false
          src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
          dest: "{{ kibana_home }}"
          extra_opts: [--strip-components=1]
          creates: "{{ kibana_home }}/bin/kibana"
        tags:
          - skip_ansible_lint
          - kibana
      - name: Set environment Kibana
        become: true
        template:
          src: templates/kib.sh.j2
          dest: /etc/profile.d/kib.sh
        tags: kibana
```
> Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть
>

![alt](https://i.ibb.co/sgjHxWg/Screenshot-3.jpg)

критических ошибок нет, но есть предупреждения, связанные с тем, что явно не объявлены права на файлы и директории.

Отсутствующий или неподдерживаемый параметр режима может вызвать непредвиденные права доступа к файлу в зависимости от используемой версии Ansible.

Попытался это исправить, используя параметры вида:
```
owner: root
group: root
mode: 0644
```
Проверка синтаксиса прошла успешно:

![alt](https://i.ibb.co/2gnssgV/Screenshot-4.jpg)

> Попробуйте запустить playbook на этом окружении с флагом --check.

```
romrsch@ubuntu:~/8.2/playbook$
romrsch@ubuntu:~/8.2/playbook$ ansible-playbook -i inventory/prod.yml site.yml --check
PLAY [Install Java] ****************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic01]
ok: [kibana01]

TASK [Set facts for Java 8 vars] ***************************************************************************************************************
ok: [elastic01]
ok: [kibana01]

TASK [Upload .tar.gz file containing binaries from local storage] ******************************************************************************
changed: [kibana01]
changed: [elastic01]

TASK [Ensure installation dir exists] **********************************************************************************************************
changed: [elastic01]
changed: [kibana01]

TASK [Extract java in the installation directory] **********************************************************************************************
fatal: [elastic01]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.11' must be an existing dir"}
fatal: [kibana01]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.11' must be an existing dir"}

PLAY RECAP *************************************************************************************************************************************
elastic01                 : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
kibana01                  : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   


```
>Запустите `playbook` на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

```
romrsch@ubuntu:~/8.2/playbook$ 
romrsch@ubuntu:~/8.2/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] ****************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic01]
ok: [kibana01]

TASK [Set facts for Java 8 vars] ***************************************************************************************************************
ok: [elastic01]
ok: [kibana01]

TASK [Upload .tar.gz file containing binaries from local storage] ******************************************************************************
diff skipped: source file size is greater than 104448
changed: [kibana01]
diff skipped: source file size is greater than 104448
changed: [elastic01]

TASK [Ensure installation dir exists] **********************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/11.0.11",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana01]
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/11.0.11",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elastic01]

TASK [Extract java in the installation directory] **********************************************************************************************
changed: [elastic01]
changed: [kibana01]

TASK [Export environment variables] ************************************************************************************************************
--- before
+++ after: /home/romrsch/.ansible/tmp/ansible-local-221239khw7w2m_/tmp_hy3d13m/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.11
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [kibana01]
--- before
+++ after: /home/romrsch/.ansible/tmp/ansible-local-221239khw7w2m_/tmpgb0gs20t/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.11
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [elastic01]

PLAY [Install Elasticsearch] *******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic01]

TASK [Upload tar.gz Elasticsearch from remote URL] *********************************************************************************************
changed: [elastic01]

TASK [Create directrory for Elasticsearch] *****************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/elastic/7.14.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elastic01]

TASK [Extract Elasticsearch in the installation directory] *************************************************************************************
changed: [elastic01]

TASK [Set environment Elastic] *****************************************************************************************************************
--- before
+++ after: /home/romrsch/.ansible/tmp/ansible-local-221239khw7w2m_/tmpxfcyrgrw/elk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export ES_HOME=/opt/elastic/7.14.1
+export PATH=$PATH:$ES_HOME/bin
\ No newline at end of file

changed: [elastic01]

PLAY [Install Kibana] **************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [kibana01]

TASK [Upload tar.gz Kibana from remote URL] ****************************************************************************************************
changed: [kibana01]

TASK [Create directrory for Kibana (/opt/kibana/7.14.1)] ***************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/kibana/7.14.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana01]

TASK [Extract Kibana in the installation directory] ********************************************************************************************
changed: [kibana01]

TASK [Set environment Kibana] ******************************************************************************************************************
--- before
+++ after: /home/romrsch/.ansible/tmp/ansible-local-221239khw7w2m_/tmpq2uv5zf0/kib.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export KIBANA_HOME=/opt/kibana/7.14.1
+export PATH=$PATH:$KIBANA_HOME/bin
\ No newline at end of file

changed: [kibana01]

PLAY RECAP *************************************************************************************************************************************
elastic01                 : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
kibana01                  : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
romrsch@ubuntu:~/8.2/playbook$ 

```

>Повторно запустите `playbook` с флагом `--diff` и убедитесь, что playbook идемпотентен.
>
```
romrsch@ubuntu:~/8.2/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] ****************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic01]
ok: [kibana01]

TASK [Set facts for Java 8 vars] ***************************************************************************************************************
ok: [kibana01]
ok: [elastic01]

TASK [Upload .tar.gz file containing binaries from local storage] ******************************************************************************
ok: [elastic01]
ok: [kibana01]

TASK [Ensure installation dir exists] **********************************************************************************************************
ok: [elastic01]
ok: [kibana01]

TASK [Extract java in the installation directory] **********************************************************************************************
skipping: [elastic01]
skipping: [kibana01]

TASK [Export environment variables] ************************************************************************************************************
ok: [elastic01]
ok: [kibana01]

PLAY [Install Elasticsearch] *******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic01]

TASK [Upload tar.gz Elasticsearch from remote URL] *********************************************************************************************
ok: [elastic01]

TASK [Create directrory for Elasticsearch] *****************************************************************************************************
ok: [elastic01]

TASK [Extract Elasticsearch in the installation directory] *************************************************************************************
skipping: [elastic01]

TASK [Set environment Elastic] *****************************************************************************************************************
ok: [elastic01]

PLAY [Install Kibana] **************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [kibana01]

TASK [Upload tar.gz Kibana from remote URL] ****************************************************************************************************
ok: [kibana01]

TASK [Create directrory for Kibana (/opt/kibana/7.14.1)] ***************************************************************************************
ok: [kibana01]

TASK [Extract Kibana in the installation directory] ********************************************************************************************
skipping: [kibana01]

TASK [Set environment Kibana] ******************************************************************************************************************
ok: [kibana01]

PLAY RECAP *************************************************************************************************************************************
elastic01                 : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
kibana01                  : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

romrsch@ubuntu:~/8.2/playbook$ 

```

https://github.com/romrsch/ansible
