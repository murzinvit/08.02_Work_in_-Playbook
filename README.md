#### Прокоментировал строки в playbook с пояснениями что выполняет каждая строка. Проверил в docker, playbook работает.</br>
#### После выполнения надо зайти по docker exec в контейнер и стартануть /etc/init.d/kibana start. Kibana доступна на порту 5601.</br>
======================================================================================</br>
`name: Install Java`  # имя playbook</br>
`hosts: all`          # где исполнять playbook, для группы all</br>
`tasks:`              # Что исполнять для группы all</br>
`name: Set facts for Java 11 vars` # Имя таски </br>
`set_fact:` # задаёт переменную доступную во всех тасках playbooka</br>
`java_home: “/opt/jdk/{{ java_jdk_version }}”` # path + group_vars</br>
`tags: java` # запуск таск в нужном порядке по тегу</br>
`name: Upload .tar.gz file containing binaries from local storage`</br>
`copy:` # копирование из src в dest</br>
`src: “{{ java_oracle_jdk_package }}”`</br>
`dest: “/tmp/jdk-{{ java_jdk_version }}.tar.gz”`</br>
`register: download_java_binaries` # запись результата таски в переменную</br>
`until: download_java_binaries is succeeded` #3 попытки записи в dest</br>
`tags: java`</br>
`name: Ensure installation dir exists`</br>
`become: true` # выполнение таски с root правами</br>
`file:` # создание папки с путём что емеется в java_home</br>
`state: directory` # указание что создание не файла а директории</br>
`path: “{{ java_home }}”`</br>
`tags: java`</br>
`name: Extract java in the installation directory`</br>
`become: true`</br>
`unarchive:` # разорхивация</br>
`copy: false`</br> # Искать архив на управляемой машине
`src: “/tmp/jdk-{{ java_jdk_version }}.tar.gz”` # источник</br>
`dest: “{{ java_home }}”` # путь куда разорхивируем </br>
`extra_opts: [–strip-components=1]` # Разархивировать файлы без папки архива</br>
`creates: “{{ java_home }}/bin/java”`</br>
`tags:java`</br>
`name: Export environment variables`</br>
`become: true` # С правами root</br> 
`template:` # копирование из j2 файла в dest</br>
`src: jdk.sh.j2` #Имя источника jinja</br>
`dest: /etc/profile.d/jdk.sh`#Место назначение</br>
`tags: java`</br>
`name: Install Elasticsearch`</br>
`hosts: elasticsearch `# выполнение для группы elasticsearch</br>
`tasks:`</br>
`name: Upload tar.gz Elasticsearch from remote` URL</br>
`get_url:` # загрузка по uRL</br>
`url: https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz`</br>
`dest: /tmp/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz`</br>
`mode: 0755` # установка разрешений на скачанный файл</br>
`timeout: 60` #время для загрузки</br>
`force: true`</br>
`validate_certs: false` # не проверять сертификат при скачевании</br>
`register: get_elastic` # переменная с результатом работы таски</br>
`until: get_elastic is succeeded` # 3 попытки загрузки</br>
`tags: elastic` # можно отдельно вызывать таску по этому тегу</br>
`name: Create directrory for Elasticsearch`</br>
`file: # создание папки`</br>
