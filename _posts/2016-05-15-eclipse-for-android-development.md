---
layout: post
title: Using Eclipse for Android Development in 2016
date: '2016-05-15T12:00:00.000-08:00'
author: David Green
tags:
  - Android
  - Eclipse
modified_time: '2016-05-05T12:00:00.000-08:01'
comments: true
---

Using the best tool for the job has a direct impact to my overall sense of well-being and happiness - not to mention productivity.  For me when it comes to Java, that means using Eclipse.  Life is just too short for messing around.  Eclipse has been relegated to the back benches for Android development now that Android Studio is based on IntelliJ.  So what is an Eclipse die-hard to do?

I found that learning Android Studio was pretty straight-forward.  I was up and running, creating a reasonable Android app within a couple of days, forcing myself to learn the "IntelliJ-way", key bindings and all.  After a couple of weeks rolled by, then months, I realized that my creativity was being hampered because I just couldn't hit the flow in IntelliJ.  The friction was affecting me: my code had fewer unit tests than I would like, and simple tasks were taking too long.  IntelliJ was just slowing me down too much and it was impacting my project.

Seeing that [Gradle buildship](http://gradle.org/eclipse/) doesn't yet support Android projects, A quick google for ``"Gradle generate Eclipse classpath"`` and I [landed on this StackOverflow](http://stackoverflow.com/questions/17470831/how-to-use-gradle-to-generate-eclipse-and-intellij-project-files-for-android-pro).  With a few Gradle edits to `build.gradle`, I was able to generate the Eclipse `.classpath` and `.project` files needed to start editing Java source in Eclipse.  My configuration is based on the answer provided by [Johannes Brodwall](http://stackoverflow.com/users/27658/johannes-brodwall):

    apply plugin: 'eclipse'

    eclipse {
    // pathVariables 'GRADLE_HOME': gradle.gradleUserHomeDir, "ANDROID_HOME": android.sdkDirectory
        classpath {
            plusConfigurations += [ configurations.compile, configurations.testCompile ]

            file {
                beforeMerged { classpath ->
                    classpath.entries.add(new org.gradle.plugins.ide.eclipse.model.SourceFolder("src/main/java", "bin"))
                    // Hardcoded to use debug configuration
                    classpath.entries.add(new org.gradle.plugins.ide.eclipse.model.SourceFolder("build/generated/source/r/debug", "bin"))
                    classpath.entries.add(new org.gradle.plugins.ide.eclipse.model.SourceFolder("build/generated/source/buildConfig/debug", "bin"))
                    classpath.entries.add(new org.gradle.plugins.ide.eclipse.model.SourceFolder("build/generated/source/aidl/debug", "bin"))
                }

                whenMerged { classpath ->
                    def aars = []
                    classpath.entries.each { dep ->
                        if (dep.path.toString().endsWith(".aar")) {
                            def explodedDir = new File(projectDir, "build/intermediates/exploded-aar/" + dep.moduleVersion.group + "/" + dep.moduleVersion.name + "/" + dep.moduleVersion.version + "/jars/")
                            if (explodedDir.exists()) {
                                explodedDir.eachFileRecurse(groovy.io.FileType.FILES) {
                                    if (it.getName().endsWith("jar")) {
                                        def aarJar = new org.gradle.plugins.ide.eclipse.model.Library(fileReferenceFactory.fromFile(it))
                                        aarJar.sourcePath = dep.sourcePath
                                        aars.add(aarJar)
                                    }
                                }
                            } else {
                                println "Warning: Missing " + explodedDir
                            }
                        }
                    }
                    classpath.entries.removeAll { it.path.endsWith(".aar") }
                    classpath.entries.addAll(aars)

                    def androidJar = new org.gradle.plugins.ide.eclipse.model.Variable(
                        fileReferenceFactory.fromPath("ANDROID_HOME/platforms/" + android.compileSdkVersion + "/android.jar"))
                    androidJar.sourcePath = fileReferenceFactory.fromPath("ANDROID_HOME/sources/" + android.compileSdkVersion)
                    classpath.entries.add(androidJar)
                }
            }
        }
    }

    eclipseClasspath.dependsOn "generateDebugSources"

The main difference between my configuration and that provided on the StackOverflow is the addition of the `aidl` path.

To get this going the first time, or to update it whenever the project configuration changes, simply run the command:

    $ gradle eclipse

Or, if using the AndroidStudio-generated `gradlew`:

    $ ../gradlew eclipse

Within a couple of minutes I had the Java imports organized and compiler warnings gone, was refactoring in anger and writing unit tests with one of my favourite Eclipse plug-ins, [MoreUnit](https://marketplace.eclipse.org/content/moreunit).
