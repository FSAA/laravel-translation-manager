# Laravel Translation Manager

Note that this package is originally based on Barry vd. Heuvel's <barryvdh@gmail.com> excellent **barryvdh/laravel-translation-manager** package but heavily reworked to add [New Features](#NewFeatures).

For Laravel 4 use the Laravel4 branch, or require: `"vsch/laravel-translation-manager": "~1.0"`

For Laravel 5 use the master branch, or require: `"vsch/laravel-translation-manager": "~2.0"`

This is a package to manage Laravel translation files. It does not replace the Translation system but augments it with:

- import/export/download the php files to/from a database
- make translations editable through a web interface.
- allow in-database translations to override the ones in the language files. Used to update translations on server clusters where updating translation files is not possible (like AWS EC2) or would cause server code to be out of sync.
- assisted translation with Yandex API integrated into the web interface.

The workflow would be:

- Import translations: Read all translation files and save them in the database
- Find all translations in php/twig sources
- Optionally: Log missing translation
- Translate all keys through the web interface
- Export: Write all translations back to the translation files or cache if in production environment where writing of files is not an option.
- Download zipped translations. The zip archive will contain the files that would be exported to the app/lang directory in the same directory structure so it is easy to replace app/lang on your development system.

This way, translations can be saved in git history and no overhead is introduced in production.

## Table of Contents
- [Web Interface](#WebInterface)
- [Limitations](#Limitations)
- [Installation](#Installation)
- [Configuration](#Configuration)
- [Artisan Commands](#ArtisanCommands)
- [New Features](#NewFeatures)
- [Yandex Supported Languages](#YandexSupportedLanguages)
- [To Do](#ToDo)

<a id="WebInterface"></a>
## Web Interface

![Translator Page ](https://raw.githubusercontent.com/vsch/laravel-translation-manager/master/images/ScreenShot_main.png)

When you have imported your translation (via buttons or command), you can view them in the web interface (on the url you defined the with the controller).
You can click on a translation and an edit field will popup. All translations are saved when the edit dialog is closed unless it is closed with the cancel button. Clicking anywhere on the page outside the edit dialog will save the current changes.
When a translation is not yet created in a different locale, you can also just edit it to create it.

Using the buttons on the web interface, you can import/export the translations. For publishing translations, make sure your application can write to the language directory or optionally configure it to do in-database publishing using the cache by adding:

The web interface lets you select the locale for the web interface and also the locale that you are currently translating and the primary locale. This is only used to order the locale columns displayed in the translation table such that the primary locale is always listed first and the translating locale is second, followed by all the other locales.

You can select which additional locales to display on the page by checking the locales clicking on the 'Working Set' button to apply your selection.

### In Place Edit

The "In Place Edit" button will allow you to edit all Translation Manager web interface translation strings right in the page. If you want to try this mode you will need to copy the lang/{locale}/messages.php from the vendors/vsch/laravel-translation-manager/src directory to your project's app/lang/packages/{locale}/laravel-translation-manager directory for all the locales whose translations you want to override in your project. Following that you will need to import the translations using the 'import' button or the artisan import command.

If you do not copy the package language files then all translation edit links will display as Empty and editing them will create an override in your project for the translation when you 'Publish' the group 'laravel-translation-manager::messages' or 'Publish All'.

### Suffixed Key Operations & Search (panel)

Click on the panel to expand it.

![Suffixed Key Operations](https://raw.githubusercontent.com/vsch/laravel-translation-manager/master/images/ScreenShot_suffixed_keyops.png)

Enter keys you want to create/delete from the current group in the left text area, and optionally any suffixes you want to permute with the keys on the left. For example, keys: abc, def, ghi; with suffixes: 1,2,3 will create/delete: abc1, abc2, abc3, def1, def2, def3, ghi1, ghi2, ghi3.

Hit the search button to see the search dialog. The database translation strings and keys will be searched for the text and the matching entries displayed. You can edit the resulting translations directly in the search dialog or alternately follow the group link to display the group translation page.

![Search](https://raw.githubusercontent.com/vsch/laravel-translation-manager/master/images/ScreenShot_search.png)

### Wildcard Key Operations (panel)

Click on the panel to expand it.

Here you can copy/delete/move keys with an optional wildcard character *, but only as a suffix or a prefix. The corresponding source and destination strings must have the * use: as suffix, prefix or not used.

Hitting the preview button will let you see what the patterns match and what the resulting operation will affect. Copy or Move will not affect destination keys that already exist.

Example of preview/delete:

![Wildcard Key Operations 1](https://raw.githubusercontent.com/vsch/laravel-translation-manager/master/images/ScreenShot_wildcard_keyops1.png)

Example of preview/copy/move:

![Wildcard Key Operations 2](https://raw.githubusercontent.com/vsch/laravel-translation-manager/master/images/ScreenShot_wildcard_keyops2.png)

### Translation Helpers (panel)

Click on the panel to expand it.

Here you can translate between the primary locale and the translating locale.

![Translation Helpers](https://raw.githubusercontent.com/vsch/laravel-translation-manager/master/images/ScreenShot_translation_helpers.png)

<a id="Limitations"></a>
## Limitations

The package has only been tested with MySQL backend. No MySQL specific syntax is being used but no other backend has been tested with the package.

Translation helpers use Yandex. You can edit any locale, even if it is not supported by Yandex. However, automatic translation will not work for unsupported locales. See [Yandex Supported languages](#YandexSupportedLanguages)

For other limitations, please see [To Do List](#ToDo).

If someone contacts me with a request to prioritize a specific area I will do it sooner :).

<a id="Installation"></a>
## Installation

1. Require this package in your composer.json and run composer update (or run `composer require vsch/laravel-translation-manager:*` directly):

        "vsch/laravel-translation-manager": "~2.0"

2. After updating composer, add the ServiceProviders to the providers array in config/app.php and comment out the original TranslationServiceProvider:

        //Illuminate\Translation\TranslationServiceProvider::class,
        Vsch\UserPrivilegeMapper\UserPrivilegeMapperServiceProvider::class,
        Vsch\TranslationManager\ManagerServiceProvider::class,
        Vsch\TranslationManager\TranslationServiceProvider::class,
        Collective\Html\HtmlServiceProvider::class,

    The TranslationServiceProvider is an extension to the standard functionality and is required in order for the web interface to work properly. It is backward compatible with the existing Translator since it is a subclass of it and only overrides implementation for new features.

3. add the Facade to the aliases array in config/app.php:

        'Form'      => Collective\Html\FormFacade::class,
        'Html'      => Collective\Html\HtmlFacade::class,
        'UserCan'   => Vsch\UserPrivilegeMapper\Facade\Privilege::class,

4. You need to publish then run the migrations for this package:

        $ php artisan vendor:publish --provider="Vsch\TranslationManager\ManagerServiceProvider" --tag=migrations
        $ php artisan migrate

5. You need to publish the config file for this package. This will add the file `config/laravel-translation-manager.php`

        $ php artisan vendor:publish --provider="Vsch\TranslationManager\ManagerServiceProvider" --tag=config

6. You need to publish the web assets used by the translation manager web interface. This will add the assets to `public/vendor/laravel-translation-manager`

        $ php artisan vendor:publish --provider="Vsch\TranslationManager\ManagerServiceProvider" --tag=public

7. By default the web interface of the translation manager is available at `http://yourdomain.com/translations`. You can change this in the configuration file.

8. <a id="step8"></a>TranslationManager uses the vsch/user-privilege-mapper package that creates a mapping layer between your User model implementation and the need to test user privileges without knowing the implementation. You need to name privileges for the UserPrivilegeMapper via the Laravel macro mechanism. This should be done in the initialization files. A good place is the app/Providers/AppServiceProvider.php file, add the following to boot() function, if your User model has is_admin and is_editor attributes to identify users that have Admin and Editor privileges or just `return true` in both cases if you don't have any way of determining user privileges:

        \UserCan::macro("admin_translations", function ()
        {
            return ($user = Auth::user()) && $user->is_admin;
        });

        // return false to use the translator missing key lottery, true to always check missing keys for the user
        \UserCan::macro("bypass_translations_lottery", function ()
        {
            return ($user = Auth::user()) && ($user->is_admin || $user->is_editor);
        });

    In this example the User model implements two attributes: is_admin and is_editor. The admin user is allowed to manage translations: import, delete, export, etc., the editor user can only edit existing translations. However, both of these users will always log missing translation keys so that any missing translations will be visible to them instead of relying on the missing key lottery settings.

9. Yandex assisted translations requires setting the `yandex_translator_key` to your Yandex API key in the `config.php` file, it is free. See: <https://tech.yandex.com/translate/>

<a id="Configuration"></a>
## Configuration

The config file `app/config/packages/laravel-translation-manager/config.php` has comments that provide a description for each option. Note that when `admin_enabled` is set to false then translation management is limited to editing existing translations all other operations have to be done through the command line. Ideally, this option needs to be dynamic based on user privileges so that translators cannot delete translations or administer translations but admins can. See [step 8](#step8) above.

By default the primary locale is en. The primary locale determines what language is used for the source text for the translate button in the edit pop-up. When editing the text for the primary locale a button in the edit pop-up allows to convert the text of the key to a default label (chage - and _ to spaces, capitalize first letter of each word).

## Modifying the default View

To create your own custom version of the index.blade.php copy this file from the `vendor/vsch/laravel-translation-manager/src/views/` directory to `app/views/packages/vsch/laravel-translation-manager` directory. The package view directory also contains a `layouts/master.blade.php` file for a default layout. The intent is for you to provide your own master layout that the index.blade.php will extend so it can match your site's style.

<a id="ArtisanCommands"></a>
## Artisan Commands

You can also use the commands below.

### Import command

The import command will search through app/lang and load all strings in the database, so you can easily manage them.

        $ php artisan translations:import
    
Note: By default, only new strings are added. Translations already in the DB are kept the same. If you want to replace all values with the ones from the files, 
add the `--replace` (or `-R`) option: `php artisan translations:import --replace`

### Find translations in source

The Find command/button will look search for all php/twig files in the app directory, to see if they contain translation functions, and will try to extract the group/item names.
The found keys will be added to the database, so they can be easily translated.
This can be done through the web interface, or via an Artisan command.

        $ php artisan translations:find

### Export command

The export command will write the contents of the database back to app/lang php files.
This will overwrite existing translations and remove all comments, so make sure to backup your data before using.
Supply the group name to define which groups you want to publish.

        $ php artisan translations:export <group>

For example, `php artisan translations:export reminders` when you have 2 locales (en/nl), will write to `app/lang/en/reminders.php` and `app/lang/nl/reminders.php`

### Clean command

The clean command will search for all translation that are NULL and delete them, so your interface is a bit cleaner. Note: empty translations are never exported.

        $ php artisan translations:clean

### Reset command

The reset command simply clears all translation in the database, so you can start fresh (by a new import). Make sure to export your work if needed before doing this.

        $ php artisan translations:reset

<a id="NewFeatures"></a>
## New Features

These features were added to the original barryvdh/laravel-translation-manager package.

- translation manager web-interface is localized. Current version has English and Russian. Others can be easily added by adding package translation overrides in `app/lang/packages/{locale}/laravel-translation-manager/messages.php` files. If messages.php files are added for en and ru locales then their contents will override the language files included in the package.

- allows in-database translations to override the translations in files.

- publishing translations on production systems where updating translation files would only update a single server, can be configured to use the cache for serving up the modified translations. This allows translations to be updated live without having to redeploy a new version of the language files.

- Translation service can be put into 'in place edit' mode that enables editing of translations where they appear on the page.
  - this eliminates the need to peruse code to find the translation group/key combination that is used for the resulting string and then looking for it in the translation files or in the web interface. Simply enable in-place editign mode and click on the string you wish to edit.
  - This may require some editing of view files to handle: string values that should not be links because they are not shown or are used for HTML attribute values, `<button>` contents that don't display links and other edge cases.

- changes to database translations that have not been published show a difference between previously published/imported translations and current unpublished changes.

- soft delete of translations that are not physically deleted from the database until translations are published.

- translation page has a dash board view showing all unpublished changes, missing translations and deleted translations.

- handling of translation files in nested directories under `lang/` and package translations under `lang/packages` to allow managing package translation overrides.

- missing translation key logging that can be used in a production environment by setting 1 of N sessions to actually log missing translations. Since checking for missing translation requires hitting the database for every translation, it can be a heavy load on the DB server. This configuration setting allows randomly selecting 1 of N user sessions to be marked as checking for missing translations allowing the benefit of finding missing translations while reducing the load burden on the server.

- in place edit mode inside the search dialog to allow editing of translation in the search result.

- Yandex translation API for assisting in the translation process.

- extra buttons added to the bootstrap x-edit component for frequent operations:
  - change case of translation or selection within translation: lowercase, first cap
  - create plural forms for use in `choice()`, currently English is automatically created and Russian will do its best by using Yandex translator to derive the plural forms.
  - recall translation text from last saved or last published.
  - simulated copy/paste buttons. Work only if the page is not reloaded and only within the translation edit dialog.

- exported language files are formatted to align `=>` for a given level, making it easier to deal with these files manually if needed.

- exported language files preserve multi-line comments and doc comments and empty first level array values.

- moved the delete translation icon before the translation key instead of after the last translation because for large number of locales it was impossible to know which key was going to be marked for deletion.

- added working locale set to limit number of locales displayed on the page and to reduce page load time.

<a id="YandexSupportedLanguages"></a>
## Yandex Supported languages

You can find an up to date list here: <https://tech.yandex.com/translate/doc/dg/concepts/langs-docpage/>

<table>
    <thead>
        <tr><th>Language</th><th>Locale</th><th>Language</th><th>Locale</th><th>Language</th><th>Locale</th><th>Language</th><th>Locale</th></tr>
    </thead>
    <tbody>
        <tr><td>Albanian</td><td>sq</td><td>Dutch</td><td>nl</td><td>Italian</td><td>it</td><td>Russian</td><td>ru</td></tr>
        <tr><td>Arabian</td><td>ar</td><td>English</td><td>en</td><td>Japanese</td><td>ja</td><td>Spanish</td><td>es</td></tr>
        <tr><td>Armenian</td><td>hy</td><td>Estonian</td><td>et</td><td>Korean</td><td>ko</td><td>Serbian</td><td>sr</td></tr>
        <tr><td>Azeri</td><td>az</td><td>Finnish</td><td>fi</td><td>Latvian</td><td>lv</td><td>Slovak</td><td>sk</td></tr>
        <tr><td>Belarusian</td><td>be</td><td>French</td><td>fr</td><td>Lithuanian</td><td>lt</td><td>Slovenian</td><td>sl</td></tr>
        <tr><td>Bosnian</td><td>bs</td><td>Georgian</td><td>ka</td><td>Macedonian</td><td>mk</td><td>Swedish</td><td>sv</td></tr>
        <tr><td>Bulgarian</td><td>bg</td><td>German</td><td>de</td><td>Malay</td><td>ms</td><td>Thai</td><td>th</td></tr>
        <tr><td>Catalan</td><td>ca</td><td>Greek</td><td>el</td><td>Maltese</td><td>mt</td><td>Turkish</td><td>tr</td></tr>
        <tr><td>Croatian</td><td>hr</td><td>Hebrew</td><td>he</td><td>Norwegian</td><td>no</td><td>Ukrainian</td><td>uk</td></tr>
        <tr><td>Czech</td><td>cs</td><td>Hungarian</td><td>hu</td><td>Polish</td><td>pl</td><td>Vietnamese</td><td>vi</td></tr>
        <tr><td>Chinese</td><td>zh</td><td>Icelandic</td><td>is</td><td>Portuguese</td><td>pt</td><td></td><td><tr><td>Danish</td><td>da</td><td>Indonesian</td><td>id</td><td>Romanian</td><td>ro</td><td></td><td></tr>
    </tbody>
</table>

<a id="ToDo"></a>
## To Do List

This package is still in development although it is successfully being used to manage translations. Here is a list of to do's and limitations:

- MySQL DB is assumed for queries in those places where Eloquent was too cumbersome or too inefficient. I will be refactoring all DB access that bypasses Eloquent into a TranslationRepository interface class so that new DB access will only need to create a new repository implementation class for a specific DB interface.

- only Yandex assisted translations are implemented. Google translate was not used since it has no free option. However it is a simple change in the translator.js file to handle alternate translation engines. I will be making a configurable item in the future.

- key operations that allow creating new keys and also keys permuted by suffixes, moving, copying, deleting keys are a bit of a kludge. I am planning to rework the web interface to make these cleaner. However, if you desperately need these to save a lot of typing and editing, the current version will do the trick.

Suggestions and priority requests are welcome. :)
