# Отношения между объектами. Папки

Интерфейс [Containable](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Containable.html) представляет объекты, которые могут храниться в папках. API Containable предоставляет, в основном, методы для настройки прав доступа и безопасности объекта. Основные типы, наследующие этот интерфейс:

* [Document](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Document.html)
* [Folder](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Folder.html)
* [CustomObject](https://www.ibm.com/support/knowledgecenter/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/CustomObject.html)

Различия между этими типами можно проиллюстрировать таблицей:

объект | Имеет класс и метаданные |  Имеет права доступа | Версионный | Может иметь эл-ты содержимого | Является контейнером | Поддерживает жизненный цикл
------------ | ------------- | ------------- | ------------- | ------------- | ------------- | -------------
Document|+|+|+|+|-|+
Folder|+|+|-|-|+|-
CustomObject|+|+|-|-|-|-

Рассмотрим методы, относящиеся к даной теме.

API | Метод | Что делает
------------ | ------------- | -------------
Containable | `ReferentialContainmentRelationshipSet get_Containers()` | Возвращает коллекцию контейнеров
Versionable | `FolderSet get_FoldersFiledIn()` | Возвращает коллекцию папок, которые содержат документ
Folder | `ReferentialContainmentRelationship file(IndependentlyPersistableObject containee,AutoUniqueName autoUniqueName,java.lang.String containmentName,DefineSecurityParentage defineSecurityParentage)` | Размещает объект в папке
Folder | `ReferentialContainmentRelationship unfile(IndependentlyPersistableObject containee)` | Удаляет размещение объекта containee в данной папке
Folder | `ReferentialContainmentRelationship unfile(java.lang.String containmentName)` | Удаляет размещение containmentName в данной папке
Folder | `Folder createSubFolder(java.lang.String name)` | Создаёт и возвращает подпапку
Folder | `FolderSet get_SubFolders()` | Возвращает коллекцию подпапок
Folder | `void move(Folder targetFolder)` | Перемещает папку со всем содержимым в папку targetFolder
Folder | `DocumentSet get_ContainedDocuments()` | Возвращает коллекцию документов, содержащихся в папке (не в подпапках)
Folder | `ReferentialContainmentRelationshipSet get_Containees()` | Возвращает коллекцию хранимых оъектов
Folder | `Folder get_Parent()` | Возвращает родительскую папку 
Folder | `void set_Parent(Folder value)` | Меняет родительскую папку
Folder | `java.lang.String get_PathName()` | Возвращает абсолютный путь к данной папке

