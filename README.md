# Magento 2 Clientside Cache
Magento already had support for ESI blocks, this will add an easy way to replace page heavy or slow blocks and
load them asynchronous without much hassle.

## Installation
```bash
composer require elgentos/magento2-clientside-cache
bin/magento setup:upgrade
```

## Usage
The easiest way is to add the `<block class="Elgentos\ClientsideCache\Block\Async" />`
around the original block, and use the correct alias `as=""` so the original block knows how which `getChildHtml` to fetch.

`layout/LAYOUT_HANDLE.xml`
```xml

<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>

        <!-- Just add the wrapper around the original block -->
        <block class="Elgentos\ClientsideCache\Block\Async" as="moved-the-alias-name-to-here">
            <block name="heavy-block-name" template="Magento_HeavySlowThingy::path/to/file.phtml"/>
        </block>

        <!-- Or if you have something existing which you don't want to touch or can the original XML -->

        <!-- Create a wrapper block -->
        <block name="topmenu_mobile-replacement" class="Elgentos\ClientsideCache\Block\Async"/>
        <!-- Move the replacement into the orignal location, use before original block name when working with containers or as when working with blocks -->
        <move element="topmenu_mobile-replacement" destination="topmenu_generic" before="topmenu_mobile"
              as="topmenu.mobile"/>
        <!-- Move the heavy block into into the wrapper -->
        <move element="topmenu_mobile" destination="topmenu_mobile-replacement"/>
    </body>
</page>
```

### Limit handles
You can limit the handles within the Async wrapper. This increases the possibility to re-use requests.
An example would be menu's which are the same on every page.

`layout/default.xml`
```xml

<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <!-- This will only use the default handle -->
        <block class="Elgentos\ClientsideCache\Block\Async" as="moved-the-alias-name-to-here">
            <argument name="handles" xsi:type="array">
                <item name="default" xsi:type="string">default</item>
            </argument>
        </block>
    </body>
</page>
```

## Features
- Fetch content async
- Group similar requests for handles into on one single request
- Evaluate Javascript

## Javascript
If you want to make sure that your global functions or objects are available you need to make sure to assign them to global scope.
Some examples;

```html
<script>
const someFunction = () => {};
// add
window.someFunction = someFunction;

// and for functions
function helloWorld() {}
// add
window.helloWorld = helloWorld;
</script>
```

## Custom integration
It's also possible to do custom integration too.

Use `clientsideCacheAsync` which will return a Promise and will with a object for all blocks you requested.
```javascript
clientsideCacheAsync(['block_name', 'other_block'], ['default', 'catalog_category_view']).then(results => console.log(results))
// {block_name: "", "other_block": ""}
clientsideCacheAsync(['another_block'], ['default', 'catalog_category_view']).then(results => console.log(results))
// {another_block: ""}
```

## Special thanks
The module was created during the Hackthon before the [Mage Unconference in Cologne](https://www.mageunconference.org/).
Thanks to the team for organising another great event and for the Hackathon.

## Author
- [Jeroen Boersma](https://github.com/JeroenBoersma)