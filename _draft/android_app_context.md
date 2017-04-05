h1. Запуск приложения в заданном контексте

h2. Задача

На телефоне стандартным методом установлено java-приложение (application.apk). По-умолчанию все сторонние приложения (не системные и не от вендора) запускаются с SELinux контекстом untrusted_app. Необходимо, чтобы наше приложение запускалось в специальном контексте.

h2. Решение 1

Расширить контекст untrusted_app, добавив необходимые разрешения в sepolicy. 

Плюсы:
* простота решения
* не нужно сильно модифицировать sepolicy

Минусы:
* все приложения, работающие в контексте untrusted_app, получают расширенный набор разрешений. Это может привести к проблемам с безопасностью.

h2. Решение 2

Создать отдельный контекст (например, trusted_app), дать ему необходимые разрешения и запускать наше приложение в данном контексте.

h4. Создание нового контекста

С помощью sepolicy-inject создаем новый контекст (trusted_app), даем ему те же разрешения, что и у контекста untrusted_app. Плюс добавляем те разрешения, которые необходимы для нашего приложения.
Приведенные ниже разрешения основаны на правилах из файла aosp_5.1.1/external/sepolicy/untrusted_app.te.

<pre><code class="sh">
SEPOLICY_FILE=sepolicy

allow() {
    [ -z "$1" -o -z "$2" -o -z "$3" -o -z "$4" ] && false
    for s in $1;do
        for t in $2;do
            sepolicy-inject -s $s -t $t -c $3 -p $(echo $4|tr ' ' ',') -P ${SEPOLICY_FILE}
        done
    done
}

noaudit() {
    for s in $1;do
        for t in $2;do
            for p in $4;do
                sepolicy-inject -s $s -t $t -c $3 -p $p -P ${SEPOLICY_FILE}
            done
        done
    done
}

TRUSTED_APP=trusted_app

# add non-permissive domain
sepolicy-inject -z ${TRUSTED_APP} -P ${SEPOLICY_FILE}

# add a type_attribute to a domain
sepolicy-inject -s ${TRUSTED_APP} -a domain -P ${SEPOLICY_FILE}
sepolicy-inject -s ${TRUSTED_APP} -a appdomain -P ${SEPOLICY_FILE}
sepolicy-inject -s ${TRUSTED_APP} -a netdomain -P ${SEPOLICY_FILE}
sepolicy-inject -s ${TRUSTED_APP} -a bluetoothdomain -P ${SEPOLICY_FILE}

# type_transitions
sepolicy-inject -s ${TRUSTED_APP} -f tmpfs -c file -t untrusted_app_tmpfs -P sepolicy
sepolicy-inject -s ${TRUSTED_APP} -f devpts -c chr_file -t untrusted_app_devpts -P sepolicy
sepolicy-inject -s untrusted_app -f app_data_file -c process -t ${TRUSTED_APP} -P sepolicy
    
allow ${TRUSTED_APP} untrusted_app_tmpfs file "read write"

# allow working with apps files (like shared libraries)
allow ${TRUSTED_APP} app_data_file file "getattr open read ioctl lock execute execute_no_trans execmod create write setattr unlink";
allow ${TRUSTED_APP} app_data_file dir "create reparent rename rmdir setattr open getattr read search ioctl search write add_name remove_name";

allow ${TRUSTED_APP} tun_device chr_file "getattr open read ioctl lock open append write";

# rules for ASEC
allow ${TRUSTED_APP} asec_apk_file file "getattr open read ioctl lock";
allow ${TRUSTED_APP} asec_public_file file "execute execmod";

# rules for pty
allow ${TRUSTED_APP} untrusted_app_devpts chr_file "open getattr read write ioctl";

# installation via adb
allow ${TRUSTED_APP} shell_data_file file "getattr open read ioctl lock";
allow ${TRUSTED_APP} shell_data_file dir "open getattr read search ioctl";

# allow reads from /data/anr/traces.txt
allow ${TRUSTED_APP} anr_data_file file "getattr open read ioctl lock";

# access /dev/mtp_usb
allow ${TRUSTED_APP} mtp_device chr_file "getattr open read ioctl lock open append write";

# access to /data/media
allow ${TRUSTED_APP} media_rw_data_file dir "create reparent rename rmdir setattr open getattr read search ioctl search write add_name remove_name";
allow ${TRUSTED_APP} media_rw_data_file file "create setattr getattr open read ioctl lock append write link unlink rename";

# write to /cache
allow ${TRUSTED_APP} cache_file dir "create reparent rename rmdir setattr open getattr read search ioctl search write add_name remove_name";
allow ${TRUSTED_APP} cache_file file "create setattr getattr open read ioctl lock append write link unlink rename";

# allow verifier to access staged apks
allow ${TRUSTED_APP} apk_tmp_file dir "open getattr read search ioctl";
allow ${TRUSTED_APP} apk_tmp_file file "getattr open read ioctl lock";
allow ${TRUSTED_APP} apk_private_tmp_file dir "open getattr read search ioctl";
allow ${TRUSTED_APP} apk_private_tmp_file file "getattr open read ioctl lock";

allow ${TRUSTED_APP} ${TRUSTED_APP} process "fork setsched transition sigchld noatsecure siginh rlimitinh"
allow ${TRUSTED_APP} ${TRUSTED_APP} fifo_file "open read write create"

# domain_transitions
allow untrusted_app app_data_file file "getattr open read execute";
allow untrusted_app ${TRUSTED_APP} process "transition";
allow ${TRUSTED_APP} app_data_file file "entrypoint open read execute getattr";
allow ${TRUSTED_APP} untrusted_app process "sigchld";
noaudit untrusted_app ${TRUSTED_APP} process "noatsecure";
allow untrusted_app ${TRUSTED_APP} process "siginh rlimitinh";
</code></pre>

h4. Получение сертификата приложения

Получаем публичный сертификат приложения в формате .pem:

<pre><code class="sh">
cp /path/with/application/
mkdir ./tmp
cd ./tmp
unzip ../application.apk

cd ./META-INF
openssl pkcs7 -inform DER -in *.RSA -out CERT.pem -outform PEM -print_certs
</code></pre>

Открываем получившийся файл CERT.pem и удаляем все строки до "-----BEGIN CERTIFICATE-----" (subject=... , issuer=...).
Далее необходимо преобразовать публичный сертификат из формата base64 в формат base16. Для этого можно воспользоваться кодом из файла _aosp/external/sepolicy/tools/insertkeys.py_ или использовать проект https://github.com/iamantony/pem-key-transformation:

<pre><code class="sh">
git clone https://github.com/iamantony/pem-key-transformation.git
cd ./pem-key-transformation

python3 pem-key-transformation.py /path/with/application/tmp/META-INF/CERT.pem --save_to /path/to/result.txt
</code></pre>

После выполнения комманд в файле _/path/to/result.txt_ будет сохранен искомый ключ.

h4. Модификация mac_permissions.xml

Скачиваем с телефона файл _mac_permissions.xml_. Согласно данным из _aosp_5.1.1/frameworks/base/services/core/java/com/android/server/pm/SELinuxMMAC.java_, этот файлик может находиться в папках:
* /security/current/mac_permissions.xml
* /etc/security/mac_permissions.xml

Этот файл представляет из себя сжатый в одну строчку xml с примерно такой структурой:

<pre>
<signer signature="429347297494" >
    <allow-all/>
    <seinfo value="release"/>
</signer>
</pre>

* signature - публичный сертификат приложения в формате base16
* seinfo - тэг записи

Преобразуем файл в нормальный xml:

<pre><code class="sh">
xmllint --format mac_permissions.xml > mac_permissions.normal.xml
</code></pre>

Добавим в конец файла _mac_permissions.normal.xml_ новую запись (до закрывающего тэга </policy> !) с сертификатом нашего приложения, который мы получили на предыдущем шаге:

<pre>
<signer signature="сертификат нашего приложения в base16" >
    <allow-all/>
    <seinfo value="trusted_app"/>
</signer>
</pre>

Преобразуем _mac_permissions.normal.xml_ в файл с одной строкой _mac_permissions.new.xml_:

<pre><code class="sh">
xmllint --noblanks mac_permissions.normal.xml > mac_permissions.new.xml
</code></pre>


h4. Модификация seapp_contexts

Скачиваем с телефона файл _seapp_contexts_ (лежит в корне файловой системы). Добавляем в конец файла строку:

<pre>
user=_app seinfo=trusted_app name=com.application.name domain=trusted_app type=app_data_file
</pre>

* seinfo - записываем значение тэга seinfo из добавленной записи в _mac_permissions.new.xml_
* name - имя приложения (когда оно запущено системой)
* domain - контекст (домен), в котором приложение должно быть запущено
* type - контекст файла приложения (по умолчанию app_data_file)

h4. Применяем изменения на устройстве

Заменяем в _boot.img_ оригинальный файл _sepolicy_ и _seapp_contexts_ на модифицированные. Прошиваем телефон новым _boot.img_.
Заменяем на устройстве файл _mac_permissions.xml_ на модифицированный _mac_permissions.new.xml_.
Перезагружаемся.
Устанавливаем наше приложение на устройство.

Далее запускаем logcat и записываем лог в файл. Запускаем наше приложение на устройство. Проверяем, что приложение запустилось в нужном контексте (ps -Z).
Вполне возможно, что в логе появятся записи "avc: denied", связанные с нашим приложением. Добавляем необходимые разрешения в sepolicy и снова тестируем работу приложения.

h2. Полезная информация:

* Exploring SE for Android - Confer
* https://boundarydevices.com/android-security-part-1-application-signatures-permissions/
* https://selinuxproject.org/page/NB_SEforAndroid_2


