=============================
 Apache
=============================

.. contents :: :local:

Introduction
=============

Apache web server is the old and faitful workhorse of the internets.
Even though Apache itself is not producing any HTML, you might
still need to consider :doc:`mobilize-mobile cookie </serverside>`.

Integration
=============

Below is a snippet example how to use mobilize-mobile cookie in
Apache configuration file::

    # Get mobilize-mobile cookie value to environment
    # so that we can use env variable in Apache control flow
    # http://stackoverflow.com/questions/3876477/how-to-append-cookie-value-to-end-of-response-location-header-with-apache
    # Note that we need special handling of hyphen which cannot be escaped in regex
    SetEnvIf Cookie "mobilize(-)mobile=([^;]+)" mobilize=$2
    
    # By default, enable XSLT based theming on the site
    # The following condition is always true
    SetEnvIfNoCase SERVER_NAME "(.*)" xslt-theming=1

    # Disable XSLT web theming if we are rendering with mobilize.js
    SetEnvIf mobilize "1" xslt-theming=0

    # http://httpd.apache.org/docs/2.0/mod/mod_headers.html
    # Output debug headers, so you can see how Apache has seen the
    # cookies and the situation when the response was served
    Header set X-XSLT-theming %{xslt-theming}e
    Header set X-Mobilize %{mobilize}e

Then you can use these variables, for example, in
choosing if the transform filter should be applied::

    # This chain is used for public web pages
    FilterDeclare THEME
    FilterProvider THEME XSLT env=xslt-theming =1
    
    TransformOptions +ApacheFS +HTML +HideParseErrors
    # This is the location of compiled XSL theme transform
    TransformSet /theme.xsl
    
    # This will make Apache not to reload transformation every time
    # it is performed. Instead, a compiled version is hold in the
    # virtual URL declared above.
    TransformCache /theme.xsl /srv/plone/cows-rock/theme.xsl
    
    # We want to apply theme only for
    # 1. public pages (otherwise Wordpress administrative interface stops working)
    <Location "/">
        FilterChain THEME
    </Location>

    # 2. Admin interface and feeds should not receive any kind of theming
    <LocationMatch "(wp-login|wp-admin|wp-includes|xmlrpc|info)">
        # The following resets the filter chain
        # http://httpd.apache.org/docs/2.2/mod/mod_filter.html#filterchain
        FilterChain !
    </LocationMatch>

Testing
--------

Use wget to test headers and content served by Apache::

    wget -S http://blog.mfabrik.com
    
    wget -S --header "Cookie: mobilize-mobile=1" http://blog.mfabrik.com

Caching
=============

If you are using server-side optimizations in HTML, make sure that HTML pages
are not cached::

        ExpiresActive On
        ExpiresByType text/html A0
    