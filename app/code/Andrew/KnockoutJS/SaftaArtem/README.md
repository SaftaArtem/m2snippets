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

#computed




