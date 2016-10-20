# Принудительная установка языка в ALfresco 5.0 и 5.1
### Содержание
**[Описание проблемы](#problem-overview)**  
**[Настройки Nginx](#nginx-setup)**  
**[Настройки Apache](#apache-setup)**  
### Описание проблемы<a name="problem-overview"></a>
В 5 версии Альфреско была убрана возможность пользователю самостоятельно установить язык в веб интерфейсе.

Теперь он определяется с помощью запроса, посылаемого от клиента к серверу, с помощью заголовка __Accept-Language__.
Если объяснять чуточку подробнее, то в запросе, посылаемым клиентом серверу, есть примерно такая строка:
```
Accept-Language: en-US
```
Это означает, что у клиента стоит английский язык (США). Сервер веб интерфейса Alfresco, получая запрос клиента, смотрит на данный заголовок запроса, и в зависимости от параметра данного заголовка выбирает язык, на котором  будет отображен интерфейс. И вроде бы это достаточно удобное решение - устанавливать язык Alfresco в соотвествии с параметрами рабочей станции клиента. 

Однако, есть нюанс. Имеем операционную систему  __Windows 8.1__ на русском языке  с установленными браузерами __Google Chrome 53__ и __Internet Explorer 8 и 11__ версий. Заголовок, который отправляет __Google Chrome 53__  имеет следующий вид:
```
Accept-Language ru,en-US;q=0.8,en;q=0.6\r\n
```
А вот __Internet Explorer 8 и 11__ при тех же самых условиях отправляет заголовок
```
Accept-Language: en-US,en;q=0.7,ru;q=0.3\r\n
```
Как мы видим, __Internet Explorer 8 и 11__, отправляет заголовок, в котором первой стоит английская локаль. Дело в том, что данный браузер использует языковые настройки операционной системы Windows, в которой первым по приоритету установлен английский язык.  И, к сожалению,  у пользователей, использующих данный браузер, Alfresco будет отображена на английском языке. К сожалению при некоторых условиях, таких как невозможность отказа от __Internet Explorer__, большое количество рабочих станций, и невозможность автоматически изменить всем языковые настройки с помощью групповой политикой, создает некоторые трудности в унифицировании используемого языка. 

С данной проблемой можно справится, изменив некоторые настройки прокси сервера (надеемся, что вы его используете), а именно заставив прокси сервер изменять заголовок в запросе, поступившим от клиента.
### Настройки Nginx<a name="nginx-setup"></a>
Для данного веб сервера, в файле конфигурации необходимо дописать следующую строку в блоке __location__:
```
proxy_set_header        Accept-Language ru;
```
Перезапустите сервер командой
```
service nginx restart
```
### Настройки Apache<a name="apache-setup"></a>
Во первых, вам необходимо включить модуль _headers_ командой
```
a2enmod headers
```
Затем, откройте файл конфигурации виртуального хоста и добавьте следующую строку:
```
RequestHeader append Accept-Language "ru"
```
Перезапустите сервер командой:
```
service apache2 restart
```
Готово! теперь клиенты, независимо от настроек своей рабочей станции будут видеть интерфейс Alfresco на русском языке.