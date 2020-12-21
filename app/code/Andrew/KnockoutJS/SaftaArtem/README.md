#Knockout JS ViewModel

Создаём phtml файл

````
<script type="text/x-magento-init">
    {
        "*": {
            "Magento_Ui/js/core/app": {
                "components": {
                    "startSimple": {
                        "component": "Module_Name/js/plain-view-model"
                    }
                }
            }
        }
    }
</script>

<div data-bind="scope: 'startSimple'">
    <h2 data-bind="text: title"></h2>
</div>
````


После этого создаём `frontend/web/js/plain-view-model.js`

````
define([], function () {
    'use strict';

    return function () {
        return {
            title: 'This is very fine title'
        }
    }
});
````


Отладка `knockout js` в консоли

`require('uiRegistry').get('startSimple');` После этого мы можем вызвать свойство или метод компонента.

# observable

Объяснение: 
`this.firstName = ko.observable("Bert");` Свойство превращается в функцию. 
 `require('uiRegistry').get('startSimple').firstName('Bert');`
Атрибут text `<p>First name: <strong data-bind="text: firstName"></strong></p>` - позволяет вывести пе в тэге стронг.
Атрибут value `<p>Last name: <input data-bind="value: lastName" /></p>` - позволяет записать в свойство `lastName` введенное значение.



Меняем  `frontend/web/js/plain-view-model.js` файл

````
define(['ko'], function (ko) {
    'use strict';

    return function () {
        return {
            title: ko.observable('This is very fine title')
        }
    }
});
````

Теперь title это функция.

Если выполнить изменения в нашем компоненте из консоли фронта. Например

`require('uiRegistry').get('startSimple').title('Something else');`;

То контент поменяется.


#subscribe 
Пример подписки на изменения `title`.
````
define(['ko'], function (ko) {
    'use strict';

    return function () {
        const title = ko.observable('This is very fine title');
        title.subscribe(function (oldValue) {
            console.log('will be changed from', oldValue );
        }, this, 'beforeChange');
        title.subscribe(function (newValue) {
            console.log('change to', newValue );
        });
        return {
            title: title
        }
    }
});
````

При изменении `title` будет срабатывать функция `subscribe`

#computed

Позволяет получать динамическое значение основанное на наблюдаемых значениях.

```
define(['ko', `jquery`], function (ko, $) {
    'use strict';

    return function (config) {
        let currencyInfo = ko.observable();

        $.getJSON(config.base_url + 'rest/V1/directory/currency', currencyInfo);

        const viewModel = {
            label: ko.observable('Currency Info')
        };

        viewModel.output = ko.computed(function () {
            if (currencyInfo()) {
                return this.label() + ':\n' +  JSON.stringify(currencyInfo(), null, 2)
            }
            return 'Loading ...'
        }.bind(viewModel));
        return viewModel;
    }
});
```

- Создаём свойство `currencyInfo` превращаем его в функцию.
- Отправляем запрос на М2 api получаем currency, вторым параметром передаём нашу функцию `currencyInfo()` при `success`
в эту функцию сетится результат от запроса к апи.
- Создаём свойство `viewModel` это объект со свойством `label()`
- Задаем свойству `output` - вычисляемое значение на основе `label()` и данных записанных в `currencyInfo`
- возвращаем viewModel


#observable arrays

```
<script type="text/x-magento-init">
    {
        "*": {
            "Magento_Ui/js/core/app": {
                "components": {
                    "startSimple": {
                        "component": "Safta_Module/js/plain-view-model"                    }
                }
            }
        }
    }

</script>
<div id="dsada" data-bind="scope: 'startSimple'">
    <table data-bind="foreach: exchange_rates">
        <tr>
            <td data-bind="text: currency_to"></td>
            <td data-bind="text: rate"></td>
        </tr>
    </table>
</div>
```

```
define(['ko'], function (ko) {
    'use strict';

    return function () {
        const viewModel = {
            exchange_rates: ko.observableArray([
                {
                    'currency_to': 'USD',
                    'rate': 1.0
                }
            ])
        }
        return viewModel;
    }
});
```


`exchange_rates` метод, который позволяет добавить элементы массива к в `js` c помощью `push`, `pop` и перерендерить данные в поле.

Например:

`require('uiRegistry').get('startSimple').exchange_rates.push({'currency_to': 'EUR', 'rate': 0.68})`

Позволяет добавить новую строку в таблице с новым `currency`

Или можно удалить `require('uiRegistry').get('startSimple').exchange_rates.pop();` последний элемент массива.

Обычный массив можно перебрать так:

```
<div id="dsada" data-bind="scope: 'startSimple'">
    <ul data-bind="foreach: values">
        <li data-bind="text: $data"></li>
    </ul>
</div>
```

```
define(['ko'], function (ko) {
    'use strict';

    return function () {
        const viewModel = {
            values: ko.observableArray([
                121,
                3123,
                1213,
                212133
            ])
        }
        return viewModel;
    }
});

```

#Виртуальные элементы
Можно выполнить цикл с помощью виртуальных элементов. Как показано на примере ниже.

```
<div data-bind="scope: 'startSimple'">
    <table>
        <tr>
        <th data-bind="i18n: 'Currency to'"></th>
        <th data-bind="i18n: 'Rate'"></th>
        </tr>
        <!-- ko foreach: exchange_rates -->
        <tr>
            <td data-bind="text: currency_to"></td>
            <td data-bind="text: rate"></td>
        </tr>
        <!-- /ko -->
    </table>
</div>
```

```
define(['ko'], function (ko) {
    'use strict';

    return function () {
        const viewModel = {
            exchange_rates: ko.observableArray([
                {
                    'currency_to': 'USD',
                    'rate': 1.0
                },
                {
                    'currency_to': 'EUR',
                    'rate': 0.86
                },
                {
                    'currency_to': 'GRIVNA',
                    'rate': 28
                }
            ])
        }
        return viewModel;
    }
});
```

#Ui component

Маппинг для этого алиаса `uiElement` находится `pub/static/frontend/Magento/luma/en_US/requirejs-config.js:468`
Наследование для ui component выглядит так.
`uiCollection -> uiElement -> uiClass`

`uiClass`  не создаёт компонентов. Самый элементарной частицей является uiElement.

Пример просто uiComponent который наследует uiElement:

```
<div data-bind="scope: 'exampleUiComponent'">
    <div data-bind="text: label"></div>
</div>
```

```
define(['uiElement'], function (uiElement) {
    'use strict';

    return uiElement.extend({
        label: 'My firts uicomponent'
    });
});
```

Ui component очень хорошо настраиваются. Свойства могут перезаписаны через конфигурацию в phtml.
html блоки необходимо выводить в отдельные файлы. Темплэйты можно установить как в самом `x-magento-init script` так и в самом компоненте в `js`

Ui component можно вкладывать один в другой. Каждый компонент хранит в себе namespace, индекс компонента и полный путь к компоненту.

Получение компонента через консоль.

- `require('uiRegistry').get('ParentComponent.ChildComponent').name`
- `require('uiRegistry').get('index = ChildComponent').name` - получение по индексу
- `require('uiRegistry').get('np = ParentComponent').name` - получение по namespace
- `require('uiRegistry').get('np = ParentComponent, index = ChildComponent').name` - комбинированные селекторы
- `require('uiRegistry').get('np = ParentComponent, index = ChildComponent', function(c) {console.log(c.name)})` - можно передать вторым параметром колбэк
- `require('uiRegistry').get(function(c) {console.log(c.name)})` - получение имен всех ui components на странице



 






