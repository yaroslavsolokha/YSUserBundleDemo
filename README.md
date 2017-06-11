YSUserBundle
=======
Inside - FOSuserbundle, SonataAdminBundle, SonataUserBundle, HWIOAuthBundle, Materialize, Bower.

##### 1. Add to parameters.yml
```
parameters:
    database_host: 172.18.0.1
    database_port: null
    database_name: ysuserbundle
    database_user: root
    database_password: root
    mailer_transport: smtp
    mailer_host: 127.0.0.1
    mailer_user: yaroslav.solokha@gmail.com
    mailer_password: null
    secret: 8ed60d15706d13261ee6705544edc13db3b61711
    facebook_client_id: xxx
    facebook_client_secret: xxx
    google_client_id: xxx
    google_client_secret: xxx
```
##### 2. Create DB
```
$ bin/console doctrine:database:create
```
##### 3. Add to composer
```
require": {
    ...
    "friendsofsymfony/user-bundle": "^2.0",
    "sonata-project/admin-bundle": "^3.16",
    "sonata-project/doctrine-orm-admin-bundle": "^3.1",
    "hwi/oauth-bundle": "^0.5.3",
    "egeloen/ordered-form-bundle": "^3.0",
    "sonata-project/user-bundle": "dev-add_support_for_fos_user2"
}
```
##### 4. Update
```
$ cd server
$ docker exec -it php /bin/sh
$ cc ysuserbundle
$ composer update
```
##### 5. Add to AppKernel.php
```
$bundles = [
    ...
    new FOS\UserBundle\FOSUserBundle(),
    new Sonata\UserBundle\SonataUserBundle('FOSUserBundle'),
    new HWI\Bundle\OAuthBundle\HWIOAuthBundle(),
    // These are the other bundles the SonataAdminBundle relies on
    new Knp\Bundle\MenuBundle\KnpMenuBundle(),
    new Sonata\CoreBundle\SonataCoreBundle(),
    new Sonata\BlockBundle\SonataBlockBundle(),
    // And finally, the storage and SonataAdminBundle
    new Sonata\DoctrineORMAdminBundle\SonataDoctrineORMAdminBundle(),
    new Sonata\AdminBundle\SonataAdminBundle(),
    new Ivory\OrderedFormBundle\IvoryOrderedFormBundle(),
    new YS\UserBundle\YSUserBundle()
];
```
##### 6. Add import to config.yml
```
imports:
    ...
    - { resource: "@YSUserBundle/Resources/config/config.yml" }
```
##### 7. Add to security.yml
```
# To get started with security, check out the documentation:
# http://symfony.com/doc/current/security.html
security:

    encoders:
        Symfony\Component\Security\Core\User\User: plaintext
        FOS\UserBundle\Model\UserInterface: sha512

    role_hierarchy:
        ROLE_SONATA_ADMIN: ROLE_USER
        ROLE_ADMIN: ROLE_SONATA_ADMIN
        ROLE_SUPER_ADMIN: ROLE_ADMIN

    providers:
        fos_userbundle:
            id: fos_user.user_provider.username

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            pattern: ^/
            form_login:
                provider: fos_userbundle
                login_path: /login
                check_path: /login_check
            oauth:
                resource_owners:
                    facebook: "/login/check-facebook"
                    gplus: "/login/check-google"
                login_path:  /login
                oauth_user_provider:
                    service: ys.user.provider
            logout:       true
            anonymous:    true

    access_control:
        - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/, role: ROLE_ADMIN }
```
##### 8. Add to routing.yml
```
...
ys_user_bundle:
    resource: "@YSUserBundle/Resources/config/routing.yml"
```
##### 9. Add translator to config.yml
```
framework:
    translator: { fallbacks: ['%locale%'] }
```
##### 10. Create schema
```
$ bin/console doctrine:schema:create
```
##### 11. Add to src - https://github.com/yaroslavsolokha/YSUserBundle
##### 12. bin/console assets:install
##### 13. bin/console fos:user:create admin --super-admin
##### 14. Go ysuserbundle:dev:8000/app_dev.php/login
##### 15. Extend Bundle layout for index page, replace app/Resources/views/default/index.html.twig to:
```
{% extends 'YSUserBundle::base.html.twig' %}
{% block body %}
    <div class="main container">
        <h1>YSUserBundle</h1>
        <ul>
            <li><a href="{{ path('sonata_admin_dashboard') }}">Admin</a></li>
            <li><a href="{{ path('fos_user_security_login') }}">Sign In</a></li>
            <li><a href="{{ path('fos_user_registration_register') }}">Sign Up</a></li>
            <li><a href="{{ path('fos_user_resetting_request') }}">Forgot password?</a></li>
            <li><a href="{{ path('fos_user_profile_show') }}">Show profile</a></li>
            <li><a href="{{ path('fos_user_profile_edit') }}">Edit profile</a></li>
            <li><a href="{{ path('fos_user_security_logout') }}">Sign Out</a></li>
        </ul>
    </div>
{% endblock %}
```
##### 16. For adding custom menu link please add:
``` 
{% extends 'YSUserBundle::base.html.twig' %}
{% block customMenu %}
    <li class="{{ app.request.attributes.get('_route') == 'routeName' ? 'active'><a href="#">Link Name</a></li>
{% endblock %}
```