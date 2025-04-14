# Автоматизация верификации и развертывания NuGet пакета с помощью GitHub actions

## Цель
На практическом примере настроить CI/DI [GitHub actions](https://github.com/features/actions) для валидации и развертывания NuGet пакета, начиная с минимально рабочего конфигурационного `yaml` файла и постепенно усовершенствовать его до полной автоматизации запланированных требований.

## Table of Contents
* [Окружение, процессы и постановка задачи](#)
* [Создание базового рабочего процесса](#)

## Окружение, процессы и постановка задачи

Предположим, мы пишем некую библиотеку, используя стек `C#/.NET` и планируем её в дальнейшем разместить в общий доступ в виде `NuGet` пакета.
На данном этапе у нас есть `.NET` решение (solution) в локальном `Git` репозитории. В качестве удаленного репозитория используем `GitHub` сервис.

Настал этап первого развертывания (deploy), который проведем вручную, чтобы понимать те процессы, которые в последствии будем автоматизировать.
Весь процесс можно условно разделить на две части. Первая часть - это разовая предварительная конфигурация окружения и проекта, вторая - непосредственно ручная рутина для каждой публикации библиотеки.

### Первоночальная разовая конфигурация

1. В качестве хоста `NuGet` пакетов будем использовать [nuget.org](https://www.nuget.org) сервис. Для этого [создадим аккаунт](https://learn.microsoft.com/en-us/nuget/nuget-org/individual-accounts#add-a-new-individual-account), если его еще нет
2. [Генерируем в аккаунте API ключ](https://learn.microsoft.com/en-us/nuget/nuget-org/publish-a-package#create-an-api-key), который нам будет необходим для последующей публикации библиотеки
3. Добавим необходмые [метаданные](https://learn.microsoft.com/en-us/nuget/create-packages/package-authoring-best-practices) в `<PropertyGroup>` и `<ItemGroup>` конфигурационного файла проекта (`*.csproj`) для публикации
```xml
<PropertyGroup>
   <!-- Уникальный индентификатор NuGet пакета -->
   <PackageId>AHSW.EventLog</PackageId>
   <Version>1.9.1</Version>
   <Authors>Alex Holenko</Authors>
   <!-- Информация о лицензии -->
   <PackageLicenseExpression>MIT</PackageLicenseExpression>
   <Title>Краткое описание пакета</Title>
   <Description>Развернутое описание пакета</Description>
   <PackageIcon>logo.png</PackageIcon>
   <PackageReadmeFile>README.md</PackageReadmeFile>
   <!-- Ссылка на репозиторий с исходным кодом -->
   <RepositoryUrl>https://github.com/software-developer/useful-library</RepositoryUrl>
   <!-- Теги описывающие проект для индексации и поиска -->
   <PackageTags>dotnet, useful, lib, etc</PackageTags>
   <GenerateDocumentationFile>True</GenerateDocumentationFile>
</PropertyGroup>

<ItemGroup>
   <!-- Добавление необходмых ссылок для ProprtyGroup элементов -->
   <None Include="..\..\images\logo.png" Pack="true" PackagePath="\"/>
   <None Include="..\..\LICENSE" Pack="true" PackagePath="LICENSE"/>
   <None Include="..\..\README.md" Pack="true" PackagePath="\"/>
</ItemGroup>
```

### Этапы ручного развертывания

1. Запуск юнит тестов
2. Инкрементация [версии](https://learn.microsoft.com/en-us/nuget/create-packages/package-authoring-best-practices#package-version) в проектном конфигурацинном файле: `<Version>1.0.1</Version>`
3. Слияние комитов в мастер ветку и добавление `Git` тега с релизной версией на комит
4. Сборка релизной версии библиотеки: `dotnet pack --configuration Release`
5. Публикации релизной версию на `nuget.org` для общего доступа: `dotnet nuget push {NUGET_NAME_WITH_VERSION} --api-key {API_KEY} --source https://api.nuget.org/v3/index.json`
6. Создание [релиза](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases) в `GitHub` репозитории с описанием изменений в текущей версии для пользователей 

После внесения изменений в нашу библиотеку необходимо раз за разом выполнять перечисленные выше 6 шагов для доставки новой версии пакета конечным пользователем. Возьмем эти пункты в качетсве требований и автоматизируем процесс.

## Создание базового рабочего процесса

### Кратко о том, что такое `GitHub actions` и о структуре `yaml` файла

`GitHub actions` это инструменты CI/CD для автоматизации рабочих процессов при создании и публикации программного обеспечения. Под рабочими процессами подразумевается управление ветками разработчиков в процессе создания пулриквестов, код ревью и слияния, а так же сборки, тестирования и публикации конечного результата.

Довольно множество операций можно рассматривать как повторяющиеся и рутинные, которые можно автоматизировать и при необходимости добавлять гибкость с помощью параметров или условных конструкций. Другими словами, автоматизировать процесс путем написание скрипта в `yaml` файле, который будет интерпретироваться `GitHUb` сервисов как набор инструкций автоматизации.

`Yaml` файл включает в себя следующие инструкции:
1. Триггеры запуска автоматизации (комит в определенную ветку, пул риквест)
2. Окружение в котором будут выполнятся команды (Тип и версия ОС, контейнер)
3. Описание исполняемых команд. Команды рассматриваются как `steps` в контексте одной `job`. Рабочий процесс `workflow` может содержать несколько `jobs`

Более подробно о том, что из себя представляют `GitHub Actions` можно почитать на [официальном сайте](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions).

Документацию по написанию рабочих процессов можно найти [здесь](https://docs.github.com/en/actions/writing-workflows).

### MVP

Для создания минимально работающей версии автоматизации нам 



## 4. Чтобы что-то задеплоить, надо иметь собранный пакет. Начальный пайплайн должен состоять минимум из двух джоб

## 5. Так как мы что-то релизим, то добавим джобу для создания релиз ноты на гитхабе

## 6. Создание тега на комите релиза с версией ньюгета

## 7. Добавление проверки версии сборки

## 8. Перед релизом было бы хорошо убедиться, что юнит тесты успешно выполняются и финальный результат

## Вывод: мы имеем настроенный энвайромент со следущим форкфлоу

done----
[How to create github actions workflow for the handling nuget package development and delivery](EN.md)
