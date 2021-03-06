# Resource Bundle Override [](id=resource-bundle-override)

This example overrides the default `add-blog-entry` language key (English and
Spanish) for @product@'s default Blogs Admin portlet. After deploying this
sample to @product@, the Blogs Admin portlet's *Add Blog Entry* button is
modified to display *Overridden Add Blog Entry*. If you change @product@'s
default language to Spanish, the modified language key is translated to display
in that language. For example, the text changes to *Añadir entrada
sobreescrita*.

![Figure 1: The customized Blogs Admin portlet displays the new `add-blog-entry` language key in English.](../../../images/hook-resourcebundle.png)

For reference, the Blogs portlet's language keys are stored in the
[liferay-portal](https://github.com/liferay/liferay-portal) Github repo's
`modules/apps/blogs/blogs-web/src/main/resources/content` folder.

## What API(s) and/or code components does this sample highlight? [](id=what-apis-and-or-code-components-does-this-sample-highlight)

This sample leverages the
[ResourceBundleLoader](@platform-ref@/7.1-latest/javadocs/portal-kernel/com/liferay/portal/kernel/util/ResourceBundleLoader.html)
API.

## How does this sample leverage the API(s) and/or code component? [](id=how-does-this-sample-leverage-the-apis-and-or-code-component)

This sample conveys the recommended approach to override a portlet's
language keys file for any module that is deployed to @product@'s OSGi runtime
(not applicable to @product@'s core language keys).

The steps to override a portlet's language keys are

- Implement a resource bundle loader.
- Register the service.
- Provide the new language keys that will override the original ones.

The resource bundle loader is a class that should implement the interface
`com.liferay.portal.kernel.util.ResourceBundleLoader`. Specifically, you must
implement the `loadResourceBundle` method, which returns the loaded resource
bundle:

    @Override
    public ResourceBundle loadResourceBundle(String languageId) {
        return _resourceBundleLoader.loadResourceBundle(languageId);
    }

Then you must set the resource bundle loader to load the resource bundles as an
`AggregateResourceBundleLoader`.

    @Reference(target = "(&(bundle.symbolic.name=com.liferay.blogs.web)(!(component.name=com.liferay.blade.samples.hook.resourcebundle.ResourceBundleLoaderComponent)))")
    public void setResourceBundleLoader(
        ResourceBundleLoader resourceBundleLoader) {

        _resourceBundleLoader = new AggregateResourceBundleLoader(
            new CacheResourceBundleLoader(
                new ClassResourceBundleLoader(
                    "content.Language",
                    ResourceBundleLoaderComponent.class.getClassLoader())),
        resourceBundleLoader);
    }

The `@Reference` annotation targets the original Blogs module by specifying
its symbolic name `com.liferay.blogs.web`. This sample's own component name
(i.e., `com.liferay.blade.samples.hook.resourcebundle.ResourceBundleLoaderComponent`)
is not targeted to use this resource bundle.

Also note the required parameters to set the resource bundle loader:

- The base language file name (e.g., `content.Language`).
- The classloader for your resource bundle loader.
- The resource bundle loader from the method's parameter.

The class should also register the resource bundle loader in the OSGi runtime.
This is done by setting the following three properties:

- `bundle.symbolic.name`: The symbolic name of the target module (i.e., the
	module's keys you're overriding).
- `resource.bundle.base.name`: The resource bundle base name that points to
  your language files.
- `servlet.context.name`: The servlet context name of the target module.

These properties are set in your class's `@Component` annotation like this:

    @Component(
        immediate = true,
        property = {
            "bundle.symbolic.name=com.liferay.blogs.web",
            "resource.bundle.base.name=content.Language",
            "servlet.context.name=blogs-web"
        }
    )

Lastly, the new `language.properties` files should be added to the folder
`src/content` for each locale's keys you want to override. Since this
example's goal is to only override the English and Spanish keys, the
`Language_en.properties` and `Language_es.properties` are added.

This approach can be used to override any portlet's language keys (i.e.,
`language.properties` files that are inside a module deployed to @product@'s
OSGi runtime). If you need to override @product@'s core language keys, see the
[Overriding Global Language Keys](/develop/tutorials/-/knowledge_base/7-1/overriding-global-language-keys)
tutorial.

For more information on using a resource bundle to override a module's
language keys, see the
[Overriding a Module's Language Keys](/develop/tutorials/-/knowledge_base/7-1/overriding-a-modules-language-keys)
tutorial.

## Where Is This Sample? [](id=where-is-this-sample)

There are three different versions of this sample, each built with a different
build tool:

- [Gradle](https://github.com/liferay/liferay-blade-samples/tree/7.1/gradle/overrides/resource-bundle-override)
- [Liferay Workspace](https://github.com/liferay/liferay-blade-samples/tree/7.1/liferay-workspace/overrides/resource-bundle-override)
- [Maven](https://github.com/liferay/liferay-blade-samples/tree/7.1/maven/overrides/resource-bundle-override)
