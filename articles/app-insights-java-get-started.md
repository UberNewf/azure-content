<properties 
	pageTitle="Get started with Application Insights in a Java web project" 
	description="Monitor performance and usage of your Java website with Application Insights" 
	services="application-insights" 
    documentationCenter=""
	authors="alancameronwills" 
	manager="keboyd"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/11/2015" 
	ms.author="awills"/>
 
# Get started with Application Insights in a Java web project

By adding Visual Studio Application Insights to your project, you can detect and diagnose performance issues and exceptions.

In addition, you can set up web tests to monitor your application's availability, and insert code into your web pages to understand usage patterns.

You'll need:

* Oracle JRE 1.6 or later, or Zulu JRE 1.6 or later
* A subscription to [Microsoft Azure](http://azure.microsoft.com/). (You could start with the [free trial](http://azure.microsoft.com/pricing/free-trial/).)


## 1. Get an Application Insights instrumentation key

1. Log into the [Microsoft Azure Portal](https://portal.azure.com)
2. Create a new Application Insights resource

    ![Click + and choose Application Insights](./media/app-insights-java-get-started/01-create.png)
3. Set the application type to Java web application.

    ![Fill a name, choose Java web app, and click Create](./media/app-insights-java-get-started/02-create.png)
4. Find the instrumentation key of the new resource. You'll need to paste this into your code project shortly.

    ![In the new resource overview, click Properties and copy the Instrumentation Key](./media/app-insights-java-get-started/03-key.png)

## 2. Add the Application Insights SDK for Java to your project

*Choose the appropriate way for your project.*

#### If you're creating a Dynamic Web project in Eclipse...

Use the [Application Insights SDK for Java plug-in][eclipse].

#### If you're using Maven...

If your project is already set up to use Maven for build, merge the following snippet of code to your pom.xml file.

Then refresh the project dependencies, to get the binaries downloaded.

    <repositories>
       <repository>
          <id>central</id>
          <name>Central</name>
          <url>http://repo1.maven.org/maven2</url>
       </repository>
    </repositories>

    <dependencies>
      <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>applicationinsights-web</artifactId>
        <!-- or applicationinsights-core for bare API -->
        <version>[0.9,)</version>
      </dependency>
    </dependencies>


* *Build or checksum validation errors? Try using a specific version:* `<version>0.9.2<\version>`

#### If you're using Gradle...

If your project is already set up to use Gradle for build, merge the following snippet of code to your build.gradle file.

Then refresh the project dependencies, to get the binaries downloaded.

    repositories {
      mavenCentral()
    }

    dependencies {
      compile group: 'com.microsoft.azure', name: 'applicationinsights-web', version: '0.9.+'
      // or applicationinsights-core for bare API
    }

* *Build or checksum validation errors? Try using a specific version:* `version:'0.9.2'`

#### Otherwise ...

Manually add the SDK:

1. Download the [Azure Libraries for Java](http://dl.msopentech.com/lib/PackageForWindowsAzureLibrariesForJava.html)
2. Extract the following binaries from the zip file, and add them to your project:
 * applicationinsights-core
 * applicationinsights-web
 * commons-codec
 * commons-io
 * commons-lang
 * commons-logging
 * guava
 * httpclient
 * httpcore
 * jsr305


*What's the relationship between the `-core` and `-web` components?*

`applicationinsights-core` gives you the bare API with no automatic telemetry. 
`applicationinsights-web` gives you metrics tracking HTTP request counts and response times. 


## 3. Add an Application Insights config file

Add ApplicationInsights.xml to the resources folder in your project. Copy into it the following XML.

Substitute the instrumentation key that you got from the Azure portal.

    <?xml version="1.0" encoding="utf-8"?>
    <ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">


      <!-- The key from the portal: -->

      <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>


      <!-- HTTP request component (not required for bare API) -->

      <TelemetryModules>
        <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule"/>
        <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebSessionTrackingTelemetryModule"/>
        <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebUserTrackingTelemetryModule"/>
      </TelemetryModules>

      <!-- Events correlation (not required for bare API) -->
      <!-- These initializers add context data to each event -->

      <TelemetryInitializers>
        <Add   type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationIdTelemetryInitializer"/>
        <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationNameTelemetryInitializer"/>
        <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebSessionTelemetryInitializer"/>
        <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserTelemetryInitializer"/>
        <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserAgentTelemetryInitializer"/>

      </TelemetryInitializers>
    </ApplicationInsights>


* The instrumentation key is sent along with every item of telemetry and tells Application Insights to display it in your resource.
* The HTTP Request component is optional. It automatically sends telemetry about requests and response times to the portal.
* Events correlation is an addition to the HTTP request component. It assigns an identifier to each request received by the server, and adds this as a property to every item of telemetry as the property 'Operation.Id'. It allows you to correlate the telemetry associated with each request by setting a filter in [diagnostic search][diagnostic].

## 4. Add an HTTP filter

The last configuration step allows the HTTP request component to log each web request. (Not required if you just want the bare API.)

Locate and open the web.xml file in your project, and merge the following snippet of code under the web-app node, where your application filters are configured.

To get the most accurate results, the filter should be mapped before all other filters.

    <filter>
      <filter-name>ApplicationInsightsWebFilter</filter-name>
      <filter-class>
        com.microsoft.applicationinsights.web.internal.WebRequestTrackingFilter
      </filter-class>
    </filter>
    <filter-mapping>
       <filter-name>ApplicationInsightsWebFilter</filter-name>
       <url-pattern>/*</url-pattern>
    </filter-mapping>

#### If you're using MVC 3.1 or later

Edit these elements to include the Application Insights package:

    <context:component-scan base-package=" com.springapp.mvc, com.microsoft.applicationinsights.web.spring"/>

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <bean class="com.microsoft.applicationinsights.web.spring.RequestNameHandlerInterceptorAdapter" />
        </mvc:interceptor>
    </mvc:interceptors>

#### If you're using Struts 2

Add this item to the Struts configuration file (usually named struts.xml or struts-default.xml):

     <interceptors>
       <interceptor name="ApplicationInsightsRequestNameInterceptor" class="com.microsoft.applicationinsights.web.struts.RequestNameInterceptor" />
     </interceptors>
     <default-interceptor-ref name="ApplicationInsightsRequestNameInterceptor" />

(If you have interceptors defined in a default stack, the interceptor can simply be added to that stack.)

## 5. View your telemetry in Application Insights

Run your application.

Return to your Application Insights resource in Microsoft Azure.

HTTP requests data will appear on the overview blade. (If it isn't there, wait a few seconds and then click Refresh.)

![](./media/app-insights-java-track-http-requests/5-results.png)
 

Click through any chart to see more detailed metrics. 

![](./media/app-insights-java-track-http-requests/6-barchart.png)

 

And when viewing the properties of a request, you can see the telemetry events associated with it such as requests and exceptions.
 
![](./media/app-insights-java-track-http-requests/7-instance.png)


[Learn more about metrics.][metrics]

## 5. Capture log traces

You can use Application Insights to slice and dice logs from Log4J, Logback or other logging frameworks. You can correlate the logs with HTTP requests and other telemetry. [Learn how][javalogs].

## 6. Send your own telemetry

Now that you've installed the SDK, you can use the API to send your own telemetry. 

* [Track custom events and metrics][track] to learn what users are doing with your application.
* [Search events and logs][diagnostic] to help diagnose problems.


In addition, you can bring more features of Application Insights to bear on your application:

* [Add web client telemetry][usage] to monitor page views and basic user metrics.
* [Set up web tests][availability] to make sure your application stays live and responsive.


## Questions? Problems?

[Troubleshooting Java](app-insights-java-troubleshoot.md)


[AZURE.INCLUDE [app-insights-learn-more](../includes/app-insights-learn-more.md)]




