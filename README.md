# Baseline Java code quality plugins

[![Build Status](https://magnum.travis-ci.com/palantir/gradle-baseline.svg?token=7PgjAzx6cN9JEwNu1QCp&branch=develop)](https://magnum.travis-ci.com/palantir/gradle-baseline)

Baseline Java is a collection of Gradle plugins for configuring code quality tools in builds and generated
Eclipse/IntelliJ projects. It configures ([Checkstyle](http://checkstyle.sourceforge.net) for style and formatting
checks, [FindBugs](http://findbugs.sourceforge.net/) for catching common bugs, and Eclipse/IntelliJ code style and
formatting configurations.

The Baseline plugins are compatible with Gradle 2.2.1 and above.






## Quick start
- Extract the `baseline-config-<version>.zip` (available from the
[Bintray](https://bintray.com/palantir/releases/gradle-jacoco-coverage) repository) file into the `.baseline` directory
of a Gradle root project.
- Add the Baseline plugins to the `build.gradle` configuration of the Gradle project:

```Gradle
buildscript {
    dependencies {
        classpath 'com.palantir:gradle-baseline-java:<version>'
    }
}

// Apply plugins selectively depending on required functionality.
apply plugin: 'com.palantir.baseline-checkstyle'
apply plugin: 'com.palantir.baseline-eclipse'
apply plugin: 'com.palantir.baseline-findbugs'
apply plugin: 'com.palantir.baseline-idea'
```

- Any subsequent ``./gradlew build`` invokes Checkstyle and FindBugs as part of the build and test tasks (if the
respective baseline-xyz plugins are applied).
- The ``eclipse`` and ``idea`` Gradle tasks generate projects pre-configured with Baseline settings:

   - Code style and code formatting rules conforming with Baseline style
   - Checkstyle and FindBugs configuration

  Note that the Checkstyle-IDEA plugin is required to run the Baseline Checkstyle within IntelliJ.




## Plugin Architecture Overview

The Baseline plugins `com.palantir.baseline-checkstyle`, `com.palantir.baseline-eclipse`,
`com.palantir.baseline-findbugs`, `com.palantir.baseline-idea` apply the configuration present in `.baseline` to the
respective Gradle tasks. For example, any Gradle Checkstyle tasks uses the Checkstyle configuration in
`.baseline/checkstyle/checkstyle.xml`, and any IntelliJ/Eclipse project generated by `./gradlew eclipse idea` is
configured with Baseline code formatting and Checkstyle rules. Note that each of these plugins automatically applies the
underlying Gradle plugin: `com.palantir.baseline-checkstyle` applies `checkstyle`, `com.palantir.baseline-eclipse`
applies `eclipse`, etc.





## Configuration

The standard Gradle configuration options for the underlying plugins (Eclipse, IntelliJ, Checkstyle, FindBugs) can be
used, with the following exception:

- `checkstyle.configFile` - not compatible with Baseline since the file location is hard-coded to
`.baseline/checkstyle/checkstyle.xml`






## Advanced usage

### Multiple-project builds

All `com.palantir.baseline-xyz` plugins can be applied selectively to subprojects. For example:

```Gradle
buildscript {
    dependencies {
        classpath 'com.palantir:gradle-baseline-java:<version>'
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
        classpath 'com.palantir:gradle-baseline-java:<version>'
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

To suppress a check from a single line, add the following to the previous line, or after the line.  For example, to
ignore the `MagicNumberCheck`:

```Java
reverseEngineerQuestion(42); // CHECKSTYLE IGNORE MagicNumberCheck
```

To disable Checkstyle for a set of lines, surround the lines with:

```Java
// CHECKSTYLE:OFF
badCode()
// CHECKSTYLE:ON
```

Finally, to disable certain checks for an entire file, apply [custom
suppressions](http://checkstyle.sourceforge.net/config.html).


### Eclipse Plugin (com.palantir.baseline-eclipse)

Run `./gradlew eclipse` to repopulate projects from the templates in `.baseline`.

The `com.palantir.baseline-eclipse` plugin automatically applies the `eclipse` plugin, but not the `java` plugin. The
`com.palantir.baseline-eclipse` plugin has no effects if the `java` plugin is not applied.

If `sourceCompatibility` is set, it is used to configure the Eclipse project settings and the Eclipse JDK version. Note
that `targetCompatibility` is also honored and defaults to `sourceCompatibility`.

Generated Eclipse projects have default per-project code formatting rules as well as Checkstyle and FindBugs
configuration.

The Eclipse plugin is compatible the following versions: Checkstyle 6.5+, FindBugs 3.0.0+, JDK 1.7, 1.8


### IntelliJ Plugin (com.palantir.baseline-idea)

Run `./gradlew idea` to (re-) generate IntelliJ project and module files from the templates in `.baseline`. The
generated project is pre-configured with Baseline code style settings and support for the Checkstyle-IDEA plugin.

The `com.palantir.baseline-idea` plugin automatically applies the `idea` plugin.

Generated IntelliJ projects have default per-project code formatting rules as well as Checkstyle configuration. The JDK
and Java language level settings are picked up from the Gradle `sourceCompatibility` property on a per-module basis.


### FindBugs Plugin (com.palantir.baseline-findbugs)

Checks can be suppressed by annotating the class/method/field in question with:

```Java
@SuppressFBWarning("BUG_PATTERN_NAME")
```

The BUG_PATTERN_NAME can be derived from the "Pattern" field in the Eclipse Bug Info View.

More complicated filters can be handled via the `.baseline/findbugs/excludeFilter.xml` file; see [FindBugs
documentation](http://findbugs.sourceforge.net/manual/filter.html) for details.


### Jacoco Coverage Plugin (jacoco-coverage)

Palantir also maintains the [GitHub: jacoco-coverage](https://github.com/palantir/gradle-jacoco-coverage) plugin which
enforces minimum code coverage thresholds.