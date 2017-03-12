# Baseline Java code quality plugins

[![CircleCI Build Status](https://circleci.com/gh/palantir/gradle-baseline/tree/develop.svg?style=shield)](https://circleci.com/gh/palantir/gradle-baseline)
[![Bintray Release](https://api.bintray.com/packages/palantir/releases/gradle-baseline/images/download.svg) ](https://bintray.com/palantir/releases/gradle-baseline/_latestVersion)

Baseline Java is a collection of Gradle plugins for configuring code quality tools in builds and generated
Eclipse/IntelliJ projects. It configures [Checkstyle](http://checkstyle.sourceforge.net) for style and formatting
checks, and Eclipse/IntelliJ code style and formatting configurations.

The Baseline plugins are compatible with Gradle 2.2.1 and above.






## Quick start
- Add the Baseline plugins to the `build.gradle` configuration of the Gradle project:

```Gradle
buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'com.palantir.baseline:gradle-baseline-java:<version>'
    }
}

repositories {
    jcenter()
}

dependencies {
    // Adds a dependency on the Baseline configuration files. Typically use 
    // the same version as the plugin itself.
    baseline "com.palantir.baseline:gradle-baseline-java-config:<version>@zip"
}

apply plugin: 'java'

// Apply for baselineUpdateConfig task
apply plugin: 'com.palantir.baseline-config'

// Apply plugins selectively depending on required functionality.
apply plugin: 'com.palantir.baseline-checkstyle'
apply plugin: 'com.palantir.baseline-eclipse'
apply plugin: 'com.palantir.baseline-idea'
```

- Run ``./gradlew baselineUpdateConfig`` to download the config files
referenced in the `dependencies.baseline` configuration and extract them to .baseline/
- Any subsequent ``./gradlew build`` invokes Checkstyle as part of the build and test tasks (if the
respective baseline-xyz plugins are applied).
- The ``eclipse`` and ``idea`` Gradle tasks generate projects pre-configured with Baseline settings:

   - Code style and code formatting rules conforming with Baseline style
   - Checkstyle configuration

  Note that the Checkstyle-IDEA plugin is required to run the Baseline Checkstyle within IntelliJ.




## Development environment
Tests are run with `./gradlew publishToMavenLocal build`, the publishing step is required in order to make Baseline
artifacts available to the tests. Note that some of the tests only work when run for the first time since they assume
particular directory structures that are unavailable when re-running tests.

IDE configurations can be generated with `./gradlew idea eclipse`.





## Plugin Architecture Overview

The Baseline plugins `com.palantir.baseline-checkstyle`, `com.palantir.baseline-eclipse`,
`com.palantir.baseline-idea` apply the configuration present in `.baseline` to the
respective Gradle tasks. For example, any Gradle Checkstyle tasks uses the Checkstyle configuration in
`.baseline/checkstyle/checkstyle.xml`, and any IntelliJ/Eclipse project generated by `./gradlew eclipse idea` is
configured with Baseline code formatting and Checkstyle rules. Note that each of these plugins automatically applies the
underlying Gradle plugin: `com.palantir.baseline-checkstyle` applies `checkstyle`, `com.palantir.baseline-eclipse`
applies `eclipse`, etc.





## Configuration

The standard Gradle configuration options for the underlying plugins (Eclipse, IntelliJ, Checkstyle) can be
used, with the following exception:

- `checkstyle.configFile` - not compatible with Baseline since the file location is hard-coded to
`.baseline/checkstyle/checkstyle.xml`






## Advanced usage

### Multiple-project builds

All `com.palantir.baseline-xyz` plugins can be applied selectively to subprojects. For example:

```Gradle
buildscript {
    dependencies {
        classpath 'com.palantir.baseline:gradle-baseline-java:<version>'
    }
}

apply plugin: 'com.palantir.baseline-idea'

subprojects {
    apply plugin: 'java'
    apply plugin: 'com.palantir.baseline-checkstyle'
    apply plugin: 'com.palantir.baseline-idea'
}
```

Depending on the Gradle setup, you may need to edit `gradle/shared.gradle` (or similar) instead. Feel free to contact
the Baseline mailing list for troubleshooting.


### Applying Baseline plugins selectively or all at once

The `com.palantir.baseline` plugin applies all `com.palantir.baseline-xyz` plugins to the current project. In order to
use only Checkstyle and IntelliJ support from Baseline, apply the required plugins selectively, e.g.:

```Gradle
buildscript {
    dependencies {
        classpath 'com.palantir.baseline:gradle-baseline-java:<version>'
    }
}

apply plugin: 'com.palantir.baseline-idea'
subprojects {
    apply plugin: 'com.palantir.baseline' // Applies all com.palantir.baseline-xyz plugins
}
```





### Checkstyle Plugin (com.palantir.baseline-checkstyle)

Checkstyle rules can be suppressed on a per-line or per-block basis. (It is good practice to first consider formatting
the code block in question according to the project's style guidelines before adding suppression statements.) To
suppress a particular check, say `MagicNumberCheck`, from an entire class or method, annotate the class or method with
the lowercase check name without the "Check" suffix:

```Java
@SuppressWarnings("checkstyle:magicnumber")
```

Checkstyle rules can also be suppressed using comments, which is useful for checks such as `IllegalImport` where
annotations cannot be used to suppress the violation. To suppress checks for particular lines, add the comment
`// CHECKSTYLE:OFF` before the first line to suppress and add the comment `// CHECKSTYLE:ON` after the last line.

To disable certain checks for an entire file, apply [custom suppressions](http://checkstyle.sourceforge.net/config.html)
in `.baseline/checkstyle/checkstyle-suppressions`.


### Eclipse Plugin (com.palantir.baseline-eclipse)

Run `./gradlew eclipse` to repopulate projects from the templates in `.baseline`.

The `com.palantir.baseline-eclipse` plugin automatically applies the `eclipse` plugin, but not the `java` plugin. The
`com.palantir.baseline-eclipse` plugin has no effects if the `java` plugin is not applied.

If set, `sourceCompatibility` is used to configure the Eclipse project settings and the Eclipse JDK version. Note
that `targetCompatibility` is also honored and defaults to `sourceCompatibility`.

Generated Eclipse projects have default per-project code formatting rules as well as Checkstyle configuration.

The Eclipse plugin is compatible with the following versions: Checkstyle 7.5+, JDK 1.7, 1.8


### IntelliJ Plugin (com.palantir.baseline-idea)

Run `./gradlew idea` to (re-) generate IntelliJ project and module files from the templates in `.baseline`. The
generated project is pre-configured with Baseline code style settings and support for the Checkstyle-IDEA plugin.

The `com.palantir.baseline-idea` plugin automatically applies the `idea` plugin.

Generated IntelliJ projects have default per-project code formatting rules as well as Checkstyle configuration. The JDK
and Java language level settings are picked up from the Gradle `sourceCompatibility` property on a per-module basis.

### Error-prone

Baseline does not provide a dedicated FindBugs plugin or configuration, but it recommends to configure Google's
[error-prone](https://github.com/google/error-prone) compiler to find and prevent common Java coding mistakes. To
configure error-prone, add the following Gradle configuration:

```groovy
buildscript {
    dependencies {
        classpath 'net.ltgt.gradle:gradle-errorprone-plugin:0.0.9'
    }
}

apply plugin: 'net.ltgt.errorprone'
dependencies {
    errorprone 'com.google.errorprone:error_prone_core:2.0.18'  // update version as desired
}

```

Tip: Warnings on generated code can be suppressed as follows:

```groovy
compileJava {
    options.compilerArgs += ['-XepDisableWarningsInGeneratedCode']
}
```



### Copyright Checks

By default Baseline enforces Palantir copyright at the beginning of files. To change this, edit the template copyright
in `.baseline/copyright/*.txt` and the RegexpHeader checkstyle configuration in `.baseline/checkstyle/checkstyle.xml`
