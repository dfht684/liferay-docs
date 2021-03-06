# Embedding Portlets in Themes and Layout Templates [](id=embedding-portlets-in-themes-and-layout-templates)

You may occasionally want to embed a portlet in a theme or layout template,
making the portlet visible on all pages where the theme/layout is used. Since
there are numerous drawbacks to hard-coding a specific portlet into place, the
*Portlet Providers* framework offers an alternative that displays the
appropriate portlet based on a given entity type and action.

In this tutorial, you'll learn how to declare an entity type and action in a
custom theme/layout, and you'll create a module that finds the correct portlet
to use based on those given parameters. You'll first learn how to embed portlets
into a theme.

## Adding a Portlet to a Custom Theme [](id=adding-a-portlet-to-a-custom-theme)

The first thing you should do is open the template file for which you want to
declare an embedded portlet. For example, the `portal_normal.ftl` template file
is a popular place to declare embedded portlets. To avoid problems, it's usually
best to embed portlets with an entity type and action, but you may encounter
circumstances where you'll want to hard code it by portlet name. Both methods
are covered here.

### Embedding a Portlet by Entity Type and Action[](id=embedding-a-portlet-by-class-name)

Start by inserting the following declaration where you want the portlet
embedded:

    <@liferay_portlet["runtime"]
        portletProviderAction=ACTION
        portletProviderClassName="CLASS_NAME"
    />

This declaration expects two parameters: the type of action and the class name
of the entity type the portlet should handle. Here's an example of an embedded
portlet declaration that uses the class name: 

    <@liferay_portlet["runtime"]
        portletProviderAction=portletProviderAction.VIEW
        portletProviderClassName="com.liferay.portal.kernel.servlet.taglib.ui.LanguageEntry"
    />

This declares that the theme is requesting to view language entries. @product@
determines which deployed portlet to use in this case by providing the portlet
with the highest service ranking.

There are five different kinds of actions supported by the Portlet Providers
framework: `ADD`, `BROWSE`, `EDIT`, `PREVIEW`, and `VIEW`. Specify the entity
type and action in your theme's runtime declaration.

Great! Your theme declaration is complete. However, the Portal is not yet
configured to handle this request. You must create a module that can
find the portlet that fits the theme's request.

1.  [Create an OSGi module](/develop/tutorials/-/knowledge_base/7-1/starting-module-development#creating-a-module). 

2.  Create a unique package name in the module's `src` directory, and create a
    new Java class in that package. To follow naming conventions, name the class
    based on the entity type and action type, followed by *PortletProvider*
    (e.g., `SiteNavigationLanguageEntryViewPortletProvider`). The class should
    extend the
    [`BasePortletProvider`](@platform-ref@/7.1/javadocs/portal-kernel/com/liferay/portal/kernel/portlet/BasePortletProvider.html)
    class and implement the appropriate portlet provider interface based on the
    action you chose in your theme (e.g.,
    [`ViewPortletProvider`](@platform-ref@/7.1/javadocs/portal-kernel/com/liferay/portal/kernel/portlet/ViewPortletProvider.html),
    [`BrowsePortletProvider`](@platform-ref@/7.1/javadocs/portal-kernel/com/liferay/portal/kernel/portlet/BrowsePortletProvider.html),
    etc.).

3.  Directly above the class's declaration, insert the following annotation:

        @Component(
            immediate = true,
            property = {"model.class.name=CLASS_NAME"},
            service = INTERFACE.class
        )

    The `property` element should match the entity type you specified in your
    theme declaration (e.g.,
    `com.liferay.portal.kernel.servlet.taglib.ui.LanguageEntry`). Also, your
    `service` element should match the interface you're implementing (e.g.,
    `ViewPortletProvider.class`). You can view an example of a similar
    `@Component` annotation in the
    [RolesSelectorEditPortletProvider](https://github.com/liferay/liferay-portal/blob/7.1.x/modules/apps/roles/roles-selector-web/src/main/java/com/liferay/roles/selector/web/internal/portlet/RolesSelectorEditPortletProvider.java)
    class.

4.  Specify the methods you want to implement. Make sure to retrieve the portlet 
    ID and page ID that should be provided when this service is called by your 
    theme.

    A common use case is to implement the `getPortletId()` and
    `getPlid(ThemeDisplay)` methods. You can view the
    [SiteNavigationLanguageViewPortletProvider](https://github.com/liferay/liferay-portal/blob/7.1.x/modules/apps/site-navigation/site-navigation-language-web/src/main/java/com/liferay/site/navigation/language/web/internal/portlet/SiteNavigationLanguageViewPortletProvider.java)
    for an example of how these methods can be implemented to provide a portlet
    for embedding in a theme. This example module returns the portlet ID of the
    Language portlet specified in
    [SiteNavigationLanguagePortletKeys](https://github.com/liferay/liferay-portal/blob/7.1.x/modules/apps/site-navigation/site-navigation-language-api/src/main/java/com/liferay/site/navigation/language/constants/SiteNavigationLanguagePortletKeys.java).
    It also returns the PLID, which is the ID that uniquely
    identifies a page used by your theme. By retrieving these, your theme will
    know which portlet to use, and which page to use it on.

The only thing left to do is generate the module's JAR file and copy it to your
Portal's `osgi/modules` directory. Once the module is installed and activated in
your Portal's service registry, your embedded portlet is available for
use wherever your theme is used.

You successfully requested a portlet based on the entity and action types
required, and created and deployed a module that retrieves the portlet and
embeds it in your theme. 

### Embedding a Portlet by Portlet Name [](id=embedding-a-portlet-by-portlet-name)

If you'd like to embed a specific portlet in the theme, you can hard code it by
providing its instance ID and name:

    <@liferay_portlet["runtime"]
        instanceId="INSTANCE_ID"
        portletName="PORTLET_NAME"
    />

+$$$

**Note:** If your portlet is instanceable, an instance ID must be provided; 
otherwise, you can remove this line. To set your portlet to non-instanceable, 
set the property `com.liferay.portlet.instanceable` in the component annotation 
of your portlet to `false`.

$$$

The portlet name must be the same as `javax.portlet.name`'s value.
 
Here's an example of an embedded portlet declaration that uses the portlet name 
to embed a web content portlet:

    <@liferay_portlet["runtime"]
        portletName="com_liferay_journal_content_web_portlet_JournalContentPortlet"
    />

You can also set default preferences for an application. This process is covered 
next.

### Setting Default Preferences for an Embedded Portlet [](id=setting-default-preferences-for-an-embedded-portlet)

Follow these steps to set default portlet preferences for an embedded portlet:

1.  Retrieve portlet preferences using the `freeMarkerPortletPreferences` 
    object. The example below retrieves the `barebone` 
    [portlet decorator](/develop/tutorials/-/knowledge_base/7-1/creating-configurable-styles-for-portlet-wrappers):

        <#assign preferences = freeMarkerPortletPreferences.getPreferences(
          "portletSetupPortletDecoratorId", "barebone"
        ) />
 
2.  Set the `defaultPreferences` attribute of the embedded portlet to the 
    `freeMarkerPortletPreferences` object you just configured:

        <@liferay_portlet["runtime"]
            defaultPreferences="${preferences}"
            portletName="com_liferay_login_web_portlet_LoginPortlet"
        />

Now you know how to set default preferences for embedded portlets! Next you can 
see the additional attributes you can use for your embedded portlets.
 
### Additional Attributes for Portlets [](id=additional-attributes-for-portlets)

Below are some additional attributes you can define for embedded portlets:

**defaultPreferences**: A string of Portlet Preferences for the application. 
This includes look and feel configurations.

**instanceId**: The instance ID for the app, if the application is instanceable.

**persistSettings**: Whether to have an application use its default settings, 
which will persist across layouts. The default value is *true*.

**settingsScope**: Specifies which settings to use for the application. The 
default value is `portletInstance`, but it can be set to `group` or `company`.

Now you know how to embed a portlet in your theme by class name and by portlet 
name and how to configure your embedded portlet! Next, you'll learn a similar 
process to embed a portlet in your layout template.

## Adding a Portlet to a Custom Layout Template [](id=adding-a-portlet-to-a-custom-layout-template)

The process for embedding portlets in layout templates is similar to that for 
themes. The only change is the declaration you insert in your template's FTL
file.

+$$$

**Note:** Velocity layout templates are supported, but deprecated as of 
@product-ver@. We recommend that you convert your Velocity layout templates to 
FreeMarker.

$$$

1.  Open your layout template's FTL file (e.g., `docroot/1_2_1-columns.ftl`).

2.  Add a `${processor.processPortlet("CLASS_NAME", ACTION)}` directive within 
    the column in which to embed the portlet.

    This declaration, just as the theme declaration, expects two parameters: the
    class name of the entity type you want the portlet to handle and the type
    of action. An example of an embedded portlet declaration can be viewed
    below:

        ${processor.processPortlet(
          "com.liferay.portal.kernel.servlet.taglib.ui.BreadcrumbEntry", 
          $portletProviderAction.VIEW
        )}

    This declares that the layout is requesting to view breadcrumb entries. The
    Portlet Providers framework offers four different actions for layout
    templates: `ADD`, `BROWSE`, `EDIT`, and `VIEW`. Specify the entity type and
    action in your layout's declaration.

Now that your layout declaration is complete, you'll need to create a module 
that can handle this request. This process is exactly the same for themes and
layout templates. Follow steps 1-4 in the 
[Embedding Portlets by Class Name](#embedding-a-portlet-by-class-name) 
section to set up your module.

+$$$

**Note:** In some cases, a default portlet is already provided to fulfill
certain requests. You can override the default portlet with your custom
portlet by specifying a higher service rank. To do this, set the following
property in your class' `@Component` declaration:

    property= {"service.ranking:Integer=20"}

Make sure you set the service ranking higher than the default portlet being used.

$$$

Lastly, generate the module's JAR file and deploy it to @product@. Now when
you assign your layout to a page, the portlet that fulfills the layout's request
is embedded on the page.

## Related Topics [](id=related-topics)

[Embedding Portlets in Themes and Layout Templates](/develop/tutorials/-/knowledge_base/7-1/embedding-portlets-in-themes-and-layout-templates)

[Portlets](/develop/tutorials/-/knowledge_base/7-1/portlets)

[Service Builder](/develop/tutorials/-/knowledge_base/7-1/service-builder)
