.. _Apache Tomcat: https://tomcat.apache.org/
.. _Jetty: https://www.eclipse.org/jetty/

.. _server-servlet:

Embedding a servlet container
=============================
You can make Armeria serve your JEE web application on the same JVM and TCP/IP port by embedding
`Apache Tomcat`_ or Jetty_. Neither Tomcat nor Jetty will open a server socket or accept an incoming
connection. All HTTP requests and responses go through Armeria. As a result, you get the following bonuses:

- Your webapp gets HTTP/2 support for free even if your servlet container does not support it.
- You can run your RPC services on the same JVM and port as your webapp with no performance loss.

Embedding Apache Tomcat
-----------------------

Add a :api:`TomcatService` to a :api:`ServerBuilder`:

.. code-block:: java

    import com.linecorp.armeria.server.ServerBuilder;
    import com.linecorp.armeria.server.tomcat.TomcatService;

    ServerBuilder sb = Server.builder();

    sb.serviceUnder("/tomcat/api/rest/v2/",
                    TomcatService.forCurrentClassPath("/webapp"));

    sb.serviceUnder("/tomcat/api/rest/v1/",
                    TomcatService.forFileSystem("/var/lib/webapps/old_api.war"));

For more information, please refer to the API documentation of :api:`TomcatService` and
:api:`TomcatServiceBuilder`.

Embedding Jetty
---------------

Unlike Apache Tomcat, you need more dependencies and bootstrap code due to its modular design:

- org.eclipse.jetty:jetty-webapp
- org.eclipse.jetty:jetty-annotations
- org.eclipse.jetty:apache-jsp
- org.eclipse.jetty:apache-jstl

.. code-block:: java

    import com.linecorp.armeria.server.ServerBuilder;
    import com.linecorp.armeria.server.jetty.JettyService;

    import org.eclipse.jetty.annotations.ServletContainerInitializersStarter;
    import org.eclipse.jetty.apache.jsp.JettyJasperInitializer
    import org.eclipse.jetty.plus.annotation.ContainerInitializer
    import org.eclipse.jetty.util.resource.Resource;
    import org.eclipse.jetty.webapp.WebAppContext;

    ServerBuilder sb = Server.builder();

    sb.serviceUnder("/jetty/api/rest/v2/",
                    JettyService.builder()
                                .handler(newWebAppContext("/webapp"))
                                .build());

    static WebAppContext newWebAppContext(String resourcePath) throws MalformedURLException {
        final WebAppContext handler = new WebAppContext();
        handler.setContextPath("/");
        handler.setBaseResource(Resource.newClassPathResource(resourcePath));
        handler.setClassLoader(/* Specify your class loader here. */);
        handler.addBean(new ServletContainerInitializersStarter(handler), true);
        handler.setAttribute(
                "org.eclipse.jetty.containerInitializers",
                Collections.singletonList(
                        new ContainerInitializer(new JettyJasperInitializer(), null)));
        return handler;
    }

For more information, please refer to the API documentation of :api:`JettyService` and
:api:`JettyServiceBuilder`.
