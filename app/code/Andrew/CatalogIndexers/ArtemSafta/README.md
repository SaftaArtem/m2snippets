# Индексация в М2

Как мы знаем данные о сущностях в м2 сохранены в таблицах базы данных. В базе данных присутствуют сложные взаимосвязи
сущностей. Для отображения некоторых страниц необходимо получать данные с помощью сложных и множественных запросов в
базу данных Это занимает очень много времени. Для оптимизации и ускорения этого процесса была внедрена индексация.
Индексация в м2 трансформирует такие данные как атрибуты продуктов взаимоотношения продукты и категорий для ускорения
отображения магазинов. После того как данные изменились (например изменили количество товара на складе, цена на товар
изменилась)
Необходимо выполнить индексацию для того чтобы данные не только обновились, но и стали доступны на фронте. Для
оптимизации отображения вебсайта м2 аккумулирует данные в специальные таблицы используя индексы.

Например для страницы категории:
без индексации м2 пришлось бы подсчитывать цену для каждого продукта на лету, с учетом cart price rule, скидки, tier
price.... Загрузка цены для продукта заняла бы много времени, как возможный результат клиент бы не совершил покупку

Терминология индексации

- `Dictionary` - оригинальные данные введенные в систему.
- `Index` - представление оригинальных данных для оптимизированного чтения и поиска. Индексы содержат результат
  объединения данных из нескольких таблиц и расчетов Индексные данные могут воссозданы из словаря используя текущий
  алгоритм.
- `Indexer` - Объект, который создает индекс.

# Реализация Индексации в Magento 2

Следующие компоненты вовлечены в процесс индексации:

![Alt text](./components.png?raw=true "Priority")

Для работы с актуальными данными необходимо с какой-то периодичностью обновлять данные. В м2 есть три способа сделать
это:

- `Scheduled updates` обновление по крону.
- `Update on save` после сохранения сущности актуальные данные не только сохраняются в основную таблицу, но и попадают
  еще в индексную
- `Manual` обновление вручную.

Каждый индекс может выполнять следующие типы операций индексации:

- Full полный реиндекс, который означает перестройку всех связанных с индексом таблиц. Полный реиндекс может быть вызван
  множеством причин, включая создание нового магазина или новой группы кастомеров. В любое время вы можете выполнить
  полный реиндекс с помощью терминала.
- Partial Частичный реиндекс, который означает перестройку таблиц связанных с изменением (например изменение атрибута
  продукта или изменение цены).

Логика выполнения операции индексации:

![Alt text](./logic.png?raw=true "Priority")

Рассмотрим команда в терминале связанные с реиндексом

- indexer:info - отображает id индексера и название
- indexer:reindex - выполняет полный реиндекс все индексеров
- indexer:reset - если какой то индексер завис то можно его инвалидировать
- indexer:set-mode - можно установить мод индексера
- indexer:show-mode - отображает мод индексера
- indexer:status - можно получить статус индексеров
- indexer:show-dimensions-mode - отображает какой мод настроен
- indexer:set-dimensions-mode - позволяет настроить параллелизацию индексеров. Параметры:
    - website
    - website_and_customer_group
    - customer_group
    - none (default)
      С м2 2.2.6 была внедрена параллелизация. Для того чтобы настроить необходимо необходимо в env.php
      добавить `'MAGE_INDEXER_THREADS_COUNT'=>3`
      или из терминала `MAGE_INDEXER_THREADS_COUNT=3 php -f bin/magento indexer:reindex catalog_product_price`

Например если у вас 3 вебсайта и 5 кастомер груп вам понадобиться разделить на 15 потоков

За хранения статуса индексера отвечают такие таблицы:

- `indexer_state`
- `mview_state`

#Indexer Table Switching

М2 оптимизирует определенные индексационные процессы для того, чтобы предотвратить взаимоблокировки и блокировки ожидания
В этих случаях magento использует отдельные для выполнения операций чтения и реиндекса. Как результат процесс переключения таблиц
не влияет на пользователей во время полного реиндекса. К примеру когда `catalog_product_price` индексируется, На клиентах это не отразится. 

Magento использует такие таблицы при процессе переключения.

![Alt text](./table_switch.png?raw=true "Priority")

Рассмотрим создание кастомного индекса:

создаём `etc/indexer.xml`

```
?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Indexer/etc/indexer.xsd">
    <indexer id="custom_index" view_id="custom_index" class="Vendor\ModuleName\Indexer\CustomIndex">
        <title translate="true">Custom Index</title>
        <description translate="true">Indexer that do something</description>
    </indexer>
</config>
```

`etc/mview.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Mview/etc/mview.xsd">
    <view id="custom_index" class="Vendor\ModuleName\Indexer\CustomIndex" group="indexer">
        <subscriptions>
            <table name="custom_table_index" entity_column="id" />
        </subscriptions>
    </view>
</config>
````

`Vendor\ModuleName\Indexer\CustomIndex`

```
<?php

namespace Vendor\ModuleName\Indexer;

use Magento\Catalog\Model\ResourceModel\Product\CollectionFactory;
use Magento\Framework\Indexer\ActionInterface;

class CustomIndex implements ActionInterface, \Magento\Framework\Mview\ActionInterface
{
    // срабатывает при полном реиндексе
    // будет срабатывать при запуски соответствующей команда из консоли
    public function executeFull()
    {
        // do something
    }
    
    // срабатывает при массэкшн 
    public function executeList(array $ids)
    {
        // do something
    }

    // срабатывает для одного продукта
    public function executeRow($id)
    {
        // do something
    }

    // Используется mview, позволяет процессить индексер в режиме "Update on schedule"
    public function execute($ids)
    {
        //do something
    }
}
```

Список индексеров можно получить с помощью коллекции
`\Magento\Indexer\Model\Indexer\Collection`.

Индексеры представлены классом `\Magento\Indexer\Model\Indexer`. В этом классе есть
три метода для обновления индексеров `reindexAll`, `reindexRow` и `reindexList`.

В каждом методе используется сущность, которая реализует `\Magento\Framework\Indexer\ActionInterface`
Должны быть реализованы такие методы `executeFull()`, `executeList(array $ids)`, `executeRow($id)`


#Индексер цены

Индексация цены определяют самую низкую цену среди `(price, special price, tier price, catalog price rules)`
Данные сохраняются в `catalog_product_index_price`.

Индексер цены перебирает в цикле модель индексации `(indexerModel)` для каждого типа продуктов.
Эти сущности должны реализовывать `\Magento\Framework\Indexer\DimensionalIndexerInterface`

Php процесс разветвляется во столько дочерних процессов во сколько необходимо.
Для того чтобы запустить реиндексацию многопоточно, необходимо перед запуском параметру
`MAGE_INDEXER_THREADS_COUNT` присвоить значение 3. Это поможет обеспечить то что
большой каталог с большим количеством `tiered price` будут индексироваться эффективнее. То есть пока будет
индексация прайсов остальные индексы пройдут.


Индексеры запускаются партиями (смотри `(see \Magento\Framework\Indexer\BatchProvider::getBatches`),
чтобы предотвратить ошибки связанные с памятью.

#Индексер инвентаря

Индексер инвентаря определяет количество товаров на складе и доступность товара для заказа.
Главный индексатор для инвентаря `\Magento\CatalogInventory\Model\Indexer\Stock`. Каждая операция индексации
`(full, list, ID)` имеет свой собственный индексирующий класс. Для каждого типа продукта можно указать
свой собственный `stock indexer` в `stockIndexerModel` классе. Это используется в `bundle`, `configurable`, `grouped`
типах продуктов, потому что из `stock status` зависит от дочерних продуктов.

#EAV индексер

Это используется для объеденения атрибутов для поиска. С добавлением ElasticSearch в Magento 2.3
из коробки. Mysql Search прекращает поддерживаться.


