# Как устроена система логирования в Андройде

## Клиент

Обычно в С++-коде чтобы написать сообщение в лог Андройда мы используем следующий код:

``` cpp
// main.cpp
#define LOG_TAG "MyApp.Main"
#include <cutils/log.h>`

int main() {
    ALOGD("Show me this number in logcat: %d", 42);
    return 0;
} 
```

Для передачи сообщения в лог, мы используем функцию (на самом деле это #define) __ALOGD__, которая определена в хэдере _cutils/log.h_. С помощью дефайна __LOG_TAG__ мы указываем тэг для всех сообщений, которые будут отправлены из файла _main.cpp_.

Файл _cutils/log.h_ в исходниках AOSP находится здесь: _aosp/system/core/include/cutils/log.h_. Если мы в него заглянем, то с удивлением узнаем, что файл содержит в себе только одну строчку:

``` cpp
 #include <log/log.h>`
```

Хорошо, давайте теперь посмотрим что внутри внутри _log/log.h_, благо находится он поблизости:  _aosp/system/core/include/log/log.h_.

В этом файле определены всем нам привычные дефайны, которые мы используем для логирования. Например, вот __ALOGD__:

``` cpp
#ifndef ALOGD
#define ALOGD(...) ((void)ALOG(LOG_DEBUG, LOG_TAG, __VA_ARGS__))
#endif
```

Как мы видим, все они, в свою очередь, передают входные данные в общий дефайн __ALOG__, в который в качестве первого аргумента передается уровень логирования. Хоть в нашем случае и используется уровень логирования _LOG\_DEBUG_, но в коде AOSP что-то вроде `#define LOG_DEBUG 1` мы найти не сможем. Вместо него в файле _aosp/system/core/include/android/log.h_ имеется следующий _enum_:

``` cpp
/*
 * Android log priority values, in ascending priority order.
 */
typedef enum android_LogPriority {
    ANDROID_LOG_UNKNOWN = 0,
    ANDROID_LOG_DEFAULT,    /* only for SetMinPriority() */
    ANDROID_LOG_VERBOSE,
    ANDROID_LOG_DEBUG,
    ANDROID_LOG_INFO,
    ANDROID_LOG_WARN,
    ANDROID_LOG_ERROR,
    ANDROID_LOG_FATAL,
    ANDROID_LOG_SILENT,     /* only for SetMinPriority(); must be last */
} android_LogPriority;
```

Вот эти значения и используются в качестве первого аргумента для __ALOG__. Каким образом узнаем чуть позже.

Вторым аргументом для __ALOG__ идет переменная _LOG\_TAG_, которая как раз и является тем самым дефайном из _main.cpp_ с тэгом для лог-сообщений.

Последним аргементом идет некий _\_\_VA\_ARGS\_\__. Это [специальный идентификатор][va_args_doc], который при дальнейшей макроподстановке (macro expansion) будет преобразован в список аргументов, переданных в изначальный макрос. В нашем случае _\_\_VA\_ARGS\_\__ будет содержать только один элемент - число 42. 

Теперь мы можем переходить к определению __ALOG__:

``` cpp
#ifndef ALOG
#define ALOG(priority, tag, ...)  LOG_PRI(ANDROID_##priority, tag, __VA_ARGS__)
#endif
```

Похоже, we need to ge deeper - __ALOG__ просто передает свои аргументы в следующий дефайн __LOG_PRI__. Обратите внимание, как _LOG\_DEBUG_, содержащийся в параметре _priority_, превращается в _ANDROID\_LOG\_DEBUG_ с помощью оператора _##_. Эта валидная магия называется [token concatenation][token_concat].

Переходим к определению __LOG_PRI__:

``` cpp
#ifndef LOG_PRI
#define LOG_PRI(priority, tag, ...) \
    android_printLog(priority, tag, __VA_ARGS__)
#endif
```

Хм, ладно, ищем __android_printLog()__:

``` cpp
#define android_printLog(prio, tag, fmt...) \
    __android_log_print(prio, tag, fmt)
```

Переходим в __android_log_print()__. Эта функция находится в файле _aosp/system/core/liblog/logd\_write.c_:

``` cpp
#define LOG_BUF_SIZE 1024
...

int __android_log_print(int prio, const char *tag, const char *fmt, ...)
{
    va_list ap;
    char buf[LOG_BUF_SIZE];

    va_start(ap, fmt);
    vsnprintf(buf, LOG_BUF_SIZE, fmt, ap);
    va_end(ap);

    return __android_log_write(prio, tag, buf);
}
```

Главная задача этой функции - скомпоновать единую строчку из строки-формата (`fmt`) и неизвестного количества дополнительных аргументов. Для этого используется [va_list][va_list_doc] - это тип объекта, который способен хранить информацию, необходимую для функций [va_start][va_start_doc] и [va_end][va_end_doc]. Вся магия подстановок происходит в [vsnprintf][vsnprintf_doc]. В ней вместо спецификаторов преобразований (conversion specificators) (в виде хорошо нам известных _%d_, _%s_ и др.) подставляются значения переменных из списка аргументов функции, доступ к которым предоставляет _va\_list_. В итоге в буфере `buf` мы получаем финальную версию лог-сообщения.

После этого мы переходим в функцию __\__android_log_write()__, которая сразу перенаправляет нас в __\__android_log_buf_write()__:

``` cpp
int __android_log_write(int prio, const char *tag, const char *msg)
{
    return __android_log_buf_write(LOG_ID_MAIN, prio, tag, msg);
}

int __android_log_buf_write(int bufID, int prio, const char *tag, const char *msg)
{
    struct iovec vec[3];
    char tmp_tag[32];

    if (!tag)
        tag = "";

    if ((bufID != LOG_ID_RADIO) &&
         (!strcmp(tag, "HTC_RIL") ||
        !strncmp(tag, "RIL", 3) || /* Any log tag with "RIL" as the prefix */
        !strncmp(tag, "IMS", 3) || /* Any log tag with "IMS" as the prefix */
        !strcmp(tag, "AT") ||
        !strcmp(tag, "GSM") ||
        !strcmp(tag, "STK") ||
        !strcmp(tag, "CDMA") ||
        !strcmp(tag, "PHONE") ||
        !strcmp(tag, "SMS"))) {
            bufID = LOG_ID_RADIO;
            /* Inform third party apps/ril/radio.. to use Rlog or RLOG */
            snprintf(tmp_tag, sizeof(tmp_tag), "use-Rlog/RLOG-%s", tag);
            tag = tmp_tag;
    }

    ...

    vec[0].iov_base   = (unsigned char *) &prio;
    vec[0].iov_len    = 1;
    vec[1].iov_base   = (void *) tag;
    vec[1].iov_len    = strlen(tag) + 1;
    vec[2].iov_base   = (void *) msg;
    vec[2].iov_len    = strlen(msg) + 1;

    return write_to_log(bufID, vec, 3);
}
```

Что здесь можно отметить. Во-первых, мы видим, что все сообщения по умолчанию попадают в лог _main_. Но если тэг сообщения как-то связан с радиомодулем смартфона (ключевые слова "GSM", "RIL", "PHONE" и другие), то сообщения будет перенаправлено в лог _radio_. Во-вторых, с этого момента приоритет сообещния, тэг и тело сообщения упаковываются в универсальную структуру - __struct iovec__ - определение которой можно найти в файле _repo/system/core/include/log/uio.h_:

``` cpp
struct iovec {
    const void*  iov_base;
    size_t       iov_len;
};
```

Далее мы перемещаемся в функцию __write_to_log()__:

``` cpp
static int __write_to_log_init(log_id_t, struct iovec *vec, size_t nr);
int (*write_to_log)(log_id_t, struct iovec *vec, size_t nr) = __write_to_log_init;
```

Ага, __write_to_log()__ это только указатель на функцию, который по умолчанию ссылается на __\__write_to_log_init()__:

``` cpp
static int __write_to_log_init(log_id_t log_id, struct iovec *vec, size_t nr)
{
    pthread_mutex_lock(&log_init_lock);
    if (write_to_log == __write_to_log_init) {
        int ret;
        ret = __write_to_log_initialize();
        if (ret < 0) {
            pthread_mutex_unlock(&log_init_lock);
            return ret;
        }

        write_to_log = __write_to_log_daemon;
    }

    pthread_mutex_unlock(&log_init_lock);
    return write_to_log(log_id, vec, nr);
}
```

Задача __\__write_to_log_init()__ проста - убедиться, что процедура инициализации логирования была выполнена. Если нет, то мы вызываем метод __\__write_to_log_initialize()__. После успешной инициализации мы изменяем указатель __write_to_log()__ - теперь он будет ссылаться на метод __\__write_to_log_daemon()__.

Рассмотрим процедуру инициализации логирования:

``` cpp
#define SOCKET_TIME_OUT 5
static int logd_fd = -1;
static int pstore_fd = -1;
...

static int __write_to_log_initialize()
{
    int i, ret = 0;
    if (pstore_fd < 0) {
        pstore_fd = TEMP_FAILURE_RETRY(open("/dev/pmsg0", O_WRONLY));
    }

    if (logd_fd < 0) {
        i = TEMP_FAILURE_RETRY(socket(PF_UNIX, SOCK_DGRAM | SOCK_CLOEXEC, 0));
        if (i < 0) {
            ret = -errno;
        } else {
            struct sockaddr_un un;
            struct timeval tm;

            tm.tv_sec = SOCKET_TIME_OUT;
            tm.tv_usec = 0;
            setsockopt(i, SOL_SOCKET, SO_RCVTIMEO, &tm, sizeof(tm));
            setsockopt(i, SOL_SOCKET, SO_SNDTIMEO, &tm, sizeof(tm));
            memset(&un, 0, sizeof(struct sockaddr_un));
            un.sun_family = AF_UNIX;
            strcpy(un.sun_path, "/dev/socket/logdw");

            if (TEMP_FAILURE_RETRY(connect(i, (struct sockaddr *)&un,
                                           sizeof(struct sockaddr_un))) < 0) {
                ret = -errno;
                close(i);
            } else {
                logd_fd = i;
            }
        }
    }

    return ret;
}
```

Суть инициализация, оказывается, сводится к получению двух файловых дескрипторов - `pstore_fd` и `logd_fd`. 

Проще всего разобраться (на первый взгляд) с `pstore_fd` - это файловый дескриптор, который мы получаем с помощью функции [open()][open_posix_doc]. Через него мы можем только записывать данные (флаг _O\_WRONLY_) в некое устройство, скрывающееся за __"/dev/pmsg0"__. Что это за устройство, узнаем чуть позже.

Сделаем небольшое отступление и обратим внимание на __TEMP_FAILURE_RETRY__.

``` cpp
#ifndef TEMP_FAILURE_RETRY
/* Used to retry syscalls that can return EINTR. */
#define TEMP_FAILURE_RETRY(exp) ({         \
    typeof (exp) _rc;                      \
    do {                                   \
        _rc = (exp);                       \
    } while (_rc == -1 && errno == EINTR); \
    _rc; })
#endif
```

Да, это еще один дефайн, который готов выполнять заданную операцию - _exp_ - до тех пор, пока она не будет выполнена. Дело в том, что существует довольно большое количество операций (системные вызовы, функции библиотек), которые могут быть прерваны обработчиками сигналов (подробнее в разделе ["Interruption of system calls and library functions by signal handlers"][signal_doc]). В этом случае операция либо автоматически перезапускается, либо она завершается с отрицательным результатом и в переменную `errno` помещается значение [EINTR][errno_doc]. Для отлавливания последнего варианта развития событий как раз и используется данный прием - в цикле _do-while_ мы проверяем значение `errno` и при необходимости повторно выполняем нашу операцию.

Возвращаемся обратно в функцию инициализации и переходим к `logd_fd`. По сути это [дескриптор сокета][socket_doc], созданный со следущими параметрами:
- [PF_UNIX][unix_socket_doc] - используем семейство сокетов, которые применяются для обмена данными между процессами, работающими на одной и той же машине;
- [SOCK_DGRAM][socket_doc] - тип [сокета][unix_socket_doc], который предполагает обмен данными при помощи датаграмм. Т.е. предварительное соединение не устанавливается, подтверждения о доставке сообщения нет и длина сообщения имеет фиксированную максимальную длину;
- __SOCK_CLOEXEC__ - данный флаг используется для предотвращения утечки
файлового дескриптора в многопоточных приложениях (см. [O_CLOEXEC][open_doc]).

После того, как мы получили валидный (>= 0) дескриптор сокета, мы производим
его настройку. Устанавливается время таймаута для операций отправки/получения
данных через [setsockopt][setsockopt_doc] (см. [SO_RCVTIMEO и SO_SNDTIMEO][socket7_doc]) с помощью структуры [__struct timeval__][timeval_doc]. Затем устанавливается путь до папки __"/dev/socket/logdw"__ к которой осуществляется привязка сокета с помощью структуры [struct sockaddr_un][unix_socket_doc].

После настройки сокета мы пробуем установить соединение при помощи [connect()][connect-doc]. Если соединение успешно установилось, то мы сохраняем созданный дескриптор сокета в `logd_fd`.

Та-даа! Инициализация закончена. Теперь можно переходить к функции, которая осуществляе дальнейшую передачу лог-сообщений - __\__write_to_log_daemon()__. Вспомним аргументы функции:
- __log_id__ - ID лога, в который попадет наше сообщений (main, system, radio ...);
- __vec__ - массив структур _iovec_, который содержит в себе приоритет сообщения, тэг и, собственно, текст сообщения;
- __nr__ - размер массива __vec__. В нашем случае nr = 3.

Начало функции - объявление переменных и структур данных:

``` cpp
static int __write_to_log_daemon(log_id_t log_id, struct iovec *vec, size_t nr)
{
    ssize_t ret;
    static const unsigned header_length = 2;
    struct iovec newVec[nr + header_length];
    android_log_header_t header;
    android_pmsg_log_header_t pmsg_header;
    struct timespec ts;
    size_t i, payload_size;
    static uid_t last_uid = AID_ROOT; /* logd *always* starts up as AID_ROOT */
    static pid_t last_pid = (pid_t) -1;
    static atomic_int_fast32_t dropped;

    if (!nr) {
        return -EINVAL;
    }
```

Из интересного и нового здесь, разве что две структуры данных для описания header-ов пакетов(?), определение которых можно найти в файле 'repo/system/core/include/private/android_logger.h':

``` cpp
#define LOGGER_MAGIC 'l'

typedef struct __attribute__((__packed__)) {
    uint8_t magic;
    uint16_t len;
    uint16_t uid;
    uint16_t pid;
} android_pmsg_log_header_t;

typedef struct __attribute__((__packed__)) {
    typeof_log_id_t id;
    uint16_t tid;
    log_time realtime;
} android_log_header_t;
```

Следующий шаг - получение UID (User ID) и PID (Process ID):

``` cpp
    if (last_uid == AID_ROOT) { /* have we called to get the UID yet? */
        last_uid = getuid();
    }
    if (last_pid == (pid_t) -1) {
        last_pid = getpid();
    }
```

Затем мы заполняем поля структур `pmsg_header` и `header` и сохраняем ссылки на них в первых двух элементах `newVec`:

``` cpp
    clock_gettime(CLOCK_REALTIME, &ts);

    pmsg_header.magic = LOGGER_MAGIC;
    pmsg_header.len = sizeof(pmsg_header) + sizeof(header);
    pmsg_header.uid = last_uid;
    pmsg_header.pid = last_pid;

    header.tid = gettid();
    header.realtime.tv_sec = ts.tv_sec;
    header.realtime.tv_nsec = ts.tv_nsec;

    newVec[0].iov_base   = (unsigned char *) &pmsg_header;
    newVec[0].iov_len    = sizeof(pmsg_header);
    newVec[1].iov_base   = (unsigned char *) &header;
    newVec[1].iov_len    = sizeof(header);

    ...

    header.id = log_id;
```

После этого проверяется, были ли неудачные записи лог-сообщений, и если да, то число неудачных попыток записывается в `logd_fd`. Пропустим эту часть функции и перейдем к записи текущего сообщения. Сначала данные из `vec` копируются в `newVec`. При этом мы следим, чтобы общий пересылаемый объем данных не превысил `LOGGER_ENTRY_MAX_PAYLOAD` - 4076 байт (дефайн задан в _repo/system/core/include/log/logger.h_). Затем мы записываем данные из `newVec` в `pstore_fd` с помощью [writev()][readv_writev_doc]:

``` cpp
    for (payload_size = 0, i = header_length; i < nr + header_length; i++) {
        newVec[i].iov_base = vec[i - header_length].iov_base;
        payload_size += newVec[i].iov_len = vec[i - header_length].iov_len;

        if (payload_size > LOGGER_ENTRY_MAX_PAYLOAD) {
            newVec[i].iov_len -= payload_size - LOGGER_ENTRY_MAX_PAYLOAD;
            if (newVec[i].iov_len) {
                ++i;
            }
            payload_size = LOGGER_ENTRY_MAX_PAYLOAD;
            break;
        }
    }
    pmsg_header.len += payload_size;

    if (pstore_fd >= 0) {
        TEMP_FAILURE_RETRY(writev(pstore_fd, newVec, i));
    }
```

Затем те же самые данные (кроме `pmsg_header`) записываются в `logd_fd`: 

``` cpp
    ret = TEMP_FAILURE_RETRY(writev(logd_fd, newVec + 1, i - 1));
```

В конце функции __\__write_to_log_daemon()__ мы проверяем результат записи в `logd_fd`. Если что-то пошло не так, то мы вновь пробуем проинициализировать логирование с помощью __\__write_to_log_initialize()__ и повторяем запись. Ну а если лог-сообщения ушли адресатам, то функция с чистой совестью завершает свое выполнение.

Вот так, если ~~вкратце~~ подробно устроено логирование в Андройде на стороне клиента. Теперь пора переходить к серверной части, т.к. кто-то же должен принять наши сообщения?

## Сервер

Как мы выяснили из предыдущей части, лог-сообщения пересылаются в два места - __"/dev/pmsg0"__ и __"/dev/socket/logdw"__. Начнем выяснять что за ними скрывается.

### /dev/pmsg0

Поиски _"pmsg0"_ по файлам AOSP-а приводят нас к файлу _repo/kernel/fs/pstore/Kconfig_, в котором сказано примерно следующее:

``` txt
config PSTORE
    bool "Persistent store support"
    default n
    select ZLIB_DEFLATE
    select ZLIB_INFLATE
    help
       This option enables generic access to platform level
       persistent storage via "pstore" filesystem that can
       be mounted as /dev/pstore ...

...

config PSTORE_PMSG
    bool "Log user space messages"
    depends on PSTORE
    help
      When the option is enabled, pstore will export a character
      interface /dev/pmsg0 to log user space messages. On reboot
      data can be retrieved from /sys/fs/pstore/pmsg-ramoops-[ID].
```

Покопавшись в интернетах на тему "что такое _pstore_", можно наткнуться на [статью][persistent_storage_article], в которой нам объясняют, что главной задачей _pstore_ является запись логов системы в некую заранее выделенную постоянную память (persistent storage) (например, Flash) при возникновении ситуаций типа [kernel oops][kernel_oops] или даже [kernel panic][kernel_panic]. Т.к. логи будут записаны в энергонезависимую память, то после перезагрузки системы мы сможем их найти в папке _/sys/fs/pstore_ и подвергнуть анализу.

Допустим, на нашем девайсе ядро сконфигурировано с включенной опцией `PSTORE_PMSG`. Регистрация _pstore_ начинается с вызова функции __pstore_register()__, объявление которой можно найти в _repo/kernel/include/linux/pstore.h_, а реализацию в _repo/kernel/fs/pstore/platform.c_. В этой функции нас интересует вызов __pstore_register_pmsg()__, которая определена в _repo/kernel/fs/pstore/pmsg.c_:

``` cpp
static const struct file_operations pmsg_fops = {
    .owner      = THIS_MODULE,
    .llseek     = noop_llseek,
    .write      = write_pmsg,
};

static struct class *pmsg_class;
#define PMSG_NAME "pmsg"

void pstore_register_pmsg(void)
{
    struct device *pmsg_device;
    pmsg_major = register_chrdev(0, PMSG_NAME, &pmsg_fops);
    ...

    pmsg_class = class_create(THIS_MODULE, PMSG_NAME);
    ...

    pmsg_class->devnode = pmsg_devnode;

    pmsg_device = device_create(pmsg_class, NULL, MKDEV(pmsg_major, 0),
                    NULL, "%s%d", PMSG_NAME, 0);
    ...

    return;
}
```

Судя по коду, именно с помощью __device_create()__ создается наш __"/dev/pmsg0"__, а запись в него осуществляется с помощью фунции __write_pmsg()__, которая определена в этом же файле.

А как можно убедиться, что перед падением системы логи будут записаны? Очень просто - нужно уронить систему и посмотреть появится ли что-нибудь в папке _/sys/fs/pstore_. Для нашего коварного плана воспользуемся системой [SysRq][linux_system_request] и выполним на устройстве следующую команду:

``` bash
echo c > /proc/sysrq-trigger
```

Команда спровоцирует падение системы. Возможно, устройство перезагрузится автоматически, а возможно придется самостоятельно перезапустить девайс. Далее на девайсе проверяем папку _/sys/fs/pstore_:

``` bash
root@aosp-device:/ # ls -l /sys/fs/pstore
-r--r----- system   log         65524 2018-06-26 15:30 console-ramoops
-r--r--r-- root     root         1868 2018-06-26 15:23 dmesg-ramoops-0.enc.z
-r--r--r-- root     root         1593 2018-06-26 15:23 dmesg-ramoops-1.enc.z
-r--r----- system   log         65524 2018-06-26 15:30 pmsg-ramoops-0

```

В файле _pmsg-ramoops-*_ как раз и будут находится последние лог-сообщения системы перед падением. Также можно взглянуть и на лог dmesg, который находится в соседних файлах.

### /dev/socket/logdw

В этот раз глубоко залазить в дебри Андройда не придется - за сокетом logdw скрывается сервис __logd__, исходники которого лежат здесь: _repo/system/core/logd_. На девайсе бинарный файл __logd__ находится в папке _/system/bin_. Он запускается при старте системы благодаря процессу __init__ (TODO объяснить что такое init процесс). Команду запуска можно найти в файле _repo/system/core/rootdir/init.rc_:

``` txt
service logd /system/bin/logd
    class core
    socket logd stream 0666 logd logd
    socket logdr seqpacket 0666 logd logd
    socket logdw dgram 0222 logd logd
    group root system
```

Как видим, с __logd__ связано минимум три сокета - _logd_, _logdw_ и _logdr_. С сокетом _logdw_ мы уже встречались - через него лог-сообщения пересылаются в __logd__. По аналогии можно предположить, что сокет _logdr_ используется для получения (чтения) лог-сообщений из __logd__. А вот с сокетом _logd_ пока не понятно.

Перейдем к коду, а именно к функции 'main()'. Здесь разработчики решили время зря не терять и с первых строк приступили к делу. Для начала, мы создаем файловые дескрипторы к __"/proc/kmsg"__ (в зависимости от системных настроек (system properties)) и к __"/dev/kmsg"__:

``` cpp
bool klogd = property_get_bool("logd.klogd", false);
if (klogd) {
    fdPmesg = open("/proc/kmsg", O_RDONLY | O_NDELAY);
}
fdDmesg = open("/dev/kmsg", O_WRONLY);
kernelLogFd = fdDmesg;
```

Затем мы проверяем было ли нам в качесстве аргумента передано ключевое слово __"reinit"__. Если да, то __logd__ открывает тот самый сокет __"logd"__ и передает в него сообщение _"reinit"_ и ждет ответа _"success"_:

``` cpp
if ((argc > 1) && argv[1] && !strcmp(argv[1], "--reinit")) {
        int sock = TEMP_FAILURE_RETRY(
            socket_local_client("logd",
                                ANDROID_SOCKET_NAMESPACE_RESERVED,
                                SOCK_STREAM));
        ...

        static const char reinit[] = "reinit";
        ssize_t ret = TEMP_FAILURE_RETRY(write(sock, reinit, sizeof(reinit)));
        ...

        struct pollfd p;
        memset(&p, 0, sizeof(p));
        p.fd = sock;
        p.events = POLLIN;
        ret = TEMP_FAILURE_RETRY(poll(&p, 1, 100));
        ...

        static const char success[] = "success";
        char buffer[sizeof(success) - 1];
        memset(buffer, 0, sizeof(buffer));
        ret = TEMP_FAILURE_RETRY(read(sock, buffer, sizeof(buffer)));
        ...

        return strncmp(buffer, success, sizeof(success) - 1) != 0;
    }
```

Кто принимает это сообщение и что после этого происходит - узнаем позже (TODO). Сейчас двигаемся дальше - запускаем detached-поток, который будет выполнять функцию _reinit\_thread\_start()_. Ее задача - найти имя пользователя по заданному UID (User ID) в файле _"/data/system/packages.list"_.

Следующий шаг - ограничить привелегии процесса. Т.к. сервис __logd__ запускает __init__, то мы автоматом получаем все его системные привелегии. А это, между прочим, практически привелегии root-пользователя (TODO: проверить так ли это). На сколько я помню, во времена до Android 5.x много методов получения root-прав на устройстве как раз основывались на использовании уязвимости в каком-либо системном процессе, который не сбросил свои привелегии (TODO проверить, привести конкретные примеры). Сброс привелегий происходит в методе _drop\_privs()_.

Далее начинается инициализация объектов, которые непосредственно участвуют в получении, обработке и хранении лог-сообщений.

``` cpp
LastLogTimes *times = new LastLogTimes();
```

`LastLogTimes` - это на самом деле список указателей класса `LogTimeEntry`, который объявлен в _repo/system/core/logd/LogTimes.h_:

``` cpp
typedef android::List<LogTimeEntry *> LastLogTimes;
```

Следующим по очереди создается объект `LogBuffer` (_repo/system/core/logd/LogBuffer.h_), в который передается созданный список `times`:

``` cpp
static LogBuffer *logBuf = NULL;
...

logBuf = new LogBuffer(times);
```

Задача `LogBuffer` - получать лог-сообщения (метод `log()`), сохранять их в контейнерах `LogBufferElement` (_repo/system/core/logd/LogBufferElement.h_)  и по запросу записывать передавать сообщения другим адресатам (метод `flushTo()`). Размер буфера лог-сообщений задается в методе `init()`, который вызывается в конструкторе объекта. Откуда мы можем получить размер буфера (в порядке приоритета):

- __"persist.logd.size"__ - это системное свойство, которое может задаваться через Настройки телефона в меню "Свойства разработчика" (метод `writeLogdSizeOption` класса `DevelopmentSettings`, файл _repo/packages/apps/Settings/src/com/android/settings/DevelopmentSettings.java_);
- __"ro.logd.size"__ - это read-only системное свойство, которое устанавливается при сборке прошивки девайса;
- `#define LOG_BUFFER_SIZE (256 * 1024)`;
- `#define LOG_BUFFER_MIN_SIZE (64 * 1024UL)`.

Далее в `main()` создается объект класса `LogReader` (_repo/system/core/logd/LogReader.h_), в который передается указатель на буфер сообщений:

``` cpp
LogReader *reader = new LogReader(logBuf);
if (reader->startListener()) {
    exit(1);
}
```

`LogReader` является наследником класса `SocketListener` (_repo/system/core/include/sysutils/SocketListener.h_). В конструкторе `LogReader` вызывается метод `getLogSocket()`, в котором мы получаем дескриптор сокета с именем __"logdr"__:

``` cpp
int LogReader::getLogSocket() {
    static const char socketName[] = "logdr";
    int sock = android_get_control_socket(socketName);

    if (sock < 0) {
        sock = socket_local_server(socketName,
                                   ANDROID_SOCKET_NAMESPACE_RESERVED,
                                   SOCK_SEQPACKET);
    }

    return sock;
}
```

Этот дескриптор сохраняется в переменной `mSock` в конструкторе `SocketListener()`. При вызове `reader->startListener()` мы попадаем в метод `SocketListener::startListener()`, в котором вызываем функцию [listen][listen_doc]. В качестве параметров передается сохраненный дескриптор сокета `mSock` и значение backlog (максимальная длина очереди ожидающих соединений) равное 4. Затем в отедельном потоке мы запускаем метод `SocketListener::runListener()`, в котором в бесконечном цикле мы начинаем ожидать новые соединения.

``` cpp
// В переменной read_fds будет храниться набор дескрипторов, от которых мы ждем соединение (https://stackoverflow.com/questions/18952564/understanding-fd-set-in-unix-sys-select-h)
fd_set read_fds;
int rc = 0;
int max = -1;

FD_ZERO(&read_fds);
if (mListen) {
    max = mSock;

    // Добавляем дескриптор сокета в read_fds
    FD_SET(mSock, &read_fds);
}
...

// mClients - список указателей на объекты SocketClient (repo/system/core/include/sysutils/SocketClient.h), с помощью которых мы работаем с сокетами (прием и отправка сообщений)
for (it = mClients->begin(); it != mClients->end(); ++it) {
    // Добавляем дескрипторы сокетов в read_fds
    int fd = (*it)->getSocket();
    FD_SET(fd, &read_fds);
    if (fd > max) {
        max = fd;
    }
}

// С помощью select (http://man7.org/linux/man-pages/man2/select.2.html) мониторим дескрипторы и ждем когда к нам придут данные
if ((rc = select(max + 1, &read_fds, NULL, NULL, NULL)) < 0) { ... }
...

if (mListen && FD_ISSET(mSock, &read_fds)) {
    struct sockaddr addr;
    socklen_t alen;
    int c;

    do {
        alen = sizeof(addr);

        // Используем accept (http://man7.org/linux/man-pages/man2/accept.2.html)
        c = accept(mSock, &addr, &alen);
        SLOGV("%s got %d from accept", mSocketName, c);
    } while (c < 0 && errno == EINTR);
    if (c < 0) {
        SLOGE("accept failed (%s)", strerror(errno));
        sleep(1);
        continue;
    }
    ...
    //
    mClients->push_back(new SocketClient(c, true, mUseCmdNum));
    ...
}
```






[va_args_doc]: https://en.wikipedia.org/wiki/Variadic_macro
[token_concat]: https://gcc.gnu.org/onlinedocs/cpp/Concatenation.html
[va_list_doc]: https://en.cppreference.com/w/cpp/utility/variadic/va_list
[va_start_doc]: https://en.cppreference.com/w/cpp/utility/variadic/va_start
[va_end_doc]: https://en.cppreference.com/w/cpp/utility/variadic/va_end
[vsnprintf_doc]: https://en.cppreference.com/w/cpp/io/c/vfprintf
[open_posix_doc]: http://man7.org/linux/man-pages/man2/open.2.html
[errno_doc]: http://man7.org/linux/man-pages/man3/errno.3.html
[signal_doc]: http://man7.org/linux/man-pages/man7/signal.7.html
[socket_doc]: http://man7.org/linux/man-pages/man2/socket.2.html
[unix_socket_doc]: http://man7.org/linux/man-pages/man7/unix.7.html
[open_doc]: http://man7.org/linux/man-pages/man2/open.2.html
[setsockopt_doc]: http://man7.org/linux/man-pages/man2/setsockopt.2.html
[socket7_doc]: http://man7.org/linux/man-pages/man7/socket.7.html
[timeval_doc]: http://pubs.opengroup.org/onlinepubs/7908799/xsh/systime.h.html
[connect-doc]: http://man7.org/linux/man-pages/man2/connect.2.html
[readv_writev_doc]: http://man7.org/linux/man-pages/man2/readv.2.html
[persistent_storage_article]: https://lwn.net/Articles/434821/
[kernel_oops]: https://en.wikipedia.org/wiki/Linux_kernel_oops
[kernel_panic]: https://en.wikipedia.org/wiki/Kernel_panic
[linux_system_request]: https://github.com/torvalds/linux/blob/master/Documentation/admin-guide/sysrq.rst
[listen_doc]: http://man7.org/linux/man-pages/man2/listen.2.html
