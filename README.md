GWT Gradle Plugin
===================

[![Build Status](https://www.travis-ci.org/esoco/gwt-gradle-plugin.svg?branch=master)](https://www.travis-ci.org/esoco/gwt-gradle-plugin)&nbsp;&nbsp;&nbsp;
[ ![Download](https://api.bintray.com/packages/esoco/esoco/gwt-gradle-plugin/images/download.svg) ](https://bintray.com/esoco/esoco/gwt-gradle-plugin/_latestVersion)&nbsp;&nbsp;&nbsp;
[![Gitter chat](https://badges.gitter.im/gwt-gradle-plugin/gitter.png)](https://gitter.im/gwt-gradle-plugin/community)

Release notes
-------------

* 1.0.9: Merged PR #12: Make CompileGwtTask depend on all transitive project dependencies
* 1.0.8: GWT compile task output can now be cached between builds
* 1.0.7: adds `gwt-dev` to the WarPlugin.providedCompile dependency to exclude it from WAR
* 1.0.6: Supersedes 1.0.1 to 1.0.5 by fixing several bugs introduced by non-published functionality from fork
* 1.0.0: first release after fork

Project
-------

This is a fork of the ["Putnami GWT plugin"](https://github.com/Putnami/putnami-gradle-plugin)
project which seems to be no longer maintained. It has been adapted to the most recent GWT version (2.8.2) and the associated Eclipse plugin and the version has been set to 1.0.0 as the current code can be considered quite stable. The project has been renamed to `gwt-gradle-plugin` but it is not related to the equally named but even longer abandoned [project by Steffen Schaefer](https://github.com/steffenschaefer/gwt-gradle-plugin).

## Requirements ##

* Java 7 or higher
* Gradle 4 or higher (https://gradle.org)

## Usage ##

Apply the plugin **de.esoco.gwt** (or **de.esoco.gwt-lib** for library projects). To apply the plugin, add the appropriate snippet to the `build.gradle` files of 
your projects:

**Gradle >= 2.1**

```groovy
plugins {
    id "de.esoco.gwt" version "1.0.7"
}
```

In the current Gradle versions this new `plugins` declaration has some limitations. For example, it cannot be used for applying the plugin for sub-projects from a root build script. The following older declaration always works.  

**Gradle < 2.1**

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'de.esoco.gwt:gwt-gradle-plugin:1.+'
    }
}
apply plugin: 'de.esoco.gwt'
```

In the case of a multi project build make sure to apply the plugin and the plugin configuration to every project.

When the plugin has been added the following GWT-specific Gradle tasks can be invoked:
  * `gradle gwtRun` To run the application with Jetty
  * `gradle gwtDev` To run the application in develop mode
  * `gradle build`  To compile GWT and Build the application 


### On Eclipse ###

This plugin supports the most recent GWT plugin for Eclipse (available from http://www.gwtproject.org/). For eclipse support just apply the standard Eclipse Gradle plugin in the `build.gradle` file. If you want to disable the support for the Eclipse GWT plugin while still using the Gradle Eclipse plugin set the configuration option `gwtPluginEclipse` to **false** (the default is true if the Eclipse plugin is active):

```groovy
apply plugin: 'eclipse'

...

gwt {
	module 'your.gwt.module.to.compile'
	gwtPluginEclipse = false
}
```

#### Eclipse Buildship Plugin ####

The Buildship Gradle plugin for Eclipse ignores the `eclipse` task and therefore overwrites any classpath generated by plugins. To prevent this the following code needs to be added to `build.gradle`. The first entry adds the GWT plugin, the second one prevents that Buildship generates the classpath containers entry:

```groovy
eclipse {
	classpath {
		containers 'com.gwtplugins.gwt.eclipse.core.GWT_CONTAINER',
		           'org.eclipse.buildship.core.gradleclasspathcontainer'
	}
}

```

## Advanced usage ##

### Plugins ###

The GWT Gradle Plugin can be used to build GWT applications or libraries by
choosing the corresponding plugin ID:

* `apply plugin: 'de.esoco.gwt-lib'` builds a library. This installs all the GWT dependencies on your project and adds the java sources into the target jar artifact (to allow the GWT compiler in dependent projects to compile the library into JavaScript).
* `apply plugin: 'de.esoco.gwt'`builds a full web application. Apart from providing the GWT dependencies this will also build a WAR file with all server and client side code. It also provides the GWT-specific run tasks mentioned above.

### Debug Server Side ###

To debug the server side in a Jetty container the following configuration can bee added to the `build.gradle` file and then run with the `gradle gwtDev` task. This can then be attached to a remote debugger from your favorite IDE.


```groovy
gwt{
    jetty {
        /** enable debugging. */
        debugJava = true
        /** debug port to listen. */
        debugPort = 8000
        /** wait for debugger attachment. */
        debugSuspend = false
    }
}
```

### Debug Client Side ###

Because GWT compiles (or transpiles) the client side Java code to JavaScript special tools are needed to debug the original Java code although the client web browser actually executes JavaScript. This is done with the *Sourcemaps* feature of modern browsers for which the Eclipse GWT plugin contains direct support. To debug the client code with the GWT Gradle plugin the following steps are needed:

1. Run the task `gwtDev` or `gwtCodeServer`
2. Add a *Launch Chrome* launch configuration to the project in Eclipse and set the URL (for example http://127.0.0.1:8080/)
3. Run the launch configuration in debug mode
4. Add breakpoints in the Java client code as necessary 


### Tasks and configuration ###

The following are the default configuration values that will be used if no different value are set:

```groovy
gwt {
	/** Module to compile, can be multiple */
	module 'your.gwt.module.to.compile'

	/** GWT version */
	gwtVersion = '2.8.2'
	/** Add the gwt-servlet lib */
	gwtServletLib = false
	/** Add the gwt-elemental lib */
	gwtElementalLib = false
	/** Add GWT plugin config (only if plugin 'eclipse' is enabled) */
	gwtPluginEclipse = true
	/** Jetty version */
	jettyVersion = '9.4.12.v20180830'
}
```

##### `gwtCompile` - invokes the GWT Java-to-Javascript compiler

The GWT compilation can be modified in the `compile` configuration:

```groovy
gwt {
	compile {
		/** The level of logging detail (ERROR, WARN, INFO, TRACE, 
		 * DEBUG, SPAM, ALL) */
		logLevel = "INFO"
		/** Compile a report that tells the "Story of Your Compile". */
		compileReport = true
		/** Compile quickly with minimal optimizations. */
		draftCompile = true
		/** Include assert statements in compiled output. */
		checkAssertions = false
		/** Script output style. (OBF, PRETTY, DETAILED)*/
		style = "OBF"
		/** Sets the optimization level used by the compiler. 
		 * 0=none 9=maximum. */
		optimize = 5
		/** Fail compilation if any input file contains an error. */
		failOnError = false
		/** Specifies Java source level. ("1.6", "1.7")*/
		sourceLevel = "1.7"
		/** The number of local workers for compiling permutations. */
		localWorkers = 2
		/** The maximum memory to be used by local workers. */
		localWorkersMem = 2048
		/** Emit extra information allow chrome dev tools to display 
		 * Java identifiers in many places instead of JavaScript functions.
		 * (NONE, ONLY_METHOD_NAME, ABBREVIATED, FULL)*/
		methodNameDisplayMode = "NONE"

		/** Specifies JsInterop mode (NONE, JS, CLOSURE) */
		jsInteropMode = "JS"
		/** Generate and export JsInterop (since GWT 2.8) */
		generateJsInteropExports = true
		/** Include filters for the JsInterop export if enabled (since GWT 2.8.1) */
		includeJsInteropExports = ["filter1", "filter2"]
		/** Exclude filters for the JsInterop export if enabled (since GWT 2.8.1) */
		excludeJsInteropExports = ["filter1", "filter2"]

		/** Extra args can be used to experiment arguments */
		extraArgs = ["-firstArgument", "-secondArgument"]

		/** shown all compile errors */
        strict = false

		/** Java args */
		maxHeapSize="1024m"
		minHeapSize="512m"
		maxPermSize="128m"
		debugJava = true
		debugPort = 8000
		debugSuspend = false
		javaArgs = ["-Xmx256m", "-Xms256m"]
	}
}
```

##### `gwtRun` - Compile the GWT web application and run it on Jetty

**Note:** This task depends on the gwtCompile task. The Jetty execution can be modified in the `jetty` configuration:

```groovy
gwt {
	jetty {
		/** interface to listen on. */
		bindAddress = "127.0.0.1"
		/** request log filename. */
		logRequestFile
		/** info/warn/debug log filename. */
		logFile
		/** port to listen on. */
		port = 8080
		/** port to listen for stop command. */
		stopPort = 8089
		/** security string for stop command. */
		stopKey = "JETTY-STOP"

		/** Java args */
		maxHeapSize="1024m"
		minHeapSize="512m"
		maxPermSize="128m"
		debugJava = true
		debugPort = 8000
		debugSuspend = false
		javaArgs = ["-Xmx256m", "-Xms256m"]
	}
}
```

##### `gwtDev` - Compile the GWT web application and run it in development mode on Jetty

The development mode (also called *Super Dev Mode* in GWT) can be controlled with
the `dev` configuration:

```groovy
gwt {
	dev {
		/** The ip address of the code server. */
		bindAddress = "127.0.0.1"
		/** Stop compiling if a module has a Java file with a compile error, even if unused. */
		failOnError = false
		/** Precompile modules. */
		precompile = false
		/** The port where the code server will run. */
		port = 9876
		/** EXPERIMENTAL: Don't implicitly depend on "client" and "public" when a module doesn't define anydependencies. */
		enforceStrictResources = false
		/** Specifies Java source level ("1.6", "1.7"). */
		sourceLevel = "1.6"
		/** The level of logging detail (ERROR, WARN, INFO, TRACE, DEBUG, SPAM, ALL) */
		logLevel = "INFO"
		/** Specifies JsInterop mode (NONE, JS, CLOSURE). JsInterop Experimental (GWT 2.7) */
		jsInteropMode = "JS"
		/** Generate and export JsInterop (since GWT 2.8) */
		generateJsInteropExports = true
		/** Emit extra information allow chrome dev tools to display Java identifiers in many placesinstead of JavaScript functions. (NONE, ONLY_METHOD_NAME, ABBREVIATED, FULL) */
		methodNameDisplayMode = "NONE"
		/** shown all compile errors */
        strict = false
        /** disable this internal server */
        noServer = false

		/** Extra args can be used to experiment arguments */
		extraArgs = ["-firstArgument", "-secondArgument"]

		/** Java args */
		maxHeapSize="1024m"
		minHeapSize="512m"
		maxPermSize="128m"
		debugJava = true
		debugPort = 8000
		debugSuspend = false
		javaArgs = ["-Xmx256m", "-Xms256m"]
	}
}
```

## License ##
As the original Putnami project this project is provided under the LGPL-3.0.
See https://www.gnu.org/licenses/lgpl-3.0.txt for details.
