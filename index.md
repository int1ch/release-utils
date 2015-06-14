---

layout: default

title: make release

---

# Яндекс

## **{{ page.title }}** {#cover}

<div class="s">
    <div class="service">HOME</div>
</div>
<!--<div class="nda"></div>-->

<div class="info">
	<p class="author">{{ site.author.name }}, <br/> {{ site.author.position }}</p>
</div>


## Обзор
{:.section}

### Огранизация релиза

## Проект и команда

* Главная страница: www.yandex.ru
    * debian пакеты
	* 4 основных пакета, 2 репозитория
	* 10 разработчиков, 5 тестировшиков, 3 админа
	* Ежедневные релизы
	* git-flow, c основными ветками `dev`+`release`, `bugfix`

## Что такое релиз

* Таск 
* Решение, тестирование, Коммит, Еще коммит, push
* Сборка пакета
* Выкладка и тестирование РелизКандидата (RC)
* Выкладка в продакшен
* Откат или Хот-Фикс

## wiki страничка: morda/release

![placeholder](pictures/wiki-morda-release.png){:.right-image}

* Какую версию?
* Какие пакеты?
* Доп. Инструкции Админам
* Ход релиза
* История
    * Что выкатили 20ого?
    * На что откатывать?


## Автоматизация Релиза
![placeholder](pictures/release-process.png)


## Инструмент
{:.section}

### make release

## Что это?
* `release_utils.pl` aka `make release`
	* https://github.yandex-team.ru/morda/gitrelease
* Задачи
	* покоммитные изменения в `debian/changelog`
	* контроль названий версий
	* мердж веток по git-flow
	* решение конфликтов
	* итерирование версий подрепозиториев

## Алтернатива dch -i

~~~ javascript
cat debian/release_utils.conf
{
    flow => {
        master => {
            release => q{master},
        },
    },
    package_version_type => 'MajorMinor'
}
~~~

~~~ javascript
yandex-digits-front (1.40) unstable; urgency=low
  * 93c4f3f124189d3c86af7ebaae0606656e3677dc | MIke NIkitin <mellior@yandex-team.ru> | Mon Jun 8 19:34:30 2015 +0300
      PASSP-11791: Поднять Капчу в MANе, ipv6only mode for finland, mixed mode for others
          404 as static files for unknown keys
 -- Mike Nikitin <buildfarm@yandex-team.ru>  Mon, 08 Jun 2015 19:35:16 +0300
~~~

## git-flow
~~~ javascript
{
    flow => {
        dev => {
            release         => 'release',
            back_merge      => [qw/master dev/],
            conflict_accept => q{debian/changelog},
        },
        release =>{
            release         => 'release',
            back_merge      => [qw/master dev/],
            conflict_accept => q{debian/changelog},
        },
        bugfix => {
            release         => 'bugfix',
            back_merge      => [qw/master dev/]
            conflict_reject => q{debian/changelog},
        },
    },
    package_version_type => 'DateMinor',
}
~~~

## Одна команда

~~~
mellior@v23:/opt/www/morda-v23d2 (git: dev)
make release
~~~

~~~
mellior@v23:/opt/www/morda-v23d2 (git: release)
make release
~~~

~~~
mellior@v23:/opt/www/morda-v23d2 (git: bugfix)
make release
~~~

## Под репозитории

* Пакет скинов
* Пакет статики

~~~ javascript
{
    prerun                => [
        "cd ./tmpl/skins/; ../../release_utils/make_release.pl $run_options --no-push --no-back-merge",
        "cd ./tmpl/; ../release_utils/make_release.pl $run_options --no-push --no-back-merge",
    ],
}
~~~ 

## Условное итерирование

~~~ javascript
{
    watch_files          => [
        qr{^tmpl/skins},
        qr{^tmpl/skins/debian},
    ],
    ignore_files         => [
        qr{(content|hash|page|config|index|skins|server)\.js$},
        sub {
            !m{bemjson\.js$} and
            m{tmpl/v12/pages-desktop/(?:[^\/]+)/[^\/]+\.js},
        },
        qr{priv\.js$},
     ],
}
~~~

## Инструмент
{:.section}

### Web версия 



## Web
{:.center}

![](pictures/release-web.png)

## Задачи

* Запуск make relase на сервере с сохранеием логов
* Контролько коммитов
    * Каие задачи в каких ветках
* Состояние пакетов: выкатился да/нет
*  

## Нерешенные задачи

* Перевод стосяний тасков в ST
* Сборка дневной версии
* Работа с несколькоми ре







## Заголовок

### Вводный текст (первый уровень текста)
![placeholder](pictures/vertical-placeholder.png){:.right-image}

*  Второй уровень текста
	* Третий уровень текста (буллиты)
	* Третий уровень текста (буллиты)

	1. Четвертый уровень текста

## &nbsp;
{:.with-big-quote}
> Цитата

Текст
{:.note}

## Пример подсветки кода на JavaScript

~~~ javascript
!function() {
    var jar,
        rstoreNames = /[^\w]/g,
        storageInfo = window.storageInfo || window.webkitStorageInfo,
        toString = "".toString;

    jar = this.jar = function( name, storage ) {
        return new jar.fn.init( name, storage );
    };

    jar.storages = [];
    jar.instances = {};
    jar.prefixes = {
        storageInfo: storageInfo
    };

    jar.prototype = this.jar.fn = {
        constructor: jar,

        version: 0,

        storages: [],
        support: {},

        types: [ "xml", "html", "javascript", "js", "css", "text", "json" ],

        init: function( name, storage ) {

            // Name of a object store must contain only alphabetical symbols or low dash
            this.name = name ? name.replace( rstoreNames, "_" ) : "jar";
            this.deferreds = {};

            if ( !storage ) {
                this.order = jar.order;
            }

            // TODO – add support for aliases
            return this.setup( storage || this.storages );
        },

        // Setup for all storages
        setup: function( storages ) {
            this.storages = storages = storages.split ? storages.split(" ") : storages;

            var storage,
                self = this,
                def = this.register(),
                rejects = [],
                defs = [];

            this.stores = jar.instances[ this.name ] || {};

            // Jar store meta-info in lc, if we don't have it – reject call
            if ( !window.localStorage ) {
                window.setTimeout(function() {
                    def.reject();
                });
                return this;
            }

            // Initiate all storages that we can work with
            for ( var i = 0, l = storages.length; i < l; i++ ) {
                storage = storages[ i ];

                // This check needed if user explicitly specified storage that
                // he wants to work with, whereas browser don't implement it
                if ( jar.isUsed( storage ) ) {

                    // If jar with the same name was created, do not try to re-create store
                    if ( !this.stores[ storage ] ) {

                        // Initiate storage
                        defs.push( this[ storage ]( this.name, this ) );

                        // Initiate meta-data for this storage
                        this.log( storage );
                    }

                } else {
                    rejects.push( storage );
                }
            }

            if ( !this.order ) {
                this.order = {};

                for ( i = 0, l = this.types.length; i < l; i++ ) {
                    this.order[ this.types[ i ] ] = storages;
                }
            }

            if ( rejects.length == storages.length ) {
                window.setTimeout(function() {
                    def.reject();
                });

            } else {
                jar.when.apply( this, defs )
                    .done(function() {
                        jar.instances[ this.name ] = this.stores;

                        window.setTimeout(function() {
                            def.resolve([ self ]);
                        });
                    })
                    .fail(function() {
                        def.reject();
                    });
            }
            return this;
        }
    };

    jar.fn.init.prototype = jar.fn;

    jar.has = function( base, name ) {
        return !!jar.fn.meta( name, base.replace( rstoreNames, "_" ) );
    };
}.call( window );
~~~

## Пример подсветки кода с дополнительным текстом

Вводный текст

~~~ javascript
!function() {
    var jar,
        rstoreNames = /[^\w]/g,
        storageInfo = window.storageInfo || window.webkitStorageInfo,
        toString = "".toString;

    jar = this.jar = function( name, storage ) {
        return new jar.fn.init( name, storage );
    };

    jar.storages = [];
    jar.instances = {};
    jar.prefixes = {
        storageInfo: storageInfo
    };
}.call( window );
~~~

## &nbsp;
{:.big-code}

~~~ javascript
!function() {
    var jar,
        rstoreNames = /[^\w]/g,
        storageInfo = window.storageInfo || window.webkitStorageInfo,
        toString = "".toString;

    jar = this.jar = function( name, storage ) {
        return new jar.fn.init( name, storage );
    };

    jar.storages = [];
    jar.instances = {};
    jar.prefixes = {
        storageInfo: storageInfo
    };
}.call( window );
~~~

## Заголовок
{:.images}

![](pictures/horizontal-placeholder.png)
*Текст*

![](pictures/horizontal-placeholder.png)
*Текст*

![](pictures/horizontal-placeholder.png)
*Текст*

## Заголовок
{:.images .two}

![](pictures/horizontal-middle-placeholder.png)
*Текст*

![](pictures/horizontal-middle-placeholder.png)
*Текст*

## Заголовок
{:.center}

![](pictures/horizontal-big-placeholder.png)

## **![](pictures/cover-placeholder.png)**

## ![](pictures/horizontal-cover-placeholder.png)
{:.cover}

## Таблица

|  Locavore      | Umami       | Helvetica | Vegan     |
+----------------|-------------|-----------|-----------+
| Fingerstache   | Kale        | Chips     | Keytar    |
| Sriracha       | Gluten-free | Ennui     | Keffiyeh  |
| Thundercats    | Jean        | Shorts    | Biodiesel |
| Terry          | Richardson  | Swag      | Blog      |
+----------------|-------------|-----------|-----------+


## Таблица с дополнительным полем

{:.with-additional-line}
|  Locavore      | Umami       | Helvetica | Vegan     |
+----------------|-------------|-----------|-----------+
| Fingerstache   | Kale        | Chips     | Keytar    |
| Sriracha       | Gluten-free | Ennui     | Keffiyeh  |
| Thundercats    | Jean        | Shorts    | Biodiesel |
| Terry          | Richardson  | Swag      | Blog      |
+----------------|-------------|-----------|-----------+
| Terry          | Richardson  | Swag      | Blog      |

## **Контакты** {#contacts}

<div class="info">
<p class="author">{{ site.author.name }}</p>
    <p class="position">{{ site.author.position }}</p>

    <div class="contacts">
        <p class="contacts-left contacts-top phone">+7 (000) 000-00-00</p>
        <p class="contacts-left mail">почта@yandex-team.ru</p>
        <p class="contacts-right contacts-top twitter">@twitter</p>
        <!-- <p class="contacts-right contacts-bottom vk">vk</p> -->
        <p class="contacts-right facebook">facebook</p>
    </div>
</div>
