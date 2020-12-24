#Демонстрация возможности выполнения комплексных операций с помощью M2 Pricing Framework


#Понимание подсчета цены и rendering framework.

Цены указаны в \Magento\Catalog\Model\Product\Type\Price классе или в его потомке.
Интерфейс для этой структуры не указан.

Вот несколько типов цены для простого продукта:

- `Price` - базовое значение, которое храниться в бд.
- `Tiered Price` - скидка которая возможна в зависимости от количества товаров, веб сайта и группы кастомеров.
- `Special price` - уменьшение цены, которая действует на определенный период времени.
- `Base price` - (принимает параметр qty) минимум от tired price и special price
- `Final Price` - Base Price плюс итоговая сумма по опциям


Практический пример:

Ставим точку останова в строке 33 (renderAmount), и проходим по цепочке для всех типов продуктов
`vendor/magento/module-configurable-product/view/base/templates/product/price/final_price.phtml`


Типы цены для продуктов:

```
\Magento\Catalog\Model\Product\Type\Price
\Magento\ConfigurableProduct\Model\Product\Type\Configurable\Price
\Magento\Bundle\Model\Product\Price
\Magento\Downloadable\Model\Product\Price
\Magento\GiftCard\Model\Catalog\Product\Price\Giftcard
\Magento\GroupedProduct\Model\Product\Type\Grouped\Price
```

`Configurable product:`

Вызывается с помощью метода `getPrice`. 
Проверяет если простой продукт сконфигурирован. 
Если есть цена рассчитанная на основе простого продукта. 
В остальных случаях возвращает 0.

`Bundle product:`

Отменяет `final price` подсчет и добавляет общее значение цены для всех айтемов к `base_price`. 
Помните что `Bundle` товары все еще имеют `base_price`.


`Downloadable product:`

Отменяет `final price` подсчет, чтобы `downloadable` ссылки могли продаваться отдельно

`Giftcard:`

Отменяет `price` и `final price` , чтобы указать значение настраиваемого параметра 
для указанной суммы.

`Grouped:`

Отменяет final price подсчет, для того чтобы добавить сумму для всех дочерних продуктов, которые можно найти в associated_product_[child product id] кастом опции

`Special price`

`Special price` имеет два добавочных поля. 
Дата начала и конца этой цены. 
Вы можете заметить что этих полей нет в `Enterprise` версии. 
Это потому что это версия использует
`staging start and stop dates`. Таким образом, поля `special_from_date` и `special_to_date` 
загружаются так же, как обычные атрибуты.

`Price rendering`

Для отображения цены в М2 создан специальный хэндл `catalog_product_prices`
`(vendor/magento/module-catalog/view/base/layout/catalog_product_prices.xml).`
Хороший пример как воспользоваться этой функцией можно найти в 
`\Magento\Catalog\Block\Product\ListProduct::getProductPrice` методе

```
$this->getLayout()->getBlock('product.price.render.default')->render(
    \Magento\Catalog\Pricing\Price\FinalPrice::PRICE_CODE,
    $product,
    [
        'include_container' => true,
        'display_minimal_price' => true
    ]
);
```

Следуя окольным путём `(vendor/magento/module-catalog/view/base/
layout/empty.xml)` этот хэндл включен на каждой странице во фронте.
М2 позволяет указывать определенные рендереры и шаблоны для каждого типа продуктов.

`Default price renderers:`

- Default: `Magento\Catalog\Pricing\Render\PriceBox`
- Amount: `Magento\Framework\Pricing\Render\Amount` Используется для отображения суммы. Например на странице категории, это рендерер отображает для конфигурируемого продукта `As low as: $12`.
- Final Price: `Magento\Catalog\Pricing\Render\FinalPriceBox` как и предполагалось это гарантирует что продукт `isSaleble`, и только потом отображается цена.
- Configured Price: `Magento\Catalog\Pricing\Render\ConfiguredPriceBox`

Эти рендереры имеют свой порядок загрузки


`Пример для простого продукта:`

- `simple/prices/final_price/render_class`
- `simple/default_render_class`
- `default/prices/final_price/render_class`
- `default/default_render_class`


`Какая роль индексации? Как работают разные модификаторы цены вместе?`

Индексация делает расчет цены и диапазона цен для продуктов. Цель этого - исключить избыточные расчеты цен и вместо этого сохранить эти значения в базе данных. 
Индексер `\Magento\Catalog\Model\ResourceModel\Product\Indexer\Price\Query\BaseFinalPrice::getQuery` 
вставляет во временные таблицы. Затем модификаторы вступают в дело и применяют свою логику и корректировки к значениям в этой таблице. Как только модификаторы отработают, 
все цены перемещаются в основную таблицу catalog_product index.


`Pricing modifiers:`
Модифицируют цену из темповых таблиц.
`(\Magento\Catalog\Model\ResourceModel\Product\Indexer\Price\BasePriceModifier::modifyPrice)`
К примеру если продукт не отображается когда он out of stock `CatalogInventory` удаляет индексы цен для всех продуктов, которые out of stock

`Customizing the Catalog`

```
Magento\CatalogInventory\Model\Indexer\ProductPriceIndexFilter
Magento\CatalogRule\Model\Indexer\ProductPriceIndexModifier
Magento\Catalog\Model\ResourceModel\Product\Indexer\Price\CustomOptionPriceModifier
```

`Практический задача:`

Создать php скрипт в PhpStorm для выполнения команды `bin/magento
indexer:reindex catalog_product_price.`

Установить точки останова в` Magento\Catalog\Model\ResourceModel\Product\Indexer\Price\SimpleProductPrice::executeByDimensions`.
Вы также можете установить точки останова в любом классе который реализует `Magento\Framework\Indexer\DimensionalIndexerInterface`.

Как цена считается странице продукта и как на странице категории?
Взаимоотношения классов подсчета цены и индексера цены

Цены на странице товара подсчитываются на лету.
Цены на странице категорий берутся из индекса.

На сколько я знаю нет связи между классами расчета цены и индекса цены. 
Причина этого в том, что они подходят к проблеме расчета цен с двух сторон 
и используются для решения двух разных задач.

Классы расчета цены оперируют добавочной информацией. 
Например они используют количество товара для определения `final price`. 
Они используют custom options, которые характеризуют дочерние продукты, 
чтобы определить сколько стоят дочерние продукты.

Индексы цены используются для отображения и начальной цены очень быстро на странице продуктов. 
Для них добавочная информация не нужна. 
Исключение для страницы категории на энтерпрайз версии там индексы не используются.


`The Magento\Framework\Price\* component and
its extension in the Catalog module`

Все ценовые сущности в M2 расширяют `\Magento\Framework\Pricing\Price\AbstractPrice`
который реализует `\Magento\Framework\Pricing\Price\PriceInterface`

В Magento_Catalog  есть такие цены:
- TierPrice 
-CustomOptionPrice
-SpecialPrice
- RegularPrice 
- FinalPrice 
- ConfiguredPrice 
      
Все наследуются от этого класса

Практическое задание:

Установить точку останова в

```
Magento\Framework\Pricing\Render::render
Magento\Framework\Pricing\Render::renderAmount
Magento\Framework\Pricing\Render\Amount::_toHtml
```


Настройки цены: 
- `tier prices `
- `special price `
- `custom option `
- `configurable adjustment `
- `catalog rules` 
- `cart rules`

`Teir prices`

`Tier prices `- это возможность сделать скидку при покупке определенного количества 
товаров Цены настраиваются для комбинация вебсайтов и груп пользователей


`Special prices`

`Special prices`- это другой путь привлечения клиентов. Есть три поля:

 -  `Special Price`,
 -  `Special Price` From (дата начала действия специальной цены)
 -  `Special Price` To (дата окончания)

`Custom options`

Это позволяет пользователю изменить информацию о продукте. 
Например пользователь может указать инициалы, которые будут выгравированы на продукте.
Это может выглядеть как для конфигурируемых продуктов. 
Главное правило это то, что
`custom options` делают модификацию юнита в инвентаре.

`Configurable adjustment`

Конфигурируемый продукт это путь объединения простых продуктов, каждый из которых имеет количество 
в инвентаре. После выбора опции для продукта, конфигурируемый продукт 
является просто оболочкой для простого продукта.
Это основное отличие от custom options. 
Конфигурируемый продукт определяет юнит инвентаря который необходимо доставить. 
Custom options модифицируют юнита в инвентаре.

`Catalog rules`

`Catalog rules` корректируют цену перед добавлением в корзину. 
Кастомные атрибуты, категории и атрибут сеты доступны для создания условий. 
Каждый рул имеет action. Например скидка


`Cart rules`

`Cart rules` корректируют цену после добавления в корзину. 
Они применяются на основе вебсайта и customer group. 
Вы можете установить код купона, который можно ввести в корзине. 
Если вы хотите вы можете добавить правило на основе атрибутов. 
Условиях будет состоять из  адреса, заказов пользователей. 
Также вес товара, адрес доставки доступны для создания условия правила.
Основная непонятная часть это то что условие и экшн имеют условия. 
Вкладка Conditions следует ли активировать это правило. 
Вкладка Actions определяет для каких продуктов будет работать правило. 
Хороший пример это бесплатная доставка для определённых продуктов. 
Может какие-то продукты слишком тяжелые для экономичной доставки. 
Вы хотите рассчитать ставки для тяжелых продуктов, но исключить расчет для легких товаров.

`Практическая задача`

- Создать продукт с custom option.
- Создать конфигурируемый продукт. Можете ли создать custom option для конфигурируемого продукта?
- Создать catalog rule. Какие условия там присутствуют? Как условия могут быть модифицированы
- Создать `cart price rule`. Настройте бесплатную доставку для женских жакетов. 
  `Shipping` должен считаться для всех других продуктов в корзине.

`Как изменить процесс отображения цены для данных типов продуктов?`

M2 использует хэндл для ценообразования продукта.
Изменить базовый блок, который отвечает за отображение цены очень легко, 
вы можете использовать плагины и переписывать шаблоны отображения.

`Как добавить кастомную корректировку цены?`

`Что случится если индекс цены будет не согласовываться с этой корректировкой?`

Плагин это лучшее изобретение со времен изобретения нарезанного хлеба, 
по крайней мере в стране Magento. 
Если вам надо поменять поведение защищенных методов, 
вы можете создать preference что изменить расчет цены.
Запомните расчет используется для страницы продуктов и не только. 
Если эти изменения не синхронизированы, то пользователь будет видеть одни 
цены на странице категорий и другие на странице продукта.


