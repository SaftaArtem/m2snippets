#Jquery Ui Widgets

#Adding validation to form

Необходимо добавить в phtml следующий код:

````
<?php declare(strict_types=1); ?>

<form data-mage-init='{"validation: {}}'>
    <div class="control">
        <input type="text"
               name="email"
               id="email"
               class="input-text"
               placeholder="<?= __('Your Email') ?>"
               data-validate="{required: true}"
        >
        <button type="submit" class="action primary"><?= __('Submit') ?></button>
    </div>
</form>
````

Все рулы находяться в `vendor/magento/module-ui/view/base/web/js/lib/validation/rules.js`

JQuery UI widgets возвращают не объект, а функцию.

Для того, чтобы расширить функциональность виджета нам необходимо воспользоваться воспользоваться `Jquery Ui extensibility system`

Пример vendor/magento/module-catalog/view/frontend/web/js/catalog-add-to-cart.js:

Перепишем `submitForm` метод

Декларируем mixin в `frontend/require-config.js`

````
var config = {
    config: {
        mixins: {
            "Magento_Catalog/js/catalog-add-to-cart": {
                "Module_Name/js/catalog-add-to-cart-mixin": true
            }
        }
    }
}
````

Создаем в `view/frontend/web/js/catalog-add-to-cart-mixin.js`

````
define(['jquery'], function () {
    'use strict';

    return function (catalogAddToCart) {
        $.widget('mage.catalogAddToCart', catalogAddToCart, {
            submitForm: function (form) {
                console.log('Add to cart successfully intercepted');

                return this._super(form);
            }
        });
        return $.mage.catalogAddToCart;
    };
});
````

#Collapsible JQuery UI Widget

Виджет находиться по пути
`vendor/magento/magento2-base/lib/web/mage/collapsible.js`

alias для этого виджета `collapsible`

Пример использования

````
<div data-mage-init='"collapsible": {}}'>
    <div data-role="title">
        <h1>How about callabsible widget</h1>
    </div>
    <div data-role="content" style="display: none">
        This is collapsible content
    </div>
</div>
````

Для того чтобы добавить кнопку как триггер для отображения контента добавляем кнопку

````
<div data-mage-init='"collapsible": {}}'>
    <div data-role="title">
        <h1>How about callabsible widget <button data-role="trigger">GO</button></h1>
    </div>
    <div data-role="content" style="display: none">
        This is collapsible content
    </div>
</div>
````

Отобразить по умолчанию контент.
Добавляем в конце урла `#example`. Например:

`http://magento.loc/test/index/index#example`

А в отображении убираем `display:none`

````
<div data-mage-init='"collapsible": {}}'>
    <div data-role="title">
        <h1>How about callabsible widget <button data-role="trigger">GO</button></h1>
    </div>
    <div data-role="content">
        This is collapsible content
    </div>
</div>
````

Ещё вариант задать по умолчанию отображение для контента
````
<div data-mage-init='"collapsible": {"active": true}}'>
    <div data-role="title">
        <h1>How about callabsible widget <button data-role="trigger">GO</button></h1>
    </div>
    <div data-role="content">
        This is collapsible content
    </div>
</div>
````

Опции `collapsible` виджета:

 - `active` content всегда отображается
 - `saveState` сохраняем состояние после перезагрузки (Сохраняется в `localStorage`)
 - `animate` добавляем анимацию 
    Можно добавить количество ms вместо true (по умолчанию стоит 100 ms).
    Также можно добавить json c конфигурацией (`"animate": {"duration": 600, "easing": "linear"`})

#Загрузка контента с помощью ajax:

Создаём phtml

````
<div id="my-collapsible" data-mage-init='"collapsible": {"ajaxContent": true}}'>
    <div data-role="title">
        <h1>How about callabsible widget <button data-role="trigger">GO</button></h1>
    </div>
    <div id="example" data-role="content"></div>
    <a href="<?= $block->getViewFileUrl('Module_Name/example.html') ?>" data-ajax="true"></a>
</div>
````

Затем `frontend/web/example.html`

```
<div>
    <strong>This is collabsible content right here</strong>
    <div>
        This is collabsible content right here
        This is collabsible content right here
        This is collabsible content right here
        This is collabsible content right here
    </div>
</div>
<script>console.log('foo')</script>
```

Это работает для статического контента.

#Способ добавить `collapsible` виджет в `collapsible` виджет

Создаём `frontend/web/js/dynamic-collapsible.js`

````
define(['jquery', 'collapsible'], function () {
    'use strict';

    const waitForUpdate = function () {
        if(! this.content.attr('aria-busy')) {
            return this.content.trigger('contentUpdated');

            setTimeout(waitForUpdate.bind(this), 100);
        }

        $.widget('mage2tv.collapsible', $.mage.collapsible, {
            _loadContent: function () {
                this._super();
                waitForUpdate.bind(this)();
            }
        });

        return $.mage2tv.collapsible;
    }
});
````

Этот виджет расширяет функционал `collapsible` виджета из ядра

Дальше добавляем в наш phtml, где инициализируем наш кастомный виджет

````
<div id="my-collapsible" data-mage-init='"Module_Name/js/dynamic-collapsible": {"ajaxContent": true}}'>
    <div data-role="title">
        <h1>How about callabsible widget <button data-role="trigger">GO</button></h1>
    </div>
    <div id="example" data-role="content"></div>
    <a href="<?= $block->getViewFileUrl('Module_Name/example.html') ?>" data-ajax="true"></a>
</div>
````

А в `frontend/web/example.html`

````
<div data-mage-init='{"collapsible": {} }'>
    <div data-role="title">
        <strong>Header</strong>
    </div>
    <div data-role="content">
        content
        content
        content
    </div>
</div>
````








