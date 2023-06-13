# Containerization



## Урок 1. Механизм пространства имен


механизмов два - namespaces и cgroups, и все остальные инструменты (Docker, LXC (Canonical) и так далее) основываются на них.

**Файловая система**

*Абсолютно все объекты в ОС Linux - файлы. Делятся они лишь на типы:*

● Обычные файлы - символьные и двоичные файлы.

● Каталоги (директории) - список ссылок на файлы либо на другие каталоги.

● Символьные ссылки - ссылки на другие файлы и каталоги по имени. 

● Блочные устройства - интерфейсы, предназначенные для взаимодействия с аппаратной составляющей. 

● Сокеты и каналы - интерфейсы для взаимодействия с программной составляющей ОС.


*root - является корнем ФС*


*Другие директории в ОС:*

● /bin –  исполняемые файлы самых необходимых утилит. Эта папка может быть символьной ссылкой на /usr/bin.

● /boot – файлы, которые необходимы для первичного этапа загрузки ОС Linux: загрузки ядра и его компоненты.

● /dev –  блочные и символьные файлы устройств (диски, терминалы, клавиатуры, принтеры и так далее).

● /etc – хнабор конфигурационных файлов системы и прикладных программ.

● /home – домашние каталоги пользователей для хранения «личных» файлов.

● /lib – файлы библиотек, необходимых для работы утилит, системного и прикладного ПО, может быть символьной ссылкой на /usr/bin.

● /mnt – каталог, для подключения файловых систем (съемных носителей и др.).

● /opt – каталог для различного рода дополнительных программ (проприетарных драйверов, агентов мониторинга и др.).

● /proc – одна из важнейших директорий, содержащая в себе сущности-ссылки на файлы, содержащиеся в оперативной памяти, в которых содержится информация о выполняемых в системе процессах.

● /root – домашний каталог пользователя root. 

● /sbin – набор файлов системных утилит, необходимых для загрузки ядра ОС, резервного копирования и восстановления системы. Также может быть символьной ссылкой на /usr/sbin.

● /sys – виртуальная файловая система sysfs, содержит информацию об аппаратном обеспечении (ЦПУ, ОЗУ, дисках, сетевых устройствах), драйверах, ядре системы и др.

● /tmp – каталог для различного рода временных файлов, обычно зачищается при каждой загрузке системы.

● /usr – каталог с исполняемыми и конфигурационными файлами.

● /var – содержит файлы, создаваемые или используемые различными программами (логи, очереди, идентификаторы процессов, базы данных и тд.).


        chroot - изменение корневого каталога сист. для запущенного процесса и всех его дочерних процессов

        Пример:

        chrroot  «имя папки»  /bin/bash - перенос данных из /bin/bash в предварительно созданную директорию.
        
        cp  /bin/bash  «имя папки»/bin – копируем данные (так же копируем все зависимости создав для них необходимые директории ldd  /bin/bash – просмотр зависимостей)
        
**Механизм пространства имен**
  
**Пространство имен (namespaces)** - это абстракция в системе Linux, в которой находятся системные ресурсы. Тип ресурса зависит от типа пространства имен. namespace - это сущность изначально предоставляется самим ядром ОС Linux и является необходимым компонентом, принимающим участие в процедуре запуска любого процесса в системе. В любой момент времени любой процесс может принадлежать только одному пространству имен каждого типа. Также namespace - это механизм, обеспечивающий изоляцию процессов друг от друга в UNIX-системах. 

*Типы namespace:*

PID - изоляция идентификатора процессов

Network - сетевая изоляция (сеть, порты, стеки и так далее)

User - изоляция пользователей и групп

Mount - переопределение точек монтирования

IPC (Inter Process Communication) - изоляция межпроцессного взаимодействия (в т.ч. очереди POSIX)

UTS (UNIX Time-Sharing) - изоляция имени хоста и доменного имени



## Урок 2. Механизмы контрольных групп


**cgroup** — это механизм для иерархической организации процессов и распределения системных ресурсов

**LXC** - подсистема позволяет запускать несколько изолированных друг от друга экземпляров ОС Linux на одном узле. Данная система не использует технологию виртуализации, то есть не создает виртуальных машин. Система создает виртуальное окружение с собственным пространством процессов и сетевым стеком.

*Возможности LXC:*

● Ограничение ресурсов

● Приоритизация (выделение различного количества ресурсов для различных приложений)

● Регистрация затрат тех или иных ресурсов приложением либо группой приложений

● Изоляция

● Управление

*Механизм состоит из двух главных частей: ядра (cgroup core) и подсистем.*

*Список подсистем зависит от версии ядра. Основные компоненты следующие:*

● blkio (block I/O) - это подсистема управления процедурами ввода/вывода блочных устройств. Для ограничения доступа приложению или процессу записываются значения в псевдофайлы. Подсистема содержит огромное количество параметров.

   - blkio.weight - позволяет определить относительный вес (в значениях от 100 до 1000) ввода-вывода контрольной группы. Чем больше значение, тем больше ресурсов получит приложение или процесс.
   
               echo 200 > blkio.weight
               
   - blkio.weight_device - определяет относительный вес для конкретного устройства. Этот параметр позволяет переопределить общее значение blkio.weight.(8:0 номер устройства /dev/sda, проверка с помощью команды lsblk).
 
               echo 8:0 200 > blkio.weight_device
               
● cpu —  отвечает за управление доступом контрольных групп к процессорам системы. Доступ регламентируется параметрами, которые записываются в псевдофайлы. Принцип: один параметр - один псевдофайл. 

   - cpu.shares - Определяет относительный вес для операций ввода/вывод.

   - cpu_rt_runtime_us - этим параметром определяется максимальный период времени (в микросекундах), в течение которого задания того или иного процесса могут использовать процессорные ресурсы. Данный параметр очень важен, так как это ограничение позволяет предотвратить монопольное использование ресурсов одной подгруппой.
  
   - cpu._rt_period_us - здесь же определяется интерфал, по истечении которого приложения из контрольной группы получат доступ к процессорным ресурсам.
  
Пример довольно прост: если нам необходимо, чтобы процессы из группы имели доступ к процессору длиной в 4 секунд каждые 10 секунд, необходимо задать следующие значения:

                cpu.rt_runtime_us 4000000
                
                cpu.rt_period_us 10000000

● cpuacct — формирует отчеты об использовании ресурсов процессора. Всего имеется три вида отчетов:

   - cpuacct.stat - возвращает число циклов процессора (величина измерения - USER_HZ), которые были затрачены на обработку заданий контрольной группы. Учитывается пользовательский и системный режим.
   
   - cpuacct.usage - возвращает суммарное время (единица измерения - наносекунды), в течение которого ресурсы процессора были заняты обработкой заданий контрольной группы.
   
   - cpuacct.usage_percpu - возвращает время, в течение которого ресурсы были заняты обработкой всех заданий контрольной группы, но попроцессорно.

● cpuset — Подсистема, отвечающая за выделение ресурсов процессора контрольным группам. Параметр хранится в отдельном псевдофайле.

   - cpuset.cpus - определяет количество процессоров, к которым могут обращаться процессы в контрольной группе. Записывать в файл можно
как диапазон используемых процессоров (0-2), так и какие-то отдельные через запятую.

   - cpuset.cpu_exclusive - этим параметром можно задать, возможность совместного использования процессоров, которые были перечислены ранее.

● devices — предоставляет доступ или же, наоборот, блокирует доступ к устройствам.

   - devices.allow - Описывает устройства, к которым разрешен доступ в рамках cgroup
 
        Содержит в себе:

   1. Типы устройств:
      
        - a - применяется ко всем символьным и блочным устройствам
        
        - b - блочное устройство
        
        - c - символьное устройство (ссылка на устройство, на файл)
        
   2. Старший и младшие номера. (8:1, 8 - старший номер, обозначающий диски SCSi, 1 - младший номер, обозначающий первый раздел на первом диске)
      
   3. Режим доступа:
      
        - r - доступ разрешен только на чтение
        
        - w - доступ разрешен на чтение и запись
        
        - m - разрешение доступа на создание файлов, если они не существуют
        
   - devices.deny - Описывает устройства, к которым запрещен доступ в рамках cgroup
   
   - devices.list - Описывает устройства, к которым запрещен доступ в рамках cgroup
   
● freezer — приостанавливает и возобновляет выполнение задач в рамках контрольной группы.

   - freezer.state - описывает статус подсистемы
  
        Доступны следующие значения:
        
        - frozen - задания приостановлены
        
        - freezing - система в стадии приостановки задач
        
        - thawed - возобновление работы заданий в группе

● memory —  создает отчеты об использовании ресурсов памяти.

   - memory.stat - статистика использования памяти 
   
   - memory.usage_in_bytes - занятая память процессами из выбранной cgroup
   
   - memory.max_usage_in_bytes - доступный объем памяти (без файла подкачки)
   
   - memory.limit_in_bytes - доступный объем памяти (с файлом подкачки)
   
   - memory.failcnt - счетчик достижения лимита памяти

● net_cls — помечает сетевые пакеты специальным тэгом, который в дальнейшем позволяет идентифицировать пакеты, порождаемые определённой задачей в рамках контрольной группы.

   - net_cls.classid представляет из себя шестнадцатиричное значение, которое идентифицирует обработчик трафика.

● netprio — используется для динамической установки приоритетов по трафику.

● ns - используется для группировки процессов в отдельное пространство имен, где процессы могут взаимодействовать между собой и при этом будут изолированы от внешних процессов.

● pids — используется для ограничения количества процессов в рамках контрольной группы.

● unified — автоматически монтирует файловую систему в каталог /sys/fs/cgroup/unified при запуске системы.

*Помимо специализированных файлов, каждая директория содержит в себе набор управляющих файлов:* 

● cgroup.clone_children — позволяет передавать дочерним контрольным группам свойства родительских.

● tasks — содержит список PID всех процессов, включённых в контрольные группы.

● cgroup.procs — содержит список TGID (Thread Group Id) групп процессов, включённых в контрольные группы.

● cgroup.event_control — позволяет отправлять уведомления в случае изменения статуса контрольной группы.

● release_agent — содержится команда, которая будет выполнена, если включена опция notify_on_release. Может использоваться, например, для автоматического удаления пустых контрольных групп.

● notify_on_release — содержит булеву переменную (0 или 1), включающую (или, наоборот, отключающую), выполнение команду, указанной в release_agent.

**Псевдофайл** - файл, который не представляет из себя файл в базовой файловой системе на диске. Это файлы, которые автоматически создаются для представления какого-либо другого объекта в виде файла с целью возможности взаимодействия с ним. Простой пример /dev/sda.


## Урок 3. Введение в Docker


**Docker** - это платформа, предназначенная для быстрой разработки, развертывания, тестирования и запуска приложений в контейнерах

**Контейнер** - это запущенная программа

**Образ (image)** - это некий неизменный шаблон проекта (либо программы), который используется в дальнейшем для создания контейнеров с одинаковой базой. В таком контейнере содержится базовая система, набор необходимых библиотек и все это собрано в виде сущности - образа, из которого можно создать контейнер.  Образ также содержит в себе и файловую систему, которая в дальнейшем будет доступна приложению, и ряд метаданных (команды, которые будут выполнены при запуске контейнера, например).

**Контейнер (Docker Container)** - это собранный проект, который состоит из образов. По сути, это упакованное приложение на основе образов. Вместе с приложением упаковке подлежит и среда контейнера. Выполняемый контейнер - это запущенный процесс, который изолирован от других процессов на выполняемой системе, имеющий свои ограничения на потребление ресурсов (ЦПУ, ОЗУ, диска и так далее).

**Демон (Docker daemon)** - это служба, управляющая Docker-объектами, такими как: образы, контейнеры, хранилища разного рода, сети и так далее. Также это программная платформа для создания, упаковки, выполнения и распространения приложения. С ее помощью можно скачать образы с хранилища (registry) и запустить из них контейнер.

**Репозиторий (Docker Registry)** - это репозиторий, где хранятся образы. После создания образа на локальном устройстве, им можно поделиться с остальными, отправив образ в registry. А затем - извлечь его оттуда. Примеры репозиториев: Docker Hub, Google Cloud Container Registry и тд. Помимо хранения образов, registry хранит в себе документацию о предоставляемом ПО.

## Урок 4. Dockerfile и слои

**Образы Docker** являются некоторым результатом процесса их сборки. 

**Контейнеры** же - это образы, которые запущены пользователем с целью выполнения каких-либо действий. 

**Dockerfile** - являются инструкцией по сборке образов, на основе которых, в дальнейшем, будут выполняться контейнеры.

**Инструкции Dockerfiles**

- FROM - Задает базовый образ (родительский);

                FROM ubuntu:22.10

- LABEL - Добавление метаданных в образ;

- ENV - Задает постоянные переменные образа (С ее помощью можно задавать глобальные переменные, которые будут использоваться в образе и, впоследствии, в запущенном контейнере.);

- ARG - Задание переменных во время сборки образа;

- RUN - Выполнение команды Linux, создав при этом очередной слой;

- COPY - Копирование файлов и папок;

ADD - Копирование файлов и папок + распаковка архивов;

WORKDIR - Задание рабочей директории для последующих инструкций, описанных в Dockerfile;

CMD - Задание команды, выполняющейся при запуске образа. Эти аргументы можно переопределить при запуске;

ENTRYPOINT - Задание команды, выполняющейся при запуске образа, нельзя переопределить;

EXPOSE - Открытие порта в образ/контейнер

VOLUME - Создание точки монтирования

## Урок 5. Docker Compose и Docker Swarm

**Docker Compose (ДК)** - это инструментальное средство, которое находится в составе докера. ДК позволяет управлять несколькими контейнерами из одного yml-файла.

Разница между Docker и Docker Compose состоит в том, что докер предназначен для управления одиночными контейнерами, из которых может состоять приложение. ДК же позволяет управлять сразу несколькими контейнерами, запуская и останавливая их в одну команду.

**YAML** — это язык для сериализации данных. Позволяет хранить сложноорганизованные данные в компактном и читаемом формате. YAML — это язык для хранения информации в формате понятном человеку.

**Особенности YAML:**

● Понятный человеку код

● Минималистичный синтаксис

● Заточен под работу с данными

● Встроенный стиль, похожий на JSON

● Поддерживает комментарии

● Поддерживает строки без кавычек

● Считается «чище», чем JSON

● Дополнительные возможности

Пример YAML файла:

                version: ‘3.9’
                services:
                db:
                        image: mariadb:10.10.2
                        restart: always
                        environment:
                                MYSQL_ROOT_PASSWORD: 12345
                adminer:
                        image: adminer:4.8.1
                        restart: always
                        ports:
                                - 6080:8080

● version: ‘3.9’ - в этой строке описывается версия compose-файла. Каждый файл обязательно должен начинаться с тега версии. Глобально версий всего три. Первая, которая более не поддерживается, вторая и третья. Каждой версии файла соответствует определенная версия докера, установленная на машине. Например, для версии 3.1 необходимо, чтобы установленный докер был не ниже версии 1.13.1, а вот версия 3.8 уже требует минимум версию 19.03.0.

● services - эта строка говорит о том, что далее в файле будут описаны сервисы (один или несколько), которые необходимо запустить или остановить (зависит от команды, передаваемой в докер). Важно помнить, что ДК работает с сервисами (1 сервис = 1 контейнер). Сервисом при этом может быть клиент, сервер, сервер БД и так далее.

● db: - это название первого сервиса. Название каждый может придумать какое угодно вам. Понятное название сервиса поможет определить его роль и сделать манифесты более читаемыми.

● build: ./db - ключевое слово, позволяющее задать путь к файлу докерфайл, который будет использован для создания собственного образа и который, в свою очередь, позволит запустить сервис.

● image: mariadb - если вместо build указан этот вариант, система будет использовать готовые образы с docker hub. На лекции, в примерах, мы будем использовать этот вариант. На практике попробуем разобрать варианты с построением собственных образов.

● restart: always - позволяет определить политику перезапуска контейнера.

Существует несколько вариантов:

        ○ no - не перезапускать контейнер автоматически. Это значение по умолчанию. Если мы ничего не укажем, контейнер не будет перезапускаться
       
        ○ on-failure [max retries] - перезапускать контейнер, если он был завершен не с нулевым кодом выхода. В данном случае необходимо указать максимальное количество попыток перезапуска
        
        ○ always - всегда перезапускать контейнер, если он был остановлен. Здесь стоит быть аккуратнее в связи с тем, что в этом случае возможен цикличный перезапуск контейнера с проблемами
        
        ○ unless-stopped - всегда перезапускать контейнер. По сути, этот вариант аналогичен always кроме случаев, когда он был остановлен вручную. При этом он не перезапускается после перезапуска демона докера.
        
        ○ environment - в этом блоке прописываются переменные, которые могут быть использованы для работы контейнера
        
        ○ ports - указывает на то, какие порты необходимо открыть для связи с контейнером

**Оркестрация контейнеров** — это централизованное и эффективное управление, а также - мониторинг контейнеров

Инструмент оркестрации должен иметь как минимум 3 ключевых элемента:

 - Развертывание - это возможность эффективно деплоить наши контейнеры в соответствии с заданными параметрами

 - масштабирование - возможность увеличения количества запущенных контейнеров простым способом.

 - Надежность - как только мы объявим контейнеры, которые необходимо развернуть для нашего приложения, оркестратор должен позаботиться о том, чтобы были развернуты они и вся инфраструктура, которая необходима.

**Основные команды ДК**

● docker-compose build - команда позволяет собрать сервисы, описанные в конфигурационных файлах

● docker-compose up -d - запускает наш проект. В данном случае проект запустится в фоновом режиме, так как в команде присутствует флаг -d

● docker-compose start - запускает любые остановленные ранее сервисы в соответствии с указанными параметрами

● docker-compose down - останавливает наш проект и, что немаловажно, удаляет все сервисы, которые были запущены ранее

● docker-compose stop - эта команда просто останавливает все сервисы, описанные в конфигурации. Она не удаляет контейнеры, тома, сети и прочие сущности, описанные в конфигурационном файле

● docker-compose logs -f [service name] - с помощью этой команды можно посмотреть логи нашего сервиса

● docker-compose ps - выводит на экран список всех доступных контейнеров

● docker-compose exec [service name] [command] - с ее помощью можно выполнить команду в сервисе, не заходя при этом в контейнер. Ранее мы рассматривали подобное на уроке “Введение в Docker”

● docker-compose images - позволяет вывести список образов.

**Docker Swarm (ДС)** - это стандартный оркестратор для контейнеров, который изначально встроен в механизм работы докера. Он отлично подойдет для развертывания приложений в рабочей среде кластера. Его несомненный плюс - не нужно будет переписывать наши ДК файлы, а просто применять их для работы с ДС.

1. Отказоустойчивость - при выходе из строя одной ноды из трех наши контейнеры смогут продолжать работу на остальных узлах кластера.

2. Увеличение ресурсов - в случае работы в кластере, существенно увеличивается мощность, доступная контейнерам для работы.

**Node (нода)** -  это наш сервер с установленным на нем Docker. По сути, нодой могут быть как физические сервера, так и виртуальные машины 

**Stack** - это набор сервисов, которые могут быть связаны между собой логически

**Сервис** - это составляющая стэка

**Task (задача)** - непосредственно созданный контейнер

у ДС есть два типа нод (узлов):

● manager node - управляющий сервер. Он способен управлять нашим кластером: добавлять и удалять сущности (например, ноды кластера). На нем также возможно запускать и контейнеры.

● worker node - на этом узле возможен лишь запуск контейнеров. Управление кластера с данного типа узлов недоступен.

**Overlay сети**

● Overlay - создает простую подсеть, которая может быть использована контейнерами на разных хостах swarm-кластера

● ingress - этот тип сети используется в ДС по умолчанию при создании кластера. Она отвечает за связи, которые устанавливаются между контейнерами и внешним миром

● vxlan - при использовании этого типа сетей, у нас не просто происходит создание Overlay-сети. В этом случае происходит инкапсуляция пакетов 2 слоя модели OSI в четвертый

● docker_gwbridge - эта сеть создается на каждом узле кластера. Она позволяет соединить трафик из контейнеров, находящихся внутри ДС кластера с внешним миром

**Использование ДК и ДС**

Совет 1 - используйте виртуальные сети для различных проектов

        Преимущества изоляции сетей:
        
        ● Разные наборы окружений = разные сети. Полная изоляция пакетов данных
        
        ● Ограничение прослушивания адреса 0.0.0.0
        
        ● Разграничение сред выполнения

Совет 2 - Корректно открывать порты, не используя адрес 0.0.0.0

        Проброс портов (NAT) Пример:
        
        ● 8080:80
        
        ● 127.0.0.1:8080:8080


