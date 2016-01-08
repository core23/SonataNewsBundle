Installation
============


Prerequisites
-------------

PHP 5.3 and Symfony 2 are needed to make this bundle work ; there are also some
Sonata dependencies that need to be installed and configured beforehand:

    - SonataAdminBundle_
    - SonataUserBundle_
    - SonataMediaBundle_
    - SonataClassificationBundle_
    - SonataFormatterBundle_
    - SonataEasyExtendsBundle_
    - SonataIntlBundle_
    - SonataDatagridBundle_
    - SonataCoreBundle_

Follow also their configuration steps; you will find everything you need in their installation chapter.

.. note::

    If a dependency is already installed somewhere in your project or in
    another dependency, you won't need to install it again.

Enable the Bundle
-----------------

.. code-block:: bash

    $ php composer.phar require sonata-project/news-bundle --no-update
    $ php composer.phar require sonata-project/doctrine-orm-admin-bundle  --no-update # optional
    $ php composer.phar require friendsofsymfony/rest-bundle  --no-update # optional when using api
    $ php composer.phar require nelmio/api-doc-bundle  --no-update # optional when using api
    $ php composer.phar update

.. note::

    The SonataAdminBundle and SonataDoctrineORMAdminBundle must be installed, please refer to `the dedicated documentation for more information <https://sonata-project.org/bundles/admin>`_.

    The `SonataDatagridBundle <https://github.com/sonata-project/SonataDatagridBundle>`_ must be added in ``composer.json``.

Next, be sure to enable the ``News`` and ``EasyExtends`` bundles in your application kernel:

.. code-block:: php

    <?php

    // app/AppKernel.php
    public function registerBundles()
    {
        return array(
            // ...
            new Sonata\NewsBundle\SonataNewsBundle(),
            new Sonata\EasyExtendsBundle\SonataEasyExtendsBundle(),
            // ...
        );
    }

Before we can go on with generating our Application files trough the `EasyExtendsBundle`_, we need to add some lines which we will override later (we need them now only for the following step):

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        sonata_news:
            title:        Sonata Project
            link:         https://sonata-project.org
            description:  Cool bundles on top of Symfony2
            salt:         'secureToken'
            permalink_generator: sonata.news.permalink.date # sonata.news.permalink.collection

            comment:
                notification:
                    emails:   [email@example.org, email2@example.org]
                    from:     no-reply@sonata-project.org
                    template: 'SonataNewsBundle:Mail:comment_notification.txt.twig'

Configuration
-------------

To use the ``PageBundle``, add the following lines to your application
configuration file.

.. note::

    If your ``auto_mapping`` have a ``false`` value, add these lines to your
    mapping configuration :

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        doctrine:
            orm:
                entity_managers:
                    default:
                        #metadata_cache_driver: apc
                        #query_cache_driver: apc
                        #result_cache_driver: apc
                        mappings:
                            ApplicationSonataNewsBundle: ~
                            SonataNewsBundle: ~

* import the ``sonata_news.yml`` file and enable json type for doctrine:

.. code-block:: yaml

    imports:
        #...
        - { resource: sonata_news.yml }
    #...
    doctrine:
        dbal:
        # ...
            types:
                json: Sonata\Doctrine\Types\JsonType

* Add a new context into your ``sonata_media.yml`` configuration if you don't have go there https://sonata-project.org/bundles/media/master/doc/reference/installation.html:

.. code-block:: yaml

    news:
        providers:
            - sonata.media.provider.dailymotion
            - sonata.media.provider.youtube
            - sonata.media.provider.image

        formats:
            small: { width: 150 , quality: 95}
            big:   { width: 500 , quality: 90}

* create configuration file ``sonata_formatter.yml`` the text formatters available for your blog post:

.. code-block:: yaml

    sonata_formatter:
        formatters:
            markdown:
                service: sonata.formatter.text.markdown
                extensions:
                    - sonata.formatter.twig.control_flow
                    - sonata.formatter.twig.gist
                    - sonata.media.formatter.twig

            text:
                service: sonata.formatter.text.text
                extensions:
                    - sonata.formatter.twig.control_flow
                    - sonata.formatter.twig.gist
                    - sonata.media.formatter.twig

            rawhtml:
                service: sonata.formatter.text.raw
                extensions:
                    - sonata.formatter.twig.control_flow
                    - sonata.formatter.twig.gist
                    - sonata.media.formatter.twig

            richhtml:
                service: sonata.formatter.text.raw
                extensions:
                    - sonata.formatter.twig.control_flow
                    - sonata.formatter.twig.gist
                    - sonata.media.formatter.twig


* Run the easy-extends command:

.. code-block:: bash

    php app/console sonata:easy-extends:generate SonataNewsBundle -d src
    php app/console sonata:easy-extends:generate SonataUserBundle -d src
    php app/console sonata:easy-extends:generate SonataMediaBundle -d src
    php app/console sonata:easy-extends:generate SonataClassificationBundle -d src

* Enable the new bundles:

.. code-block:: php

    <?php

    // app/AppKernel.php
    public function registerBundles()
    {
        return array(
            // ...
            new Application\Sonata\NewsBundle\ApplicationSonataNewsBundle(),
            new Application\Sonata\UserBundle\ApplicationSonataUserBundle(),
            new Application\Sonata\MediaBundle\ApplicationSonataMediaBundle(),
            new Application\Sonata\ClassificationBundle\ApplicationSonataClassificationBundle(),
            // ...
        );
    }

Update database schema by running command ``php app/console doctrine:schema:update --force``

* Complete the FOS/UserBundle install and use the ``Application\Sonata\UserBundle\Entity\User`` as the user class

* Add SonataNewsBundle routes to your application routing.yml:

.. code-block:: yaml

    # app/config/routing.yml
    news:
        resource: '@SonataNewsBundle/Resources/config/routing/news.xml'
        prefix: /news

Enable the extended Bundle
--------------------------

.. code-block:: php

    <?php
    // app/AppKernel.php

    public function registerBundles()
    {
        return array(
            // ...

            // Application Bundles
            new Application\Sonata\NewsBundle\ApplicationSonataNewsBundle(),

            // ...
        );
    }

And now, you're good to go !


.. _SonataAdminBundle: https://sonata-project.org/bundles/admin
.. _SonataUserBundle: https://sonata-project.org/bundles/user
.. _SonataMediaBundle: https://sonata-project.org/bundles/media
.. _SonataClassificationBundle: https://sonata-project.org/bundles/classification
.. _SonataFormatterBundle: https://sonata-project.org/bundles/formatter
.. _SonataEasyExtendsBundle: https://sonata-project.org/bundles/easy-extends
.. _SonataIntlBundle: https://sonata-project.org/bundles/intl
.. _SonataDatagridBundle: https://sonata-project.org/bundles/datagrid
.. _EasyExtendsBundle: https://sonata-project.org/bundles/easy-extends/master/doc/index.html
.. _SonataCoreBundle: https://sonata-project.org/bundles/core
