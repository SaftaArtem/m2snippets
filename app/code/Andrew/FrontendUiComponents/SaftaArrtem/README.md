#Frontend Ui Components

Компоненты могут обладать такими параметрами:

- `component` - название компонента
- `children` - массив с другими дочерними компонентами
- `template` - html шаблон для компонента
- `displayArea` - группа к которой относится компонент
- `deps` - директива, которая позволяет передавать данные из дочерних компонентов



Для того чтобы получить массив дочерних компонентов одной группы, 
необходимо вызвать у компонента метод `getRegion('groupA)`
Метод `getTemplate()` запускает отрисовку компонента.


Для того чтобы превратить обычное свойство компонента в `observable` 
необходимо к дефолтным значениям компонента добавить свойство `tracks`


UiComponents могут хранить данные в других компонентах. Для того чтобы передавать
данные между компонентами используют свойства `imports`,  `exports`.
```
define(['UiComponent'], function (Component) {
   'use strict'

   return Component.extend({
      defaults: {
          label: 'Component A',
          amount: 11
      }
   });
});


define(['UiComponent'], function (Component) {
    'use strict'

    return Component.extend({
        defaults: {
            label: 'Component B',
            value: 5.5,
            imports: {
                value: 'component-a:amount'
            }
        }
    });
});
```
`imports` - это объект, который записывает в переменую внешние данные, Также мы можем передать туда
функцию

```
define(['UiComponent'], function (Component) {
   'use strict'

   return Component.extend({
      defaults: {
          label: 'Component A',
          amount: 11
      }
   });
});


define(['UiComponent'], function (Component) {
    'use strict'

    return Component.extend({
        defaults: {
            label: 'Component B',
            value: 5.5,
            imports: {
                onAmountUpdate: 'component-a:amount'
            }
        },
        onAmountUpdate: function (newValue) {
            console.log(newValue)
        }
    });
});
```

Основная проблема порядок загрузки uiComponents. Если мы хотим импортировать данные из компонента,
который еще не загрузился надо добавить  к свойству, которое мы хотим получить `tracks: {importProps: true}`

`links` - выполняет те же действия, что и `imports`,  `exports`.

Конфигурацию можно выполнить не только в phtml, но и через xml

![Alt text](./component-config.png?raw=true "payment-schema")

Существующие компоненты можно кастомизировать используя `mixins`

Все компоненты имеют методы `get`, `set`, `remove`

Есть возможность добавить любое свойство компонента в localstorage. 
Для этого используется свойство `statefull`

Пример использования:

```
define(['uiComponent'], function (Element) {
    'use strict';

    return Element.extend({
       defaults: {
           tracks: {
               input: true
           },
           statefull: {
               input: true
           },
           input: 'some random string!'
       }
    });
});
```

```
<div data-bind="scope: 'exampleUiComponent'">
    <span>Your input:</span>
    <input data-bind="textInput: input" autofocus>
</div>
```


Пример получения данных из дочернего компонента, который грузится позже.
Для этого используется директива `deps`. Её можно использовать 
только в конфигурации в самом файле js компонента она не работает

```
<script type="text/x-magento-init">
    {
        "*": {
            "Magento_Ui/js/core/app": {
                "components": {
                    "exampleUiComponent": {
                        "component": "Safta_Module/js/plain-view-model",
                        "deps": ["exampleUiComponent.sharedState"],
                        "children": {
                            "sharedState": {
                                 "component": "Safta_Module/js/shared-state"
                            }
                        }
                    }
                }
            }
        }
    }

</script>
<div data-bind="scope: 'exampleUiComponent'">
   <div>
       The value is:
       <span data-bind="text: value"></span>
   </div>
</div>
```

```
// component exampleUiComponent
define(['uiComponent'], function (Element) {
    'use strict';

    return Element.extend({
        defaults: {
            imports: {
                value: "exampleUiComponent.sharedState:value"
            },
            tracks: {
                input: true
            },
            value: 0
        }
    });
});

// component sharedState
define(['uiComponent'], function (Element) {
    'use strict';

    return Element.extend({
        value: 42
    });
});
```

Пример использования внешнего js файла для хранения состояния.
Это два компонента кнопка(по клику на которую будет прибавляться значение) и компонент отображения значения (на каждый клик добавляется инкремент по дефолту 1)
Конфигурируем компоненты в phtml
```
<script type="text/x-magento-init">
    {
        "*": {
            "Magento_Ui/js/core/app": {
                "components": {
                    "mutator": {
                        "component": "Safta_Module/js/value-mutator",
                        "template": "Safta_Module/button",
                        "children": {
                            "display": {
                                 "component": "Safta_Module/js/value-display",
                                 "template": "Safta_Module/display"
                            }
                        }
                    }
                }
            }
        }
    }


</script>
<div data-bind="scope: 'mutator'">
    <!-- ko template: getTemplate() --><!-- /ko -->
</div>
```

Дальше создаём `Module_Name/view/frontend/web/js/counter-state.js`. В этом файле будет храниться состояние(количество кликов и значение)

```
define(['ko'], function (ko) {
    'use strict';

    return ko.track({
        counter: 0,
        increment: 1
    });
});
```

Создаём `Module_Name/view/frontend/web/js/value-mutator.js`

```
define(['uiComponent', 'Safta_Module/js/counter-state'], function (Component, state) {
    'use strict';

    return Component.extend({
        inc: function () {
            return state.increment;
        },
        increment: function () {
            state.counter += state.increment
        }
    });
});

```


Создаём `Module_Name/view/frontend/web/template/button.html`
```
<button data-bind="click: increment">Increment by <!-- ko text: inc() --><!-- /ko --></button>
<!-- ko foreach: elems -->
<!-- ko template: getTemplate() --><!-- /ko -->
<!-- /ko -->
```

По клику на кнопку в состояние добавляется в `counter` значение, все то что было и значение свойства increment. Также в методе `inc()` отображается значение инкремента

Создаём `Module_Name/view/frontend/web/js/value-display.js`

```
define(['uiComponent', 'Safta_Module/js/counter-state'], function (Component, state) {
    'use strict';

    return Component.extend({
        value: function () {
            return state.counter
        }
    });
});

```

Создаём `Module_Name/view/frontend/web/template/button.html`
```
<div>Current count: <span data-bind="text: value()"></span></div>
```

В этом компоненте отображается результирующее значение `counter` 