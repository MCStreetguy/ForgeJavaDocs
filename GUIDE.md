# Generating Forge JavaDocs

This document describes my personal method for generating the Forge JavaDocs.
It is by no means a very stable or efficient approach and i make no guarantees that this will continue to work in the future.
It is based on a custom Gradle task, added to the official Forge repository source to generate a combined JavaDoc for all packages.

If you got access to the IntelliJ IDEA IDE and this approach does not work for you, please also check out [the guide made by Nekoyue](https://github.com/Nekoyue/ForgeJavaDocs-NG/blob/master/GUIDE.md) in their predecessor repository.

You can also try to use Eclipse for this purpose, but be prepared for some tinkering and errors. I never got it to work that way and personally find this method far easier, but if you really want to try it out please go ahead, I wish you good luck.

## Requirements

Please make sure that you have the corresponding Java Development Kit version installed for the Forge version you want to address!

## Preparation

1. Clone the [official Forge git repository](https://github.com/MinecraftForge/MinecraftForge) and check out the tag of the version you want to address.  
   This can be simply done using the following command:  
   `git clone --depth=1 --branch=<TAG_NAME> https://github.com/MinecraftForge/MinecraftForge.git`
2. Fully setup the workspace by running the `./gradlew setup` (or `.\gradlew.bat setup`) command in the project directory.
   This step is crucial as it downloads and decompiles the actual Minecraft sources required for the project to work.

## Generator Snippet

Open the `build.gradle` file from the project root directory in your favorite editor and append the following snippet at the end of the file:

```gradle
def exportedProjects = [
    ":forge",
    ":fmlcore",
    ":fmlearlydisplay",
    ":fmlloader",
    ":javafmllanguage",
    ":lowcodelanguage",
    ":mclanguage"
]

task alljavadoc(type: Javadoc) {
    options.tags = [
        'apiNote:a:<em>API Note:</em>',
        'implSpec:a:<em>Implementation Requirements:</em>',
        'implNote:a:<em>Implementation Note:</em>'
    ]
    options.addStringOption('Xdoclint:none', '-private')
    options.addStringOption('encoding', 'utf-8')
    options.addStringOption('docencoding', 'utf-8')
    options.addStringOption('charset', 'utf-8')
    options.addStringOption('windowtitle', 'Forge 47.1.0 Modding API for Minecraft 1.20.1')
    options.addStringOption('doctitle', 'Forge 47.1.0 Modding API for Minecraft 1.20.1')

    source exportedProjects.collect { project(it).sourceSets.main.allJava }
    classpath = files(exportedProjects.collect { project(it).sourceSets.main.compileClasspath })
    destinationDir = file("${buildDir}/docs/javadoc")
}
```

You must adapt this snippet slightly:

1. Make sure all required projects are included in the `exportedProjects` list.  
   _(This normally means all but the following projects: "clean", "fmlonly", "mdk")_
2. Also make sure that no projects are included, that do not exist in the workspace!
3. Change the versions for both Forge and Minecraft in the options `windowtitle` and `doctitle`.
4. _(optional)_ Change the `destinationDir` path if you want to store the docs somewhere else than the default `build/docs/javadoc` directory.

## Building

All you still have to do now is to run `./gradlew alljavadoc` (or `.\gradlew.bat alljavadoc` respectively).
This will go through all projects and generate a combined JavaDoc for all of them in the configured location.

Please note that a lot of warnings will be thrown during the process, but these can safely be ignored.
Most of them complain about missing reference links or inproper formatting and do not really affect the result.
As long as you get no errors you should be fine.

## Troubleshooting

If you run into any errors:

1. Make sure the `exportedProjects` list is correct! _This is the most common source of errors in this process._
2. Make sure all options have been changed correctly and that the overall syntax of the file is still valid.
