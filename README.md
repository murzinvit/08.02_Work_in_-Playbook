Прокоментировал строки в playbook с пояснениями что выполняет каждая строка. Readme.md открываю в IntellijIDEA.

name: Install Java
    hosts: all # где исполнять playbook, для группы all
    tasks: # Что исполнять для группы all
        name: Set facts for Java 11 vars
        set_fact: # задаёт переменную доступную во всех тасках playbooka
        java_home: “/opt/jdk/{{ java_jdk_version }}” # path + group_vars
        tags: java # запуск таск в нужном порядке по тегу
        name: Upload .tar.gz file containing binaries from local storage
        copy: # копирование из src в dest
        src: “{{ java_oracle_jdk_package }}”
        dest: “/tmp/jdk-{{ java_jdk_version }}.tar.gz”
        register: download_java_binaries # запись результата таски в переменную
        until: download_java_binaries is succeeded #3 попытки записи в dest
        tags: java
        name: Ensure installation dir exists
        become: true # выполнение таски с root правами
        file: # создание папки с путём что емеется в java_home
        state: directory
        path: “{{ java_home }}”
        tags: java
        name: Extract java in the installation directory
        become: true
        unarchive: # разорхивация 
        copy: false
        src: “/tmp/jdk-{{ java_jdk_version }}.tar.gz”
        dest: “{{ java_home }}”
        extra_opts: [–strip-components=1]
        creates: “{{ java_home }}/bin/java” 
        tags:java
        name: Export environment variables
        become: true
        template: # копирование из j2 файла в dest
        src: jdk.sh.j2
        dest: /etc/profile.d/jdk.sh
        tags: java
    name: Install Elasticsearch 
    hosts: elasticsearch # выполнение для группы elasticsearch
    tasks:
        name: Upload tar.gz Elasticsearch from remote URL
        get_url: # загрузка по uRL
        url: https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz
        dest: /tmp/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz
        mode: 0755 # установка разрешений на скачанный файл
        timeout: 60 #время для загрузки
        force: true
        validate_certs: false # не проверять сертификат при скачевании
        register: get_elastic # переменная с результатом работы таски
        until: get_elastic is succeeded # 3 попытки загрузки
        tags: elastic # можно отдельно вызывать таску по этому тегу
