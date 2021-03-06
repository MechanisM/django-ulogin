django-ulogin
=============

Django-ulogin является приложением для социальной аутентификации пользователей с помощью интернет-сервиса `ULOGIN.RU <http://ulogin.ru>`_

Требования
-----------
- Python 2.6 и выше;
- Django Framework версии 1.3.1;
- requests версии 0.7.4 или выше;
- mock версии 0.8.0 (используется только для запуска тестов).

Лицензия
--------
Распространяется по лицензии MIT


Установка
---------

Установка финальной версии ``django-ulogin`` производится утилитами ``pip``.

::

    $ pip install django-ulogin

или ``easy_install``:

::

    $ easy_install django-ulogin

Для установки dev-версии нужно воспользоваться ``pip``:

::

    $ pip install -e git+github.com/marazmiki/django-ulogin.git#egg=django-ulogin

Все внешние зависимости будут утановлены автоматически


Конфигурация проекта
--------------------

Для подключения к проекту откройте ``settings.py`` и добавьте в кортеж ``INSTALLED_APPS`` приложение ``django_ulogin``

:: 

    INSTALLED_APPS += ('django_ulogin', )

Для работы django-ulogin необходим подключенный контекст-процессор ``django.core.context_processors.request``. Обратите внимание, что по умолчанию в файле ``settings.py`` параметр ``TEMPLATE_CONTEXT_PROCESSORS`` отсутствует. Добавьте следующие строки в ``settings``, если их там нет:

::

    TEMPLATE_CONTEXT_PROCESSORS = (
        'django.contrib.auth.context_processors.auth',
        'django.core.context_processors.debug',
        'django.core.context_processors.i18n',
        'django.core.context_processors.media',
        'django.core.context_processors.static',
        'django.core.context_processors.request',
        'django.contrib.messages.context_processors.messages',
    )

Добавьте схему URL-адресов к списку ``urlpatterns`` Вашего проекта (``urls.py``):

::

    urlpatterns += patterns('',
        url(r'^ulogin/', include('django_ulogin.urls')),
    )

Затем следует синхронизировать базу данных

::

    $ ./manage.py syncdb


Использование
-------------

Для использования приложения достаточно в любом месте шаблона вставить подключение шаблонной библиотеки ``ulogin_tags`` и вызов тега ``ulogin_widget``.

::

    {% load ulogin_tags %}
    {% ulogin_widget %}

На месте тега ``ulogin_widget`` при рендеринге появится код интеграции Вашего сайта c ULOGIN.


Тег ``{% ulogin_widget %}`` принимает один необязательный аргумент - ``scheme_name``, который указывает на имя используемой схемы настроек.

::

    {% ulogin_widget "scheme_name" %}

Использование различных схем особенно удобно, если нужно на одной странице разместить несколько виджетов, обладающих различными настройками.


Тонкая настройка
----------------

По умолчанию ``django_ulogin`` требует от сервиса только одно обязательное поле - ``email``. Вы можете указать для проекта список как необходимых полей (определив в ``settings`` список ``ULOGIN_FIELDS``), так и опциональных (``ULOGIN_OPTIONAL``):

::
    
    # Поля first_name и last_name обязательны
    ULOGIN_FIELDS = ['first_name', 'last_name']

    #  Необязательные поля: пол, URL аватара, дата рождения
    ULOGIN_OPTIONAL = ['sex', 'photo', 'bdate'] 

Список всех полей, которые сообщает ULOGIN:

- first_name
- last_name
- email
- nickname
- bdate *(дата рождения, передаётся в формате dd.mm.yyyy)*
- sex *(пол: 1 означает женский, 2 - мужской)*        
- photo *(аватар, размер 100х100 пикселей)*    
- photo_big  
- city
- country
- phone

Внешний вид виджета определяется параметром ``ULOGIN_DISPLAY``. Доступно три варианта:

- panel
- small *(по умолчанию)*
- button

Список используемых провайдеров определяется директивой ``ULOGIN_PROVIDERS``. По умолчанию включены:

- vkontakte
- facebook
- twitter
- google
- livejournal

Дополнительные провайдеры, которые будут показаны внутри выпадающего меню, определяются в директиве ``ULOGIN_HIDDEN``. По умолчанию:

- yandex
- odnoklassniki
- mailru
- openid

Если при входе нужно выполнить какую-то JavaScript-функцию, укажите её в виде строки в переменной ``ULOGIN_CALLBACK``.

Схемы
-----

Как упоминалось выше, в некоторых случаях нужно разместить на одной странице несколько виджетов ulogin с различными настройками. В этом случае целесообразно создать нужное количество схем и настроить их.

Схемы определяются как словарь ``ULOGIN_SCHEMES``, ключи которого - названия схем, используемые в шаблонном теге ``{% ulogin_widget "scheme_name" %}``, а значения - словари с настройками. 

Ключи этого словаря совпадают с названиями соответствующих "глобальных" настроек, но без префикса ``ULOGIN_``. Это означает, что в пределах настройки схемы ключ ``DISPLAY`` будет отвечать за вид панели виджета, как и его глобальный "коллега" ```ULOGIN_DISPLAY`` 

Кроме того, настройки схем наследуют глобальные настройки. Например, такая настройка:

::

    ULOGIN_PROVIDERS = ['google', 'twitter']
    ULOGIN_HIDDEN = ['odnoklassniki', 'mailru']
    ULOGIN_DISPLAY = 'panel'

    ULOGIN_SCHEMES = {
        'default': {'HIDDEN': ['yandex']},
        'comments': {'DISPLAY': 'small'}
    }

означает, что по умолчанию включены провайдеры ``google`` и ``twitter``, ``odnoklassniki`` и ``mailru`` скрыты, а виджет выводится в раскладке ``panel``.

Однако при использовании схемы ``default`` скрытым провайдером окажется ``yandex``, а схема ``comments`` будет выведена в раскладке ``small``. Настройки, которые не переопределены, будут браться из глобальной области.

Сигналы
-------

При аутентификации пользователя создаётся новый Django-пользователь, ``username`` которого заполняется uuid4-хешем. Однако при создании новой аутентификации срабатывает сигнал ``django_ulogin.signals.assign``, в котором передаётся объект ``request``, пользователь Django, аутентификация и флаг ``registered`` , показывающий, была ли создана запись.

Чтобы сделать имя поля дружественным пользователю, достаточно создать объект, подписанный на сигннал ``django_ulogin.signals.assign``:

::

    def catch_ulogin_signal(*args, **kwargs):
        """
        Обновляет модель пользователя: исправляет username, имя и фамилию на 
        полученные от провайдера.

        В реальной жизни следует иметь в виду, что username должен быть уникальным,
        а в социальной сети может быть много "тёзок" и, как следствие,
        возможно нарушение уникальности.

        """
        user=kwargs['user']
        json=kwargs['ulogin_data']

        if kwargs['registered']:
            user.username=json['username']
            user.first_name=json['first_name']
            user.last_name=json['last_name']
            user.email=json['email']
            user.save()

    from django_ulogin.models import ULoginUser

    assign.connect(receiver = catch_ulogin_signal,
                   sender   = ULoginUser,
                   dispatch_uid = 'customize.models')


Можно изучить тестовый проект, в котором реализована функция сохранения данных, полученных от ULogin:

- https://github.com/marazmiki/django-ulogin/tree/master/test_project
- https://github.com/marazmiki/django-ulogin/blob/master/test_project/customize/models.py#L47
