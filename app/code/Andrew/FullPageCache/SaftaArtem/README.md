# Обзор Full Page Cache

FPC перехватывает запросы от браузера к серверу.
Если необходимый ресурс отсутствует в кэше, то FPC отправляет запрос на приложение и получает ответ.
По необходимости этот ответ сохраняется. Есть shared cache, который может попадать в разные браузеры.
Какие-то ресурсы кэшируются, а другие не кэшируются. У некоторого кэша есть срок действия, а есть бессрочный.
Сервер добавляет заголовок в ответ. Этот заголовок сигнализирует, как ресурс будет кэшироваться.

Основные заголовки, которые M2 использует в ответе:

- Cache-Control: must-revalidate
- Cache-Control: no-cache
- Cache-Control: no-store
- Cache-Control: no-transform
- Cache-Control: public
- Cache-Control: private
- Cache-Control: proxy-revalidate
- Cache-Control: max-age=<seconds>
- Cache-Control: s-maxage=<seconds>


По умолчанию все `GET` и `HEAD` запросы кэширются.

`POST`, `PUT`, `DELETE` не кэшируются.

Аякс запросы к M2 api не кэшируются тоже.

M2 из коробки поддерживает два типа `fpc` кэша из коробки:

- `Built-in`
- `Varnish`

Настройка для кэша находятся в настройках в админ панели по такому пути:
`Stores -> Configuration -> Advanced -> System -> Full Page Cache`

`Varnish` требует конфигурации, которая зависит от многих факторов.
`Built-in` не требует конфигурации

`Built-in` - хранит кэш в папке `/var/page/cache`

`bin/magento cache:clean full_page`, команда которая сбрасывает `fpc`
`bin/magento cache:status` - отображает какой кэш включен.
`bin/magento cache:disable full_page` - отключает `fpc`.
`bin/magento cache:enable full_page` - включает `fpc`.

Для `developer mode` `fpc` добавляет такие заголовки:
- `X-MAGENTO-CACHE-DEBUG: MISS` данные с сервера
- `X-MAGENTO-CACHE-DEBUG: HIT` - кэш из `Varnish`


M2 FPC хранит три типа контента:

- `public content (cms page, catalog page)`
- `shared content (bestsellers)`
- `private content (minicart)`

В большинстве случаев страницы кэшируются.

Способы создания некэшируемого контента:

- `LAYOUT XML` В блоке можно указать атрибут `cachable="false"`
- `HTTP Cache Headers` Можно добавить header с помощью плагина на класс \Magento\Framework\View\Layout\BuilderInterface и метод build.
Пример:
  
```
  /**
     * @var Http
     */
    private Http $response;

    public function __construct(
        Http $response
    ) {
        $this->response = $response;
    }

    /**
     * @param BuilderInterface $subject
     */
    public function afterBuild(BuilderInterface $subject, $result)
    {
        $this->response->setNoCacheHeaders();
        return $result;
    }
```

#Shared content
В отличие от `public content`  `Shared content` не ссылается на полностью сохранённый `html` документ, он ссылается только на часть документа.
Еще у `Shared content` есть время жизни кэша. Если у `public content`время жизни кэша может быть один день, то для `Shared content` время жизни составляет 1час.
`Built-in FPC` не поддерживает `Shared content` 

Для того чтоб создать `Shared content` необходимо к блоку в `xml` добавить атрибут `ttl=10`. Время зависит от бизнес логики.

M2 предоставляет два способа реализации `private content`
- `Private Scope Blocks` это блок, которые наследуются от `\Magento\Framework\View\Element\AbstractBlock` и в них перезаписан метод `isScopePrivate`
  Также для этого блока необходимо создать `depersonalization plugin`. Название класса DepersonalizePlugin. В коре можно найти примеры
- CustomerData


Как M2 рендерит приватный контент

`page_cache Miss`:

- Обезличивание сессии (depersonalize session)
- Это отображает всю страницу с приватными блоками без данных сессии
- Private content отображается в комментах
- Страница кэшируется в full page cache
- Страница отправляется в браузер

Следующие шаги работают только private_content_version кука установлена если запрос отправлен с помощью POST
- JavaSctipy парсит комменты которые генерируются для приватных блоков
- Фронтенд отправляет запрос /page_cache/block/renderer
- Контролер пересоздаёт окружение и загружает блоки без вызова `generateXml()`  
- Каждый запрошенный блок рендерится с данными из сессии





 


 



























