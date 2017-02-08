# Соединение

Здесь будут рассмотрены процесс соединения с сервером CE, получение объекта хранилища данных (ObjectStore), а также типы Connection, UserContext, Domain, ObjectStore, EngineRuntimeException.

## Исходные данные

* URI для подключения к CE - String. Пример: "http://172.19.215.15:9080/wsi/FNCEWS40MTOM/"
*	имя пользователя - String
*	пароль - String
*	имя Object Store - String

Вид URI для подключения к CE зависит от протокола. Для подключения по http (Web Services) изспользуется URI вида http://server:port/wsi/FNCEWS40MTOM/

## Общий порядок действий при работе с хранилищем объектов

1. Получить объект Connection
2. Получить объект UserContext
3. Создать объект javax.security.auth.Subject
4. Сделать полученный Subject активным (pushSubject)
5. Получить объект Domain
6. Получить объект ObjectStore
7. Выполнить необходимую работу с данными (получение, изменение, удаление...)
8. Вернуть предыдущий Subject (pushSubject)

Важно: каждому вызову pushSubject() должен соответствовать вызов popSubject().

## Пример кода

Порядок действий иллюстрирует пример кода. 
Метод popSubject() необходимо вызывать в блоке finally, чтобы гарантировать его исполнение.

```java
Connection conn = Factory.Connection.getConnection(uri);
userContext = UserContext.get();
subject = UserContext.createSubject(conn, login, password, null);

UserContext.pushSubject(subject);

try {
    Domain domain = Factory.Domain.getInstance(conn, null);

    objectStore = Factory.ObjectStore.fetchInstance(domain,
            objectStoreName, null);
            
    //работа с данными

} catch (EngineRuntimeException ex) {
    throw new Exception(ex.getExceptionCode().toString());
} finally {
    UserContext.popSubject();
}
```

## Connection

Интерфейс [Connection](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Connection.html) представляет логическое соединение с CE. Для получения объекта Connection используется один из двух методов фабричного класса Factory.Connection:

`public static Connection getConnection( java.lang.String uri)`
`public static Connection getConnection( java.lang.String uri, ConfigurationParameters parameters)`

Первый метод создаёт соединение с параметрами по умолчанию.

## UserContext

Класс [UserContext](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/util/UserContext.html) используется для установки локали и параметров аутентификации. Эта информация подставляется в запросы к серверу CE, которые отправляет текущий поток (т.е. поток, с которым связан объект UserContext). 
Для каждого потока имеется свой стек из объектов Subject (JAAS). Также предоставляется возможность работать с внешним (ambient) JAAS-субъектом. В этом случае, внутренний стек игнорируется.

Основные методы:

`static javax.security.auth.Subject createSubject(Connection conn, java.lang.String user, java.lang.String password, java.lang.String optionalJAASStanzaName)`
 Создать субъект средствами JAAS. Последний параметр можно указать null, в этом случае stanza будет «FileNetP8»

`static UserContext get()`
Получить объект UserContext, связанный с текущим потоком
	
`javax.security.auth.Subject getSubject()`
Получить объект Subject, связанный с текущим потоком

`javax.security.auth.Subject popSubject()`
Извлечь из стека текущий активный SUbject и сделать активным предыдущий SUbject

`void pushSubject(javax.security.auth.Subject sub)`
Поместить sub на вершину стека

`java.util.Locale getLocale()`
Получить текущую локаль для данного контекста

`void setLocale(java.util.Locale locale)`
Установить локаль в данном контексте




