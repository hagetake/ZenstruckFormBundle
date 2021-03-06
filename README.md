# ZenstruckFormBundle

Provides Twitter Bootstrap form theme, useful FormType Extensions and javascript helpers.

[View Example Source Code](https://github.com/kbond/sandbox)

## Installation

1. Add to your `composer.json`:

    ```
    composer require zenstruck/form-bundle:~1.4
    ```

2. *Optional* If using the `ajax_entity_controller` feature, add `zendframework/zend-crypt` to your `composer.json`:

    ```
    composer require zendframework/zend-crypt:~2.0,!=2.1.1
    ```

    **Note:** Version 2.1.1 of `zend-crypt` does not have it's autoloader configured correctly.

3. *Optional* If using the Grouped form feature, add
[cocur/slugify](https://github.com/cocur/slugify#symfony2) to your `composer.json`

    ```
    composer require cocur/slugify:~0.8
    ```

4. Register the bundle with Symfony2:

    ```php
    // app/AppKernel.php

    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Zenstruck\Bundle\FormBundle\ZenstruckFormBundle(),

            // enable if you want to use the grouped form
            // new Cocur\Slugify\Bridge\Symfony\CocurSlugifyBundle()
        );
        // ...
    }
    ```
5. If using 'Select2', be sure to download the required files from http://ivaynberg.github.io/select2/ and include the files in your template.

    ```
    //base.html.twig Example

    //...
    {% block stylesheets %}
        <link href="{{ asset('path/to/select2.css') }}" type="text/css" rel="stylesheet" />

    //...
    {% block javascripts %}
        <script type="text/javascript" src="{{ asset('path/to/select2.js') }}"></script>
        <script type="text/javascript" src="{{ asset('path/to/zenstruckform/js/helper.js') }}"></script>
        <script type="text/javascript" src="{{ asset('path/to/zenstruckform/js/script.js') }}"></script>
    ```


## Twitter Bootstrap form layout

To use, do one of the following:

- Add for a single template:

    ```jinja
    {# for bootstrap 2.x #}
    {% form_theme form 'ZenstruckFormBundle:Twitter:form_bootstrap_layout.html.twig' %}

    {# for bootstrap 3.x #}
    {% form_theme form 'ZenstruckFormBundle:Twitter:form_bootstrap3_layout.html.twig' %}
    ```

- Add globally in your `config.yml`:

    ```yaml
    twig:
        form:
            resources:
                # for bootstrap 2.x
                - 'ZenstruckFormBundle:Twitter:form_bootstrap_layout.html.twig'

                # for bootstrap 3.x
                - 'ZenstruckFormBundle:Twitter:form_bootstrap3_layout.html.twig'
    ```

## FormType Extensions

### AjaxEntityType

![AjaxEntityType screenshot](https://lh3.googleusercontent.com/-qH5_q34yrjc/URvBEa_eydI/AAAAAAAAKEY/Yywbz7A2OqA/s384/ajax-entity.jpg)

Creates a `1-m` or `m-m` entity association field.  This type simply creates a hidden field that takes
an either 1 or multiple comma separated entity ids. **Note:** Ensure the entity has `__toString()` defined.

Enable in your `config.yml` (disabled by default):

```yaml
zenstruck_form:
    form_types:
        ajax_entity: true
```

There are several ways to use this type:

1. Default - creates a hidden field type.  It is up to the user to add functionality.

    ```php
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class MyFormType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('name', 'zenstruck_ajax_entity', array(
                    'class' => 'AppBundle:MyEntity' // ensure MyEntity::__toString() is defined
                ))
            ;
        }

        // ...
    }
    ```

2. Select2 with built in entity finder (`zendframework/zend-crypt` required):

    Enable the controller in your `config.yml` (disabled by default):

    ```yaml
    zenstruck_form:
        form_types:
            ajax_entity_controller: true
    ```

    Add the route to your `routing.yml`:

    ```yaml
    zenstruck_form:
        resource: "@ZenstruckFormBundle/Resources/config/ajax_entity_routing.xml"
    ```

    Add to your form type:

    ```php
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class MyFormType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('name', 'zenstruck_ajax_entity', array(
                    'class'             => 'AppBundle:MyEntity', // ensure MyEntity::__toString() is defined
                    'use_controller'    => true,
                    'property'          => 'name', // the entity property to search by
                    // 'repo_method'    => 'findActive' // for using a custom repository method
                    // 'extra_data'     => array() // for adding extra data in the ajax request (only applicable when using repo_method)
                ))
            ;
        }

        // ...
    }
    ```

    **Note:** The URL is dynamically generated for each entity but is encrypted with the application's `secret` for
     security purposes.

3. Select2 with custom URL.  This will create a Select2 widget for this field.

    ```php
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class MyFormType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('name', 'zenstruck_ajax_entity', array(
                    'class' => 'AppBundle:MyEntity', // ensure MyEntity::__toString() is defined
                    'url' => '/myentity/find'
                ))
            ;
        }

        // ...
    }
    ```

    The url endpoint receives the search string as a `q` request parameter and must return a json encoded array.
    Here is an example:

    ```json
    [
        {"id":2004,"text":"dolorem"},
        {"id":2008,"text":"inventore"}
    ]
    ```

#### FormType options

* `class`: The entity the field represents. *Required.*
* `url`: The url that Select2 will send search queries to
* `property`: The entity property to search by (Overrides `url`)
* `method`: The custom repository method to call for searches (Overrides `property`)
* `placeholder`: The Select2 placeholder text. Default: *Choose an option*
* `multiple`: Whether this is allows for multiple values. Default: *false*
* `use_controller`: Whether to use the bundled controller or not (``).  Default: *false*
* `repo_method`: For using a custom repository method. Default: *null*
* `extra_data`: For adding extra data in the ajax request (only applicable when using repo_method). Default *array()*

#### Select2 Javascript Helper

Enables the [Select2](http://ivaynberg.github.com/select2/) widget for `AjaxEntityType`.  Requires
[Select2](https://github.com/ivaynberg/select2/tags).

Enable with `ZenstruckFormHelper.initSelect2Helper()`

### TunnelEntityType

![TunnelEntityType screenshot](https://lh3.googleusercontent.com/-G4TtaRInANM/URvBEjb541I/AAAAAAAAKEc/tPOlE47Yj_s/s423/entity-tunnel.jpg)

Creates an entity association field with a select button. A javascript callback for the select button may be defined.
Can be used for opening a dialog to choose an entity.

1. Enable in your `config.yml` (disabled by default):

    ```yaml
    zenstruck_form:
        form_types:
            tunnel_entity: true
    ```

2. Add help option to your form fields

    ```php
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class MyFormType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('name', 'zenstruck_tunnel_entity', array(
                    'class' => 'AppBundle:MyEntity',
                    'callback' => 'MyApp.selectMyEntity',
                    'required' => false
                ))
            ;
        }

        // ...
    }
    ```

The widget html generated by the above example is as follows:

```html
<div class="input-append zenstruck-tunnel-widget">
    <input type="hidden" class="zenstruck-tunnel-id" />
    <span class="uneditable-input zenstruck-tunnel-title">{{ title }}</span>
    <a href="#" class="btn zenstruck-tunnel-clear"><b class="icon-remove"></b></a>
    <a href="#" class="btn zenstruck-tunnel-select" data-callback="MyApp.selectMyEntity">Select...</a>
</div>
```

Your javascript can hook into the clear button and select button.  Here are the useful classes:

* `.zenstruck-tunnel-id`: id of the selected entity
* `.zenstruck-tunnel-title`: title of the selected entity
* `.zenstruck-tunnel-clear`: button that clears the title/id (only available if `required` is `false`)
* `.zenstruck-tunnel-select`: button that initiates the entity selection

#### FormType options

* `class`: The entity the field represents. *Required.*
* `callback`: The javascript callback
* `button_text`: The text for the select button.  Default: *Select...*

#### Tunnel Javascript Helper

Adds events to the clear and select buttons.  The select button calls the `callback` defined in the type options.
The callback receives the following parameters:

- `id`: the id of the currently selected entity (if any)
- `element`: the hidden input element

Enable with `ZenstruckFormHelper.initTunnelHelper()`

### HelpType

Allow you to add help messages to your form fields.

1. Enable in your `config.yml` (disabled by default):

    ```yaml
    zenstruck_form:
        form_types:
            help: true
    ```

2. Add help option to your form fields

    ```php
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class MyFormType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('name', 'text', array(
                    'help' => 'Your full name'
                ))
            ;
        }

        // ...
    }
    ```

### Group Type

![Group Type screenshot](https://lh6.googleusercontent.com/-LpFkVgg71nc/UWbRMFGn8cI/AAAAAAAAKFc/XBtlO4b4Uok/s433/form-group.jpg)

This type allows you group large forms into tabs.

1. Enable in your `config.yml` (disabled by default):

    ```yaml
    zenstruck_form:
        form_types:
            group: true
    ```

2. Add help option to your form fields

    ```php
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class MyFormType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('name', 'text', array(
                    'group' => 'Foo'
                ))
                ->add('name', 'text', array(
                    'group' => 'Bar'
                ))
            ;
        }

        // ...
    }
    ```

    **Note:** fields without a group will be in the first, default tab.

3. When creating your form view in your controller, wrap it with `Zenstruck\Bundle\FormBundle\Form\GroupedFormView`

    ```php
    class MyController extends Controller
    {
        public function newAction(Request $request)
        {
            // ...
            return array(
                'grouped_form' => new \Zenstruck\Bundle\FormBundle\Form\GroupedFormView($form->createView())
            );
        }
    }
    ```

    **Note:** to name your default tab to something other than *Default*, pass it as the second parameter
     to the `GroupedFormView` constructor.

4. In your template, include `grouped_form.html.twig` to render the form.

    ```html+jinja
    <form method="post" {{ form_enctype(grouped_form.form) }}>
        {% include 'ZenstruckFormBundle:Twitter:grouped_form.html.twig' with { 'grouped_form': grouped_form } %}
    </form>
    ```

    **Note:** to use the wrapped form, use `grouped_form.form`

#### Add custom data to `GroupedFormView`

```php
// ..
$groupedForm = new \Zenstruck\Bundle\FormBundle\Form\GroupedFormView($form->createView());
$groupedForm->setData('foo', 'bar');
```

In your template:

```html+jinja
{# ... #}
{{ grouped_form.data('foo') }} {# returns bar #}
```

#### Custom group order

```php
$groupedForm = new GroupedFormView($form->createView(), 'Default', array(
    'Bar', 'Foo', 'Default'
));
```

### Theme Type

Allow you to add theme options to your form fields.  The `theme_options` variable will be
available in your form theme.  *The bootstrap3 theme currently utilizes.*

1. Enable in your `config.yml` (disabled by default):

    ```yaml
    zenstruck_form:
        form_types:
            theme: true
    ```

2. Set default theme options in your `config.yml`

    ```yaml
    zenstruck_form:
        theme_options:
            control_width: col-md-4
            label_width: col-md-2
            # ...
    ```

3. Set theme options on a field in your form

    ```php
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class MyFormType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('name', 'text', array(
                    'theme_options' => array('control_width' => 'col-md-6')
                ))
            ;
        }

        // ...
    }
    ```

## Miscellaneous Javascript helpers

This bundle comes with a set of useful javascript helpers.  To enable, add the following javascipt file (or add to your
assetic javascripts):

```html+jinja
<script type="text/javascript" src="{{ asset('bundles/zenstruckform/js/helper.js') }}"></script>
```

Initialize all helpers with:

```js
$(function() {
    ZenstruckFormHelper.initialize();
});
```

### PostLinkHelper

Allows a standard `<a>` tag to become a method="POST" link.  Add the class `method-post`, `method-post-confirm`
or `method-delete` to an `<a>` tag for it's href value to become a POST link.

- `method-post`: standard post link (no confirmation)
- `method-post-confirm`: `method-post` with a confirmation dialog that is customizable via the `data-message` attribute
- `method-delete`: cross browser compatible DELETE link with a "Are you sure you want to delete?" confirmation dialog

Enable with `ZenstruckFormHelper.initPostLinkHelper()`

### FormCollectionHelper

Adds Symfony2 form collection 'add' and 'delete' button functionality.  See the
[Symfony2 docs](http://symfony.com/doc/current/cookbook/form/form_collections.html).  This works out of the box when
using the `form_bootstrap_layout.html.twig` form layout provided by this bundle.

**Note:** Do not add the javascript provided in the [Symfony2 cookbook article](http://symfony.com/doc/current/cookbook/form/form_collections.html)

##

Enable with `ZenstruckFormHelper.initFormCollectionHelper()`

## Full default config

```yaml
zenstruck_form:
    form_types:
        help:                   false
        group:                  false
        tunnel_entity:          false
        ajax_entity:            false
        ajax_entity_controller: false
```
