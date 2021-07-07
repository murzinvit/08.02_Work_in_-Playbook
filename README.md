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
        become: true # выполнение таски с root правами <br/>
        file: # создание папки с путём что емеется в java_home<br/>
        state: directory<br/>
        path: “{{ java_home }}”<br/>
        tags: java<br/>
        name: Extract java in the installation directory<br/>
        become: true<br/>
        unarchive: # разорхивация <br/>
        copy: false
        src: “/tmp/jdk-{{ java_jdk_version }}.tar.gz”<br/>
        dest: “{{ java_home }}”<br/>
        extra_opts: [–strip-components=1]<br/>
        creates: “{{ java_home }}/bin/java” <br/>
        tags:java<br/>
        name: Export environment variables<br/>
        become: true<br/>
        template: # копирование из j2 файла в dest<br/>
        src: jdk.sh.j2<br/>
        dest: /etc/profile.d/jdk.sh<br/>
        tags: java<br/>
    name: Install Elasticsearch <br/>
    hosts: elasticsearch # выполнение для группы elasticsearch<br/>
    tasks:<br/>
        name: Upload tar.gz Elasticsearch from remote URL<br/>
        get_url: # загрузка по uRL<br/>
        url: https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz<br/>
        dest: /tmp/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz<br/>
        mode: 0755 # установка разрешений на скачанный файл<br/>
        timeout: 60 #время для загрузки<br/>
        force: true<br/>
        validate_certs: false # не проверять сертификат при скачевании<br/>
        register: get_elastic # переменная с результатом работы таски<br/>
        until: get_elastic is succeeded # 3 попытки загрузки<br/>
        tags: elastic # можно отдельно вызывать таску по этому тегу<br/>
