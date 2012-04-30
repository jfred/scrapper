SCRAPPER - *build script*
===========================

This was something I used once upon a time...
in a job far far away

INTRODUCTION
============

The build system is build around the idea of modules. A module is collection of files and classes that make up an associated module type.

Current supported module types are: JAR, WAR, EJB and EAR

By setting the type property you let the build know what properties and build steps should involve this module.

    project.module.${modulename}.type=${type}

An additional description property is optional, add is used for help messages.

    property.module.${modulename}.description=some descriptive text

Modules should be specified in order of dependence.  If you have a jar module, it should be specified before any of the modules that depend on classes in the jar module.  This is particularly important when dealing with war, ejb and ear modules.

Global Properties
-----------------

### IVY Integration

IVY settings (in project.properties or any other property loaded)

	build.ivy=true		the build will ivy to retrieve lib files if the lib directory is not present
	build.ivy.clean=true	the build will remove the lib directory when a clean is performed

### JSP 

If you set

	build.jsp.filter=true

JSP's will be processed through a filterset replacing any properties available in between @ symbols. I.e. @BUILD\_TIMESTAMP@ would be replaced with the build timestamp

### JavaScript

If you set
	
	build.js.optimize=true

java style comments will be removed, tabs will be converted to spaces, spaces will be trimmed and new lines will be removed (only on WAR creation - not on copy)

Running Java Classes
--------------------

The run task is used to execute any java class in the project's class path. If the property project.run.classname is set, then the Class specified will be run. If the property is not set, the user will be asked for the class.

MODULES
=======

JAR Modules
-----------

A JAR module consists of a set of classes and associated other files (log3j xml files, property files, etc.)

To configure a JAR module one additional property is requried: src

    project.module.${modulename}.src=${src.dir}

The src property specified the root of the modules classes directory. The path is relative the the ${src.base.dir}.    

By default the build will use the modulename to create the jar file (i.e., modulename.jar) or if project.name is specificed ${project.name}.jar.
If you want to override the resulting jar file, specify a distfile property

    project.module.${modulename}.distfile=someothername.jar

WAR Modules
-----------

A WAR module consists of a set of classes and files that make up a Web Application Archive.

To configure a WAR module one additional property is required: src

    project.module.${modulename}.src=${src.dir}

The src property specified is the root of the WAR, which should ususally contain a WEB-INF directory 
(the build can be overwritten to move the WEB-INF files elsewhere)    

By default the build will use the modulename to create the war file (i.e., modulename.war) or if project.name is specificed ${project.name}.war.
If you want to override the resulting war file, specify a distfile property

    project.module.${modulename}.distfile=someothername.war

If the war requires additional library files to be included (in the WEB-INF/lib directory), specify there names in the require.match property.

    project.module.${modulename).require.match=some.jar someother.jar athirdparty.jar
    
This will include any jars created by modules in the build, or available in the ${lib.base.dir} directory that match the names provided.


EJB Modules
-----------

A EJB module consists of a set of classes and files that make up a EJB Archive.

To configure a EJB module one additional property is required: src

    project.module.${modulename}.src=${src.dir}

The src property specified is the root of the EJB jar, which should ususally contain a META-INF directory 
(the build can be overwritten to move the META-INF files elsewhere)    

By default the build will use the modulename to create the war file (i.e., modulename-ejb.jar) or if project.name is specificed ${project.name}-ejb.jar.
If you want to override the resulting war file, specify a distfile property

    project.module.${modulename}.distfile=someothername-ejb.jar


EAR Modules
-----------

A EAR module consists of a set of wars, ejb-jars and files that make up a Web Application Archive.

To configure a EAR module 3 additional properties are required: META-INF.dir, META-INF.match, files

    project.module.${modulename}.META-INF.dir=etc/someear
    project.module.${modulename}.META-INF.match=*
    project.module.${modulename}.files=some.war someother.war some-ejb.jar

These properties specifiy the location and names of files that will be included in the ear's META-INF directory (like application.xml, orion-application.xml, etc.) and files to include in the war.

META-INF.dir specified the root of the configuration directory, relative to the project root (i.e., ./)
META-INF.match specifies which files will be included in the ear.
files specifies the war and ejb-jar files that will be in the ear file. The build will include and files in the ${lib.base.dir} and any modules distfiles that match.

By default the build will use the modulename to create the ear file (i.e., modulename.ear) or if project.name is specificed ${project.name}.ear. 
If you want to override the resulting war file, specify a distfile property

    project.module.${modulename}.distfile=someothername.ear

CUSTOM BEHAVIOR
===============

Task specific
-------------

To override the way a certain task is performed for all project modules, for example war creation, you specify a macrodef with the name "module-war-war". The name is composed of three section module-{type}-{target}. The type is the type of module that will be effected by this task (ie, jar, war, ejb, ear) and the target is the ant target (compile, jar, war, ejb, test...)

The Following example prints message before and after creating the war:

    <macrodef name="module-war-war">
      <attribute name="module" />
      <sequential>
        <echo>Before!</echo>
          <module-war module="@{module}"/>
        <echo>After!</echo>
      </sequential>
    </macrodef>

Module-Task specific
--------------------

Setting up a module specific version of a target is very similar to overriding a task for all modules (3.1). The name of the macrodef should be ${module.name}-${target}.

For example if we had the following module definition in project.properties:

    project.module.somewebmodulename.type=war

The Following example would prints message before and after creating the war for the "somemodulename" module:

<macrodef name="somewebmodulename-war">
  <sequential>
    <echo>Before!</echo>
    <module-war module="somewebmodulename" />
    <echo>After!</echo>
  </sequential>
</macrodef>

Target specific
---------------

### Adding Targets

Adding a new target is as simple as creating a target in any ant build.xml, just put the definition in the project.xml

For example if the following were put in the project.xml:

    <target name="bogus">
      <echo>some bogus task</echo>
    </target>

the command "ant bogus" would result in printing of "some bogus task"

### Overriding Targets

You can also completely override the targets provided (war,jar,ejb...) by specificying your own version in the project.xml

The Following example would print a message and fail if the war target is invoked:

<target name="war">
    <fail>WAR's are not supported any longer</fail>
</target>

