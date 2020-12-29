#Localization in JavaScript

Добавить существующий перевод во фронтенд можно двумя способами:
- Добавить `jquery` в зависимости. Потом для строки использовать метод `$.mage.__('Строка')`
- добавить зависимость `mage/translate` с алиасом `$t`. Потом `$t('Строка')`

Пример:

```
define(['uiComponent', 'jquery', ], function (Component, $) {
    'use strict';

    return Component.extend({
        getTitle: function () {
            const remaining = 60 - new Date().getSeconds();
            return $.mage.__('Please order within %1 seconds!').replace('%1', remaining);
        }
    });
});
```

Строку нельзя записывать в переменную. Переводы храняться в `localStorage`

Также перевод можно добавить в `knockout templates`

Есть три способа:

- `<div data-bind="i18n: 'Welcome!'"></div>`
- `<div translate="'Welcome!'"></div>` Это не работает в phtml
- `<translate args="'Welcome!'"/>` Это не работает в phtml

