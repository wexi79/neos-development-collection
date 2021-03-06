.. _adjusting-output:

=====================
Adjusting Neos Output
=====================

Page Template
=============

The page template defines the overall structure of the generated markup: what is
rendered in the body and head of the resulting document.

The Body
--------

As briefly explained in :ref:`page-rendering` the path to your own template for the
``body`` of a generated page can be set using TypoScript::

	page = Page
	page.body.templatePath = 'resource://My.Package/Private/Templates/PageTemplate.html'

The file will the be used to render the body content and any Fluid placeholders will be
substituted, ViewHelpers will be executed. Since no further information is given
to the rendering process, the full content of the template will be used for the body.

If the template contains a full HTML page, this will lead to invalid markup. But in
most cases having the template as a full HTML document is desired, as it allows easy
handling by the developer and can be previewed as is in a browser.

To use just a part of the document for the body, that part can simply be enclosed in
a Fluid section::

	<!DOCTYPE html>
	<html>
	<head>
		…
	</head>
	<body>
	<f:section name="body">
		<h1>{title}</h1>
	</f:section>
	</body>
	</html>

The TypoScript is then amended with the declaration of the section to use::

	page = Page
	page.body {
		templatePath = 'resource://My.Package/Private/Templates/PageTemplate.html'
		sectionName = 'body'
	}

This results in only the part inside the template's "body" section to be used for
rendering the body of the generated page.

To add actual content from Neos to the desired places in the markup, a special
ViewHelper to turn control back to TypoScript is used. This has been mentioned
in :ref:`page-rendering` already.

This template uses the ``render`` ViewHelper twice, once to render the
path `parts/menu` and once to render the path `content.main`::

	<f:section name="body">
		<ts:render path="parts.menu" />
		<h1>{title}</h1>
		<ts:render path="content.main" />
	</f:section>

Those paths are relative to the current path. Since that part of the template is
rendered by the TypoScript object at `page.body`, this is the starting point
for the relative paths. This means the ``Menu`` and the ``ContentCollection`` in this
TypoScript are used for rendering the output::

	page = Page
	page.body {
		templatePath = 'resource://My.Package/Private/Templates/PageTemplate.html'
		sectionName = 'body'
		parts.menu = Menu
		content.main = ContentCollection
		content.main.nodePath = 'main'
	}

The Head
--------

The ``head`` of a page generated by Neos contains only minimal content by default.
Apart from the meta tag declaring the character set it is empty::

	<head>
		<meta charset="UTF-8" />
	</head>

To fill this with life, it is recommended to add sections to the head of your HTML template that
group the needed parts. Additional TypoScript `Template` objects are then used to include them
into the generated page. Here is an example:

*Page/Default.html*

::

	<head>
		<f:section name="meta">
			<title>{title}</title>
		</f:section>

		<f:section name="stylesheets">
			<!-- put your stylesheet inclusions here, they will be included in your website by TypoScript -->
		</f:section>

		<f:section name="scripts">
			<!-- put your javascript inclusions here, they will be included in your website by TypoScript -->
		</f:section>
	</head>

*Library/Root.fusion*

::

	page.head {
		meta = TYPO3.TypoScript:Template {
			templatePath = 'resource://Acme.DemoCom/Private/Templates/Page/Default.html'
			sectionName = 'meta'

			title = ${q(node).property('title')}
		}
		stylesheets.site = TYPO3.TypoScript:Template {
			templatePath = 'resource://Acme.DemoCom/Private/Templates/Page/Default.html'
			sectionName = 'stylesheets'
		}
		javascripts.site = TYPO3.TypoScript:Template {
			templatePath = 'resource://Acme.DemoCom/Private/Templates/Page/Default.html'
			sectionName = 'scripts'
		}
	}

The TypoScript fills the `page.head` instance of ``TYPO3.TypoScript:Array`` with content. The predefined paths for
`page.head.stylesheets`, `page.head.javascripts` or `page.body.javascripts` should be used to add custom includes. They
are implemented by a TypoScript `Array` and allow arbitrary items to specify JavaScript or CSS includes without any
restriction on the content.

This will render some more head content::

		<head>
		…
		<title>Home</title>
		<!-- put your stylesheet inclusions here, they will be included in your website by TypoScript -->
		<!-- put your javascript inclusions here, they will be included in your website by TypoScript -->
		…
	</head>

This provides for flexibility and allows to control precisely what ends up in the generated
markup. Anything that is needed can be added freely, it just has to be in a section that is
included.

Menu Rendering
==============

Out of the box the `Menu` is rendered using a simple unsorted list::

	<ul class="nav">
		<li class="current">
			<a href="home.html">Home</a>
		</li>

		<li class="normal">
			<a href="blog.html">Blog</a>
		</li>
	</ul>

Wrapping this into some container (if needed) in a lot of cases provides for enough possibilities
to style the menu using CSS. In case it still is needed, it is possible to change the rendered markup
of `Menu` using TypoScript. `Menu` is defined inside the core of Neos together with Neos.Neos.NodeTypes:

*Neos.Neos/Resources/Private/DefaultTypoScript/ImplementationClasses.fusion*

::

	prototype(Neos.Neos:Menu).@class = 'Neos\\Neos\\Fusion\\MenuImplementation'

*Neos.Neos.NodeTypes/Resources/Private/Fusion/Root.fusion*

::

	prototype(Neos.Neos.NodeTypes:Menu) < prototype(Neos.Neos:Menu)
	prototype(Neos.Neos.NodeTypes:Menu) {
		templatePath = 'resource://Neos.Neos.NodeTypes/Private/Templates/FusionObjects/Menu.html'
		entryLevel = ${String.toInteger(q(node).property('startLevel'))}
		maximumLevels = ${String.toInteger(q(node).property('maximumLevels'))}
		node = ${node}
	}

The above code defines the *prototype* of `Menu` with the `prototype(Menu)` syntax.
This prototype is the "blueprint" of all `Menu` objects which are instantiated.
All properties which are defined on the prototype (such as `@class` or `templatePath`)
are automatically active on all `Menu` *instances*, if they are not explicitly overridden.

One way to adjust the menu rendering is to override the `templatePath` property, which
points to a Fluid template. To achieve that, we have two possibilities.

First, the `templatePath` for the menu at `page.body.parts.menu` can be set::

	page.body.parts.menu.templatePath = 'resource://My.Package/Private/Templates/MyMenuTemplate.html'

This overrides the `templatePath` which was defined in `prototype(Menu)` for
this single menu.

Second, the `templatePath` inside the `Menu` prototype itself can be changed::

	prototype(Menu).templatePath = 'resource://My.Package/Private/Templates/MyMenuTemplate.html'

In this case, the changed template path is used for *all menus* which do not override
the `templatePath` explicitly. Every time `prototype(...)` is used, this can be
understood as: "For *all* objects of type ..., define *something*"

After setting the path, changing the menu is simply a job of copying the default
`Menu` template into `MyMenuTemplate.html` and adjusting the markup as needed.

Menu states
-----------

The default `Menu` implementation assigns CSS classes to the `li` tags depending on
their state:

:current: A menu item pointing to the page that is currently shown
:active: Any menu item that is on the path to the `current` page
:normal: Any menu item that is neither `current` nor `active`

Content Element Rendering
=========================

The rendering of content elements follows the same principle as shown for the `Menu`.
The default TypoScript is defined in the Neos.NodeTypes package and the content elements
all have default Fluid templates.

Combined with the possibility to define custom templates per instance or on the prototype
level, this already provides a lot of flexibility. Another possibility is to inherit from
the existing TypoScript and adjust as needed using TypoScript.

The available properties and settings that the TypoScript objects in Neos provide are
described in :ref:`neos-typoscript-reference`.


Including CSS and JavaScript in a Neos Site
===========================================

Including CSS and JavaScript should happen through one of the predefined places of the `Page` object. Depending on
the desired position one of the `page.head.javascripts`, `page.head.stylesheets` or `page.body.javascripts` Arrays
should be extended with an item that renders script or stylesheet includes::

	page.head {

		stylesheets {
			bootstrap = '<link href="//netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css" rel="stylesheet">'
		}

		javascripts {
			jquery = '<script src="//code.jquery.com/jquery-1.10.1.min.js"></script>'
		}

	}

	page.body {

		javascripts {
			bootstrap = '<script src="//netdna.bootstrapcdn.com/bootstrap/3.0.3/js/bootstrap.min.js"></script>'
		}

	}

The `page.body.javascripts` content will be appended to the rendered page template so the included scripts should be
placed before the closing body tag. As always in TypoScript the elements can be a simple string value, a TypoScript
object like `Template` or an expression::

	page.head {
		# Add a simple value as an item to the javascripts Array
		javascripts.jquery = '<script src="//code.jquery.com/jquery-1.10.1.min.js"></script>'

		# Use an expression to render a CSS include (this is just an example, bootstrapVersion is not defined by Neos)
		stylesheets.bootstrap = ${'<link href="//netdna.bootstrapcdn.com/bootstrap/' + bootstrapVersion + '/css/bootstrap.min.css" rel="stylesheet">'}
	}

	page.body {
		# Use a Template object to access a special section of the site template
		javascripts.site = TYPO3.TypoScript:Template {
			templatePath = 'resource://Acme.DemoCom/Private/Templates/Page/Default.html'
			sectionName = 'bodyScripts'
		}
	}

The order of the includes can be specified with the `@position` property inside the `Array` object. This is especially
handy for including JavaScript libraries and plugins in the correct order::

	page.head {
		jquery = '<script src="//code.jquery.com/jquery-1.10.1.min.js"></script>'

		javascripts.jquery-ui = '<script src="path-to-jquery-ui"></script>'
		javascripts.jquery-ui.@position = 'after jquery'
	}

CSS and JavaScript restrictions in a Neos Site
==============================================

Very little constraints are imposed through Neos for including JavaScripts or stylesheets.
But since the Neos user interface itself is built with HTML, CSS and JavaScript itself, some caveats exist.

Since the generated markup contains no stylesheets by default and the generated JS is minimal,
those restrictions affect only the display of the page to the editor when logged in to the Neos
editing interface.

In this case, the Neos styles are included and a number of JavaScript libraries are loaded,
among them jQuery, Ember JS and VIE. The styles are all confined to a single root selector and
for JavaScript the impact is kept as low as possible through careful scoping.

CSS Requirements
----------------

* the `<body>` tag is not allowed to have a CSS style with `position:relative`,
  as this breaks the positions of modal dialogs we show at various places.
  *Zurb Foundation* is one well-known framework which sets this as default, so
  if you use it, then fix the error with `body { position: static }`.

TODO check if this is still true

JavaScript Requirements
-----------------------

TODO "what about the UI below a single DOM element idea"

Adjusting the HTTP response
===========================

It is possible to set HTTP headers and the status code of the response from TypoScript. See :ref:`TYPO3_TypoScript__Http_Message`
for an example.
