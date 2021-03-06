# Customizing the Control Menu [](id=customizing-the-control-menu)

The Control Menu is the most visible and accessible menu in Liferay. It is
visible to the user in most places, always displaying helpful text or options
at the top of the page. For example, on your home page, the Control Menu offers
default options for accessing the Product Menu, Simulation Menu, and Add Menu.
You can think of this menu as the gateway to configuring options in Liferay.

![Figure 1: The Control Menu has three configurable areas: left, right, and middle.](../../images/control-menu-home.png)

If you navigate away from the home page, the Control Menu adapts and provides
helpful functionality for what ever option you're using. For example, if you
navigated to Site Administration &rarr; *Content* &rarr; *Web Content*, you'd be
displayed a Control Menu with different functionality tailored for that option.

![Figure 2: When switching your context to web content, the Control Menu adapts to provide helpful options for that area.](../../images/control-menu-web-content.png)

The default Control Menu is made up of three categories that represent the left,
middle, and right portions of the menu. You can create navigation entries for
each category, which can provide options or further navigation for the
particular screen you're on.

+$$$

**Note:** You can add the Control Menu to a custom theme by adding the following
snippet into your `portal_normal.ftl`:

    <@liferay.control_menu />

The other product navigation menus (e.g., Product Menu, Simulation Menu) are
included in this tag, so specifying the above snippet will embed all three menus
into your theme. Embedding the User Personal Bar is slightly different. Visit
the [Providing the User Peronsal Bar](/develop/tutorials/-/knowledge_base/7-0/providing-the-user-personal-bar)
tutorial for more information.

$$$

In this tutorial, you'll learn how to create your own entries to customize the
Control Menu. Make sure to read the
[Adding Custom Panel Categories](/develop/tutorials/-/knowledge_base/7-0/customizing-the-product-menu#adding-custom-panel-categories
before beginning this tutorial. This tutorial assumes you have knowledge on
creating a panel category. You'll begin by creating an entry for the Control
Menu.

1. Create a generic OSGi module using your favorite third party tool, or use
   [Blade CLI](/develop/tutorials/-/knowledge_base/7-0/blade-cli). 

2. Create a unique package name in the module's `src` directory and create a
   new Java class in that package. To follow naming conventions, give your
   class a unique name followed by *ProductNavigationControlMenuEntry* (e.g.,
   `StagingProductNavigationControlMenuEntry`).

3. Directly above the class's declaration, insert the following annotation:

        @Component(
            immediate = true,
            property = {
                "product.navigation.control.menu.category.key=" + CONTROL_MENU_CATEGORY,
                "product.navigation.control.menu.category.order:Integer=INTEGER"
            },
            service = ProductNavigationControlMenuEntry.class
        )

    The `product.navigation.control.menu.category.key` property specifies the
    category your entry should reside in. As mentioned previously, the default
    Control Menu provides three categories: Sites (left portion), Tools (middle
    portion), and User (right portion).
    
    ![Figure 3: This image shows where your entry will reside depending on the category you select.](../../images/control-menu-areas.png)
    
    To specify the category, reference the appropriate key in the
    [ProductNavigationControlMenuCategoryKeys](https://github.com/liferay/liferay-portal/blob/master/modules/apps/web-experience/product-navigation/product-navigation-control-menu-api/src/main/java/com/liferay/product/navigation/control/menu/constants/ProductNavigationControlMenuCategoryKeys.java)
    class. For example, the following property would place your entry in the
    middle portion of the Control Menu:

        "product.navigation.control.menu.category.key=" + ProductNavigationControlMenuCategoryKeys.TOOLS

    Similar to panel categories, you'll also need to specify an integer for the
    order in which your entry will be displayed in the category.

    Lastly, your `service` element should specify the
    `ProductNavigationControlMenuEntry.class` service. You can view an example
    of a similar `@Component` annotation in the
    [IndexingProductNavigationControlMenuEntry](https://github.com/liferay/liferay-portal/blob/master/modules/apps/foundation/portal-search/portal-search-web/src/main/java/com/liferay/portal/search/web/product/navigation/control/menu/IndexingProductNavigationControlMenuEntry.java)
    class.

4. Implement the
   [ProductNavigationControlMenuEntry](https://github.com/liferay/liferay-portal/blob/master/modules/apps/web-experience/product-navigation/product-navigation-control-menu-api/src/main/java/com/liferay/product/navigation/control/menu/ProductNavigationControlMenuCategory.java)
   interface. You can also extend the
   [BaseProductNavigationControlMenuEntry](https://github.com/liferay/liferay-portal/blob/master/modules/apps/web-experience/product-navigation/product-navigation-control-menu-api/src/main/java/com/liferay/product/navigation/control/menu/BaseProductNavigationControlMenuEntry.java)
   or
   [BaseJSPProductNavigationControlMenuEntry](https://github.com/liferay/liferay-portal/blob/master/modules/apps/web-experience/product-navigation/product-navigation-control-menu-api/src/main/java/com/liferay/product/navigation/control/menu/BaseJSPProductNavigationControlMenuEntry.java)
   abstract classes. Typically, the `BaseProductNavigationControlMenuEntry` is
   extended for basic entries (e.g.,
   [IndexingProductNavigationControlMenuEntry](https://github.com/liferay/liferay-portal/blob/master/modules/apps/foundation/portal-search/portal-search-web/src/main/java/com/liferay/portal/search/web/product/navigation/control/menu/IndexingProductNavigationControlMenuEntry.java))
   that only display a link with text or a simple icon. If you'd like to provide
   a more complex UI, like buttons or a sub-menu, you can do so by overriding
   the `include()` and `includeBody()` methods. If you are going to use JSPs for
   generating the UI, you can extend `BaseJSPProductNavigationControlMenuEntry`
   to save time. This will be elaborated on more extensively in the next step.

5. Define your Control Menu entry. You'll explore two examples to discover some
   options you have available for defining your entry. First, let's take a look
   at a simple example for providing text and an icon. The
   [IndexingProductNavigationControlMenuEntry](https://github.com/liferay/liferay-portal/blob/master/modules/apps/foundation/portal-search/portal-search-web/src/main/java/com/liferay/portal/search/web/product/navigation/control/menu/IndexingProductNavigationControlMenuEntry.java)
   extends the `BaseProductNavigationControlMenuEntry` class and is used when
   Liferay is indexing. For this process, the indexing entry is displayed in the
   *Tools* (middle) area of the Control menu with a *Refresh* icon and text
   stating *The Portal is currently indexing*. This is done by calling the
   following methods:

        @Override
        public String getIconCssClass(HttpServletRequest request) {
            return "icon-refresh";
        }

    More icons can be found in the
    [FontAwesome 4.6.1 docs](https://fortawesome.github.io/Font-Awesome/icons/).

        @Override
        public String getLabel(Locale locale) {
            ResourceBundle resourceBundle = ResourceBundleUtil.getBundle(
                "content.Language", locale, getClass());

            return LanguageUtil.get(
                resourceBundle, "the-portal-is-currently-reindexing");
        }

    For a more advanced example, you can inspect the
    [ProductMenuProductNavigationControlMenuEntry](https://github.com/liferay/liferay-portal/blob/master/modules/apps/web-experience/product-navigation/product-navigation-product-menu-web/src/main/java/com/liferay/product/navigation/product/menu/web/product/navigation/control/menu/ProductMenuProductNavigationControlMenuEntry.java).
    This entry displays in the *Sites* (left) area of the Control Menu, but
    unlike the previous example, the `BaseJSPProductNavigationControlMenuEntry`
    class is extended. This provides several more methods that allows you to use
    JSPs to define your entry's UI. There are two methods to pay special
    attention to:
 
        @Override
        public String getBodyJspPath() {
            return "/portlet/control_menu/product_menu_control_menu_entry_body.jsp";
        }

        @Override
        public String getIconJspPath() {
            return "/portlet/control_menu/product_menu_control_menu_entry_icon.jsp";
        }

    The `getIconJspPath()` method provides the Product Menu icon
    (![Menu Closed](../../images/icon-menu.png) &rarr; ![Menu Open](../../images/icon-menu-open.png))
    and the `getBodyJspPath()` method adds the UI body for the entry outside of
    the Control Menu. The latter method must be used when providing a UI outside
    the Control Menu. You can easily test this when you open and close the
    Product Menu on the home page. For best practices on how to construct a body
    and icon JSP for your entry, visit the
    [product_menu_control_menu_entry_body.jsp](https://github.com/liferay/liferay-portal/blob/master/modules/apps/web-experience/product-navigation/product-navigation-product-menu-web/src/main/resources/META-INF/resources/portlet/control_menu/product_menu_control_menu_entry_body.jsp)
    and
    [product_menu_control_menu_entry_icon.jsp](https://github.com/liferay/liferay-portal/blob/master/modules/apps/web-experience/product-navigation/product-navigation-product-menu-web/src/main/resources/META-INF/resources/portlet/control_menu/product_menu_control_menu_entry_icon.jsp)
    files, respectively.

    Lastly, if you're planning on providing functionality that will stay
    exclusively inside the Control Menu, you can inspect how the
    [Staging](https://github.com/liferay/liferay-portal/blob/master/modules/apps/web-experience/staging/staging-bar-web/src/main/java/com/liferay/staging/bar/web/product/navigation/control/menu/StagingProductNavigationControlMenuEntry.java)
    class calls its JSP:

        @Override
        public String getIconJspPath() {
            return "/control_menu/entry.jsp";
        }

    In particular, the
    [entry.jsp](https://github.com/liferay/liferay-portal/blob/master/modules/apps/web-experience/staging/staging-bar-web/src/main/resources/META-INF/resources/control_menu/entry.jsp)
    is returned, which embeds the Staging Bar portlet into the Control Menu.

    You will also need to specify the servlet context from where you are loading
    the JSP files. If this is inside an OSGi module, make sure your `bnd.bnd`
    file has defined a web context path:

        Bundle-SymbolicName: com.sample.my.module.web
        Web-ContextPath: /my-module-web

    And then reference the Servlet context using the symbolic name of your
    module like this:

        @Override
        @Reference(
            target = "(osgi.web.symbolicname=com.sample.my.module.web)",
            unbind = "-"
        )
        public void setServletContext(ServletContext servletContext) {
            super.setServletContext(servletContext);
        }

6. Define when to display your new entry in the Control Menu. As you've learned
   already, the Control Panel displays different entries depending on the page
   you've navigated to. You can specify when your entry should display using the
   `isShow(HttpServletRequest)` method.

    For example, the `IndexingProductNavigationControlMenuEntry` class queries
    the number of indexing jobs when calling `isShow`. If the query count is
    *0*, then the indexing entry is not displayed in the Control Menu:

        @Override
        public boolean isShow(HttpServletRequest request) throws PortalException {
            int count = _indexWriterHelper.getReindexTaskCount(
                CompanyConstants.SYSTEM, false);

            if (count == 0) {
                return false;
            }

            return super.isShow(request);
        }

    The `StagingProductNavigationControlMenuEntry` class is selective over which
    pages to display for. The staging entry is configured to never display if
    the page is an administration page (e.g., *Site Administration*, *My
    Account*, etc.):

        @Override
        public boolean isShow(HttpServletRequest request) throws PortalException {
            ThemeDisplay themeDisplay = (ThemeDisplay)request.getAttribute(
                WebKeys.THEME_DISPLAY);

            Layout layout = themeDisplay.getLayout();

            // This controls if the page is an Administration Page

            if (layout.isTypeControlPanel()) {
                return false;
            }

            // This controls if Staging is enabled

            if (!themeDisplay.isShowStagingIcon()) {
                return false;
            }

            return true;
        }

Excellent! You've created your entry in one of the three default panel
categories in the Control Menu. You learned a basic way and an advanced way of
providing that entry, and learned how to apply both.
