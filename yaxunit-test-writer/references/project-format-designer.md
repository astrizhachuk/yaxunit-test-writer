# Формат проекта: выгрузка Конфигуратора

Файлы метаданных, их раскладка и регистрация для исходников в формате выгрузки Конфигуратора. BSL-код модулей от формата не зависит.

## Признаки формата

В корне выгрузки лежит `Configuration.xml`.

## Расположение проектов

- Тестовое расширение `Тесты` — каталог `test/` (корень выгрузки расширения с `Configuration.xml`).
- Основная конфигурация — каталог `src/`.

## Раскладка файлов

Метаданные объекта — файл `[Тип]/[ИмяОбъекта].xml`, модули — в каталоге `[Тип]/[ИмяОбъекта]/Ext/`. Одинаково для `test/` и `src/`.

| Тип объекта                        | Каталог модулей                        | Имя файла             |
| ---------------------------------- | -------------------------------------- | --------------------- |
| Общий модуль                       | `test/CommonModules/ИмяМодуля/Ext/`    | `Module.bsl`          |
| Модуль менеджера документа         | `test/Documents/ИмяДок/Ext/`           | `ManagerModule.bsl`   |
| Модуль объекта документа           | `test/Documents/ИмяДок/Ext/`           | `ObjectModule.bsl`    |
| Модуль менеджера справочника       | `test/Catalogs/ИмяСпр/Ext/`            | `ManagerModule.bsl`   |
| Модуль объекта справочника         | `test/Catalogs/ИмяСпр/Ext/`            | `ObjectModule.bsl`    |
| Модуль менеджера регистра сведений | `test/InformationRegisters/ИмяРС/Ext/` | `ManagerModule.bsl`   |
| Модуль набора записей регистра     | `test/InformationRegisters/ИмяРС/Ext/` | `RecordSetModule.bsl` |

Объект есть в расширении (новый или заимствованный), если есть файл `test/[Тип]/[ИмяОбъекта].xml`.

## Новый тестовый модуль

```
test/
  CommonModules/
    [ИмяТестовогоМодуля].xml    ← описание модуля (метаданные)
    [ИмяТестовогоМодуля]/
      Ext/
        Module.bsl              ← код теста
  Configuration.xml             ← регистрация всех модулей
```

**Перед созданием** — проверь, существует ли уже модуль с таким именем (`test/CommonModules/[ИмяТестовогоМодуля].xml`). Если файл есть, модуль уже зарегистрирован: не трогай `.xml` и `Configuration.xml`, работай только с `Module.bsl`.

При создании нового модуля:
1. Создай `test/CommonModules/[ИмяТестовогоМодуля]/Ext/Module.bsl`
2. Зарегистрируй модуль в `test/Configuration.xml` (раздел «Регистрация в `Configuration.xml`»).
3. Создай файл `test/CommonModules/[ИмяТестовогоМодуля].xml`. `{uuidgen}` — уникальный UUID объекта метаданных; каждый модуль должен иметь свой UUID (генерация — раздел «Генерация UUID» навыка).

   Значения `ClientManagedApplication` и `ClientOrdinaryApplication` — по правилу контекстов компиляции из навыка:

   - **ОМ_ (общий модуль)** — прочитай `src/CommonModules/[ИмяТестируемогоМодуля].xml` и скопируй значения `ClientManagedApplication` и `ClientOrdinaryApplication` оттуда.
   - **Остальные типы (Док_, РС_, Спр_ и др.)** — всегда `false`/`false` (только сервер).

   Если средства создания объектов метаданных в среде нет (см. навык), создай файл инструментом Write напрямую — никаких дополнительных скриптов не нужно. Отступы — **табуляция**.

   Прочитай шаблон из `templates/{версия-платформы}/common-module.xml` и подставь значения: `{uuidgen}` → новый UUID, `[ИмяТестовогоМодуля]` → имя модуля, `???` → значения из правил выше.

## Заимствование объекта для перехватчика

Перехватчик — это **adopted**-расширение существующего объекта основной конфигурации. Действия зависят от состояния объекта в расширении:

- **Объект не заимствован** (нет `test/[Тип]/[ИмяОбъекта].xml`) — создай XML-метаданные (ниже) и зарегистрируй объект (раздел «Регистрация в `Configuration.xml`»).
- **Объект заимствован, нужного модуля нет** (например, есть `ObjectModule.bsl`, а нужен `ManagerModule.bsl`) — в существующем `test/[Тип]/[ИмяОбъекта].xml` добавь в конец `<InternalInfo>` блок `<xr:PropertyState>` для нового модуля (правило — в п. 2 ниже). UUID не меняй, `Configuration.xml` не трогай.
- **Модуль есть** — метаданные и регистрацию не трогай.

### Общий принцип построения XML

Структура XML перехватчика строится на основе `src/[Тип]/[ИмяОбъекта].xml`:

1. **Корневой тег** (`<CommonModule>`, `<Document>`, `<Catalog>`, `<InformationRegister>`) — новый UUID.
2. **`<InternalInfo>`** — скопировать блок `<xr:GeneratedType>` из src, **заменив все UUID (TypeId и ValueId) на новые**; структура (имена, категории, порядок) остаётся точно такой же. Добавить в конец блока один или несколько блоков `<xr:PropertyState>` — по одному на каждый расширяемый модуль:
   - Для общего модуля: `<xr:Property>Module</xr:Property>`
   - Для модуля менеджера: `<xr:Property>ManagerModule</xr:Property>`
   - Для модуля объекта: `<xr:Property>ObjectModule</xr:Property>`
   - Для набора записей регистра сведений: `<xr:Property>RecordSetModule</xr:Property>`

   Если расширение перекрывает несколько модулей, добавь соответствующие блоки `<xr:PropertyState>` подряд.
3. **`<Properties>`** — минимальный набор: `<Name>`, `<Comment/>`, `<ObjectBelonging>Adopted</ObjectBelonging>`, `<ExtendedConfigurationObject>{uuid корневого тега из src}</ExtendedConfigurationObject>`. Для регистров сведений дополнительно скопировать `<InformationRegisterPeriodicity>` и `<WriteMode>` из src.
4. Общие модули не имеют `<xr:GeneratedType>` ни в src, ни в test — `<InternalInfo>` содержит только `<xr:PropertyState>`.

UUID — раздел «Генерация UUID» навыка (для документа или справочника обычно нужно 11, для регистра 15).

### Шаблон для общего модуля

Нужен **1 UUID** (корневой тег). Свойства `ClientManagedApplication`, `ExternalConnection`, `ClientOrdinaryApplication`, `ServerCall` скопировать из `src/CommonModules/[ИмяМодуля].xml`.

Прочитай шаблон из `templates/{версия-платформы}/common-module-adopted.xml` и подставь значения: `{новый-uuid}` → новый UUID, `[ИмяМодуля]` → имя модуля, `{uuid-из-src-xml}` → UUID корневого тега из `src`, `???` → значения из src.

## Регистрация в `Configuration.xml`

Добавь в `<ChildObjects>` файла `test/Configuration.xml` в **алфавитном порядке** среди тегов того же типа тег, соответствующий типу объекта:

```xml
<ChildObjects>
    ...
    <CommonModule>ОМ_Колеровка</CommonModule>
    <CommonModule>[ИмяТестовогоМодуля]</CommonModule>
    <CommonModule>ОМ_МерныеТоварыВызовСервера</CommonModule>
    ...
</ChildObjects>
```

```xml
<CommonModule>ИмяМодуля</CommonModule>
<!-- или -->
<Document>ИмяДокумента</Document>
<!-- или -->
<Catalog>ИмяСправочника</Catalog>
<!-- или -->
<InformationRegister>ИмяРегистра</InformationRegister>
```

Adopted-объекты регистрируются так же, как новые — просто по имени, без специальных атрибутов.

## Чек-лист формата

- [ ] Файл создан в `test/CommonModules/[Имя]/Ext/Module.bsl`
- [ ] Создан файл метаданных `test/CommonModules/[Имя].xml` с уникальным UUID
- [ ] Модуль зарегистрирован в `test/Configuration.xml` (в `<ChildObjects>`, алфавитный порядок)
- [ ] Перехватчик (если объект заимствован впервые) зарегистрирован в `test/Configuration.xml`; если к уже заимствованному объекту добавлен модуль — в его `.xml` добавлен `<xr:PropertyState>`
