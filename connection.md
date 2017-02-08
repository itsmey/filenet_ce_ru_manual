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

