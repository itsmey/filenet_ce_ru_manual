# Отношения между объектами. Папки

Здесь будут рассмотрены типы: Containable, Folder, ReferentialContainmentRelationship, Link.

## Интерфейс [Containable](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Containable.html)

Представляет объекты, которые могут храниться в папках. Основные типы, наследующие этот интерфейс:

* [Document](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Document.html)
* [Folder](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Folder.html)
* [CustomObject](https://www.ibm.com/support/knowledgecenter/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/CustomObject.html)

Различия между этими типами можно проиллюстрировать таблицей:

объект | Имеет класс и метаданные |  Имеет права доступа | Версионный | Может иметь эл-ты содержимого | Является контейнером | Поддерживает жизненный цикл
------------ | ------------- | ------------- | ------------- | ------------- | ------------- | -------------
Document|+|+|+|+|-|+
Folder|+|+|-|-|+|-
CustomObject|+|+|-|-|-|-

API Containable предоставляет, восновном, методы для настройки прав доступа и безопасности объекта. К данной теме относится метод 

`ReferentialContainmentRelationshipSet get_Containers()`

Он возвращает коллекцию контейнеров данного объекта, т.е. папок, содержащих данный объект.
