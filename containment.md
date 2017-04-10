# Отношения между объектами. Папки

Здесь пойдёт речь об отношениях типа **хранение** (containment). Это отношение устанавливается между двумя объектами:

1. *Контейнер (хранящий объект, container)*. В CE это папка, т.е. объект Folder
2. *Хранимый объект (containee)*. В CE это объект Containable

Один контейнер может хранить множество объектов. Хранимый объект, если это не папка, может храниться в нескольких контейнерах. Папка, как правило, хранится только в своей родительской папке.

На вершине иерархии находится корневая папка (root folder, "/"). Эта папка не имеет родительской папки. Получить её можно методом get_RootFolder() хранилища или обычным способом, через фабричные методы. Таким образом, у каждой папки в CE есть единственный путь, начинающийся с корневой папки (абсолютный путь).

Папки, чья родительская папка - корневая, называются папками верхнего уровня (top folders). Коллекцию этих папок возвращает метод хранилища get_TopFolders().

## Containable

Тип [Containable](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Containable.html) представляет объекты, которые могут храниться в папках. API Containable предоставляет, в основном, методы для настройки прав доступа и безопасности объекта. Основные типы, наследующие этот интерфейс:

* [Document](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Document.html)
* [Folder](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Folder.html)
* [CustomObject](https://www.ibm.com/support/knowledgecenter/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/CustomObject.html)

Различия между этими типами можно проиллюстрировать таблицей:

объект | Имеет класс и метаданные |  Имеет права доступа | Версионный | Может иметь эл-ты содержимого | Является контейнером
------------ | ------------- | ------------- | ------------- | ------------- | -------------
Document|+|+|+|+|-
Folder|+|+|-|-|+
CustomObject|+|+|-|-|-

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

Важно! Методы get_Containers() и get_Containees() возвращают коллекции не самих хранящих/хранимых объектов, а их размещений.

## Размещения

Каждая пара "контейнер - хранимый объект" представлена в СЕ специальным объектом - **размещением** (Containment relationship). Размещение является "посредником" между контейнером и хранимым объектом: сами они не обладают информацией, кто где размещён. Иерархию размещений в CE можно упрощённо изобразить так:

* [Relationship](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Relationship.html). Этот тип представляет некое бинарное отношение и вводит понятия Head и Tail - объекты, которые участвуют в отношении
* ContainmentRelationship. Неинстанциируемый тип
* [ReferentialContainmentRelationship](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/ReferentialContainmentRelationship.html). Статическое размещение. Вводит методы get_ContainmentName() и set_ContainmentName(). Имя размещения (containment name) должно быть уникальным в пределах папки
* [DynamicReferentialContainmentRelationship](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/DynamicReferentialContainmentRelationship.html). Динамическое размещение

отношение | между кем |  тип Head | тип Tail 
------------ | ------------- | ------------- | -------------
ReferentialContainmentRelationship|между папкой и CustomObject/определённой версией документа|Containable|Folder
DynamicReferentialContainmentRelationship|между папкой и текущей версией документа|Versionable|Folder

Таким образом, Head указывает на хранимый объект, а Tail - на контейнер.

Существует 2 способа создания размещения:

1. Явно - через фабричный метод Factory.ReferentialContainmentRelationship.createInstance(), и далее вручную назначить Head и Tail
2. Неявно - через метод Folder.file(). Этот метод создаёт статическое размещение для CustomObject и динамическое - для Document

## Примеры кода

### Создание папки

Создать папку можно несколькими способами:

#### Через фабричный метод

```java
// если вы используете Folder.createInstance(), нужно явно задать значения для свойств Parent и FolderName
Folder myFolder = Factory.Folder.createInstance(os, null);

myFolder.set_Parent(os.get_RootFolder());
myFolder.set_FolderName("Loans");
myFolder.save(RefreshMode.NO_REFRESH);
```

#### Через метод createSubFolder() родительской папки

```java
// получаем родительскую папку
String parentFolderPath = "/Loans";
Folder parentFolder = Factory.Folder.getInstance(os, null, parentFolderPath);

// при таком создании необходимо указать только FolderName как параметр метода createSubFolders()
Folder newFolder = parentFolder.createSubFolder("MyLoans");
newFolder.save(RefreshMode.NO_REFRESH);
```

### Получение папки

Получить папки можно также несколькими способами:

* через фабричные методы fetchInstance() или getInstance()
* через метод get_Parent() - для получения родительской папки
* сналача получить объект ReferentialContainmentRelationship через фабричный метод по ID объекта или абсолютному пути, затем получить значение свойства Tail
* ObjectStore.get_RootFolder() - для получения коренвой папки хранилища
* ObjectStore.get_TopFolders() - для получения кколлекции папок верхнего уровня
* Folder.get_SubFolders() - получить коллекцию подпапок некоторой папки
* Versionable.get_FoldersFiledIn() - получить коллекцию папок, размещающих документ
* Containable.get_Containers() - получить размещения для контейнеров объекта
* Folder.get_Containees() - получить размещения для хранимых объектов некоторой папки

### Получение содержимого папки

* get_Containees(), и далее для каждого ReferentialContainmentRelationship дёргать свойство Head
* get_ContainedDocuments() - возвращает коллекцию документов, хранящихся в папке
* получить ReferentialContainmentRelationship через фабричный метод, далее - свойство Head

### Создание размещения

#### Явное

Рассмотрим создание динамического размещения, в котором Head имеет тип Versionable. Статическое размещение создаётся схожим образом, но Head в нём имеет тип Containable. 

Динамическое размещение работает не с документом, а с набором версий (version series). Это позволяет следить за тем, чтобы в папке была всегда размещена текущая версия из набора.

```java
String parentFolderPath = "/Loans";

// создаём документ
Document doc = Factory.Document.createInstance(os, null);
doc.save(RefreshMode.NO_REFRESH); 

// получаем папку-контейнер
Folder container = Factory.Folder.getInstance(os, null, parentFolderPath); 

// создаём инстанс DynamicReferentialContainmentRelationship
DynamicReferentialContainmentRelationship drcr = 
    Factory.DynamicReferentialContainmentRelationship.createInstance(os, null); 

// назначаем свойства Head и Tail
drcr.set_Head(doc); 
drcr.set_Tail(container); 
drcr.save(RefreshMode.NO_REFRESH);
```

#### Неявное

Рассмотрим параметры метода file():

`ReferentialContainmentRelationship file(IndependentlyPersistableObject containee,AutoUniqueName autoUniqueName,java.lang.String containmentName,DefineSecurityParentage defineSecurityParentage)`

* containee - хранимый объект
* autoUniqueName - параметр, определяющий, нужно ли автоматически разрешать коллизии имён:
  - AUTO_UNIQUE - нужно
  - NOT_AUTO_UNIQUE - не нужно
* defineSecurityParentage - определяет, наследовать ли параметры безопасности хранимого объекта у данного контейнера:
  - DEFINE_SECURITY_PARENTAGE - наследовать. Если указано это значение, свойству SecurityFolder объекта автоматически присваивается ссылка на контейнер
  - DO_NOT_DEFINE_SECURITY_PARENTAGE - не наследовать
  
```java
String folderPath = "/ExistingLoans/MyLoans";
String newFolderPath = "/NewLoans/MyNewLoans";
String newContainmentName = "Loan1";

Document myDoc = Factory.Document.fetchInstance(os, folderPath + "/MyLoan_1", null);
Folder newFolder = Factory.Folder.fetchInstance(os, newFolderPath, null);

// т.к. хранимым объектом является документ, для коррекного создания динамического размещения нужно
// привести значение к типу DynamicReferentialContainmentRelationship
try {
   DynamicReferentialContainmentRelationship drcr = 
      (DynamicReferentialContainmentRelationship) newFolder.file(
         myDoc, AutoUniqueName.NOT_AUTO_UNIQUE, newContainmentName, DefineSecurityParentage.DEFINE_SECURITY_PARENTAGE);
   drcr.save(RefreshMode.NO_REFRESH);
}
catch (Exception ex) 
{   
  if (ex instanceof EngineRuntimeException) {
    EngineRuntimeException fnEx = (EngineRuntimeException) ex;
        // обработка исключения E_NOT_UNIQUE - нарушение уникальности containment name
        if (fnEx.getExceptionCode().equals(ExceptionCode.E_NOT_UNIQUE) ) {
           System.out.println("Exception: " + ex.getMessage() + "\n" +
              "The name " + "\"" + newContainmentName + "\"" + " is already used.\n" + 
              "Please file the document with a unique name.");
        }
        else
           System.out.println("Exception: " + ex.getMessage());
   }
   else
      // исключение Java
      System.out.println("Exception: " + ex.getMessage());
}
```

**Важно**. Если требуется создать размещение конкретной версии документа в папке, используйте явный способ, т.к. это невозможно сделать через метод file(). Метод file() автоматически создаёт динамическое размещение, если в качестве containee в него передаётся инстанс Document.

## Ссылки

Ссылка ([Link](https://www.ibm.com/support/knowledgecenter/SSGLW6_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Link.html)) представляет произвольное отношение между двумя объектами CE. То же, что и Relationship, но объекты типа Link можно инстанцировать ([Factory.Link](https://www.ibm.com/support/knowledgecenter/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/Factory.Link.html)).

Объекты-ссылки имеют свойства Head и Tail типа IndependentObject и предоставляют геттеры и сеттеры для этих свойств.

**Важно**. Если объект, на который указывает Head или Tail, был удалён, ссылка остаётся без изменений, т.е. указывает на несуществующий объект. Это может привести к ошибкам.

## Дополнительная информация

* [Containment concepts](https://www.ibm.com/support/knowledgecenter/SSGLW6_5.2.0/com.ibm.p8.ce.dev.ce.doc/containment_concepts.htm)
* [Containment procedures](https://www.ibm.com/support/knowledgecenter/SSGLW6_5.2.0/com.ibm.p8.ce.dev.ce.doc/containment_procedures.htm)
