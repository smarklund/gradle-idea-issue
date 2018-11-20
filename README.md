# Gradle 4.7 Regression in Idea Plugin

The purpose of this repository is illustrate the regression in generation of Idea modules introduced in Gradle 4.7
and onwards including Gradle 5.0-rc-3 when the source directory folder is set to the same folder as the resources folder.
The same issue occurs when the test source directory is set to the same as the test resources directory.

I am aware that recommended convention is to organise code as src/{main,test}/{java,resources}.
However, we have a lot of (old) code with the 'java' and 'test' folders set this way and I assume we are not alone
out there with repositories set up this way.

## Project Setup

* java directory - Contains source code as well as properties and xml files
  * Because of this both sourceSets.main.java and sourceSets.main.resources are set the 'java' directory.

* test directory - Contains the test source code as well as resources related to testing
  * Because of this both sourceSets.test.java and sourceSets.test.resources are set the 'test' directory.

## How to Test

* Run the wrapper task to generate wrapper files for the version of Gradle to test.
* Run the 'cleanIdea' and 'idea' tasks to generate a new environment.
* Import into Intellij Idea.

## Cause of Issue

The function addSourceAndExcludeFolderToXml in subprojects/ide/src/main/java/org/gradle/plugins/ide/idea/model/Module.java
currently have some logical issues. 

* If a folder is listed as a resources folder then no source folder is generated.
* If a folder is listed as a source folder then no resources folder is generated.
* Combining those two checks makes no folder at all being generated.

That is for Gradle 5. I have not looked exactly why version 4.7 onwards flagged the folders as resources instead of sources.

## Issue remedy

If a sources folder share location with a resources folder then flagging it as a sources should take priority.
This goes for both sources and test sources folders.

## Gradle 4.6 - Fully working

 * java directory flagged as source code
 * test directory flagged as test source code

### Generated iml file
```
./gradlew wrapper --gradle-version=4.6
./gradlew cleanIdea idea
> cat gradle-idea-issue.iml 
<?xml version="1.0" encoding="UTF-8"?>
<module relativePaths="true" type="JAVA_MODULE" version="4">
  <component name="NewModuleRootManager" inherit-compiler-output="true">
    <exclude-output/>
    <orderEntry type="inheritedJdk"/>
    <content url="file://$MODULE_DIR$/">
      <sourceFolder url="file://$MODULE_DIR$/java" isTestSource="false"/>
      <sourceFolder url="file://$MODULE_DIR$/test" isTestSource="true"/>
      <excludeFolder url="file://$MODULE_DIR$/.gradle"/>
      <excludeFolder url="file://$MODULE_DIR$/build"/>
    </content>
    <orderEntry type="sourceFolder" forTests="false"/>
  </component>
  <component name="ModuleRootManager"/>
</module>
 
```

## Gradle 4.7,4.8, 4.9, 4.10 - Java and test folders are flagged as resource folders
 * java directory flagged as "java-resource"
 * test directory flagges as "java-test-resource"

Having the 'java' and 'test' directories be flagged as resources and test resources 
causes the Java files to be copied like resources and not compiled. The resource handling is still working ok though.

### Generated iml file
```
./gradlew wrapper --gradle-version=4.7
./gradlew cleanIdea idea

> cat gradle-idea-issue.iml 
<?xml version="1.0" encoding="UTF-8"?>
<module relativePaths="true" type="JAVA_MODULE" version="4">
  <component name="NewModuleRootManager" inherit-compiler-output="true">
    <exclude-output/>
    <orderEntry type="inheritedJdk"/>
    <content url="file://$MODULE_DIR$/">
      <sourceFolder url="file://$MODULE_DIR$/java" type="java-resource"/>
      <sourceFolder url="file://$MODULE_DIR$/test" type="java-test-resource"/>
      <excludeFolder url="file://$MODULE_DIR$/.gradle"/>
      <excludeFolder url="file://$MODULE_DIR$/build"/>
    </content>
    <orderEntry type="sourceFolder" forTests="false"/>
  </component>
  <component name="ModuleRootManager"/>
</module>
```



## Gradle 5.0-rc-3 - Java and test folders are not generated at all
 * java directory not generated at all
 * test directory not generated at all

No source folders are generated.

### Generated iml file

```
./gradlew wrapper --gradle-version=5.0-rc-3
./gradlew cleanIdea idea
> cat gradle-idea-issue.iml 
<?xml version="1.0" encoding="UTF-8"?>
<module relativePaths="true" type="JAVA_MODULE" version="4">
  <component name="NewModuleRootManager" inherit-compiler-output="true">
    <exclude-output/>
    <orderEntry type="inheritedJdk"/>
    <content url="file://$MODULE_DIR$/">
      <excludeFolder url="file://$MODULE_DIR$/.gradle"/>
      <excludeFolder url="file://$MODULE_DIR$/build"/>
    </content>
    <orderEntry type="sourceFolder" forTests="false"/>
  </component>
  <component name="ModuleRootManager"/>
</module>

```

