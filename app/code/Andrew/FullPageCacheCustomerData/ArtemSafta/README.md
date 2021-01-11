#FPC Customer Data approach adding  private content

Модуль который будет добавлять свою секцию должен декларировать 
зависимость от `Magento_Customer`.

Расширение Customer Data Section должна включать в себя такие части:

- Класс на бэкенде, который должен обеспечить данными с помощью `array_map`
- JS модуль, который извлекает и отображает данные.
- Правила, которые определяют когда данные должны снова извлекаться с помощью браузера. 
  Данные должны быть снова извлечены, потому что браузер их хранит в `LocalStorage`
  и с какой-то периодичностью нужно извлекать актуальные данные. 

По дефолту кэш в локал сторадж хранится 1 час.

Это можно сконфигурировать в админ панели по пути:

`Stores -> Configuration -> Customers -> Customer Configuration -> Online Customers Options` 

Браузер обновляет customer data section только если есть кука `private_content_version`

Класс на бэкенде будет храниться в папке `CustomerData`
Пример класса:
```
<?php

namespace Module\Name\CustomerData;

use Magento\Customer\CustomerData\SectionSourceInterface;
use Module\Name\Model\FavoriteProductsRepository;

class FaboriteProducts implements SectionSourceInterface
{
    private FavoriteProductsRepository $favoriteProductsRepository;

    public function __construct(FavoriteProductsRepository $favoriteProductsRepository)
    {
        $this->favoriteProductsRepository = $favoriteProductsRepository;
    }

    public function getSectionData()
    {
        return ['skus' => $this->favoriteProductsRepository->getSkus()];
    }
}

```

Должен реализовывать `SectionSourceInterface`
и метод `getSectionData`.
Возвращаемый массив должен быть ассоциативным.

Дальше регистрируем наш класс в di.xml

```
    <type name="Magento\Customer\CustomerData\SectionPool">
        <arguments> 
            <argument name="sectionSourceMap" xsi:type="array">
                <item name="favorite_products" xsi:type="string">Module\Name\CustomerData\FaboriteProducts</item>
            </argument>
        </arguments>
    </type>
```

Далее создаём рул при котором данные обновятся.

Если не задать рулы браузер не сможет обновлять данные.

Создаём `sections.xml`. Она доступна только для area frontend

```
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Customer:etc/sections.xsd">
    <action name="result/V1/module_name/favouriteproducts">
        <section name="favorite_products"/>
    </action>
</config>
```

Пример использования customer data на фронте

````
define(['jquery', 'Magento_Customer/js/customer-data'], function ($m, customerData) {
    'use strict';

    const favourites = customerData.get('favorite_products');

    const favouriteSkus = function () {
        return -1 !== favouriteSkus().indexOf(sku)
    };

    return function (config) {
        this.favouriteSkus = favouriteSkus;
        this.isFavourite = isFavourite;
        this.select = function (sku) {
            return function () {
                const url = config.update_url + sku;
                if (isFavourite(sku)) {
                    $.ajax(url, {method: 'DELETE'})
                } else {
                    $.ajax(url, {method: 'POST'})
                }
            }
        }
    }
});
````

За извлечение данных отвечает `Magento_Customer/js/customer-data` модуль
