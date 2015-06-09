---

layout: default

title: make release

---

## **{{ page.title }}** {#cover}

<div class="s">
    <div class="service">HOME</div>
</div>
<!--<div class="nda"></div>-->

<div class="info">
	<p class="author">{{ site.author.name }}, <br/> {{ site.author.position }}</p>
</div>

## Что это?
* `release_utils.pl` aka `make release`
	* https://github.yandex-team.ru/morda/gitrelease
* Задачи
	* покоммитные изменения в `debian/changelog`
	* контроль названий версий
	* мердж веток по git-flow
	* решение конфликтов
	* итерирование версий подрепозиториев

## Где работает?

* Главная страница: https://www.yandex.ru
	* 10 разработчиков, 5 тестировшиков
	* 4 основных репозитария
	* Ежедневные релизы
	* git-flow, c основными веткками `dev`+`release`, `bugfix`

## Цепочка автоматизации
{:.center}

![](pictures/horizontal-big-placeholder.png)

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

## dch -i

~~~
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
~~~ 
yandex-digits-front (1.40) unstable; urgency=low
  * 93c4f3f124189d3c86af7ebaae0606656e3677dc | MIke NIkitin <mellior@yandex-team.ru> | Mon Jun 8 19:34:30 2015 +0300
      PASSP-11791: Поднять Капчу в MANе, ipv6only mode for finland, mixed mode for others
          404 as static files for unknown keys
 -- Mike Nikitin <buildfarm@yandex-team.ru>  Mon, 08 Jun 2015 19:35:16 +0300
~~~

## git-flow

~~~ 
{
    flow => {
        dev => {
            release         => 'release',
            back_merge      => [qw/master dev/],
            conflict_accept => q{debian/changelog debian/substvars tmpl/debian/changelog auto/mordax_version.pl},
            conflict_reject => q{},
        },
        release =>{
            release         => 'release',
            back_merge      => [qw/master dev/],
            conflict_accept => q{debian/changelog debian/substvars tmpl/debian/changelog auto/mordax_version.pl},
            conflict_reject => q{},
        },
        bugfix => {
            release         => 'bugfix',
            back_merge      => [qw/master dev/]
            conflict_accept => q{debian/substvars tmpl/debian/changelog auto/mordax_version.pl},
            conflict_reject => q{debian/changelog},
        },
    },
    package_version_type => 'DateMinor',
}
~~~

## Под репозитории

~~~
{
    prerun                => [
        "cd ./tmpl/skins/; ../../release_utils/make_release.pl $run_options --no-push --no-back-merge",
        "cd ./tmpl/; ../release_utils/make_release.pl $run_options --no-push --no-back-merge",
    ],
    after_new_version_cb => sub {
        my ( $package, $version ) = @_;
        print "====================================== NEW VERSION ==\n";
        print "== package $package=$version \n";
        print "==================================================\n";
        write_file(
            "./auto/mordax_version.pl",
            "\$mordax::version = '$version';\n"
        );

    },
    commit_with_new_version => qq{./auto/mordax_version.pl}
}
~~~ 

## Под репозитории: изменение инклудов

~~~ 
{
    watch_files          => [
        qr{^tmpl/skins},
        qr{^tmpl/skins/debian},
        ],
    ignore_files         => [
         ],
    after_new_version_cb => sub {
        my ( $packege, $version ) = @_;
        write_file(
            "../../auto/skins_version.pl",
            "\$mordax::skinsVersion = '$version';\n"
        );
        write_file(
            "./skins_version.js",
            "home.skinsVersion = '$version';\n"
        );
        rewrite_substvars("../../debian/substvars", 'skins', "portal-morda-tmpl-skins (=$version)");
    },
    commit_with_new_version => '../../auto/skins_version.pl ./skins_version.js ../../debian/substvars',
    commit_message          => 'HOME-0 skins changelog',
}
~~~

## Под репозитории: условное итерирование

~~~ 
{
    watch_files          => [
        qr{\.(styl|css|js|png|jpg|jpeg|svg|gif|iddqd)$},
        qr{^tmpl/debian},
        qr{\.data-image},
        qr{GNUmakefile},
        qr{Makefile},
        qr{^tmpl/js_libs/franky},
        qr{^tmpl/skins/skins_version\.js},
  	    qr{lang[^\.\/]*\.json},
	    qr{auto/Lang_auto.js},
    ],
    ignore_files         => [
        qr{(content|hash|page|config|index|skins|server)\.js$},
        sub {
            !m{bemjson\.js$} and
            m{tmpl/v12/pages-desktop/(?:[^\/]+)/[^\/]+\.js},
        },
        qr{priv\.js$},
        qr{^tmpl/[^\/]+.js$},
    ],
}
~~~

## Web
{:.center}

![](pictures/release-web.png)

## **Контакты** {#contacts}

<div class="info">
<p class="author">{{ site.author.name }}</p>
   
    <div class="contacts">
        <p class="contacts-left contacts-top">https://github.yandex-team.ru/morda/gitrelease</p>
        <p class="contacts-left mail">mellior@yandex-team.ru</p>
        
    </div>
</div>


