# Maven-NodeJS sample project

Sample project to show how to configure Maven to run NodeJS modules during the build.

This sample setup demonstrates how to use [JAM](http://jamjs.org/) to manage client side JS packages (and also shows how to compile them into one JS) and the usage of the [LESS](http://lesscss.org/) compiler is also presented.

## How to use

Simply download/fork and modify the source. To build the project enter:

`> mvn install`

This will:

1. If NodeJS environment is not ready (`nodejs` directory doesn't exist)
  * Extract the NodeJS binary suitable for your system to the `nodejs` directory (using the [nodejs-maven-plugin](https://github.com/skwakman/nodejs-maven-plugin))
  * Download the NPM package manager and extract to the `nodejs` directory (using the [maven-download-plugin](https://github.com/maven-download-plugin/maven-download-plugin))
  * Call the NPM using the [exec-maven-plugin](http://mojo.codehaus.org/exec-maven-plugin/) to install JAM and LESS modules
2. Use the [exec-maven-plugin](http://mojo.codehaus.org/exec-maven-plugin/) to call the JAM and LESS modules like if we would enter the following commands into the command line:
  * `jam install`
  * `jam compile -i app.js ${basedir}/src/main/webapp/resources/js/app.js`
  * `lessc style.less ${basedir}/src/main/webapp/resources/css/style.css`
  

### JavaScript

To add a new JS dependency for JAM edit the `src/main/javascript/package.json` file and add it to the `dependencies` section. More information about the contents of this file can be found in the official [JAM documentation](http://jamjs.org/docs).

For your own code use the `src/main/javascript` directory. You can create subdirectories for more complex projects. Don't forget to enumerate the JS module in the `app.js` require call to make sure the optimizer adds it to the compiled code.

### LESS

Similarly to the JavaScript `app.js` LESS sources also have a "bootstrap" file in this setup, called `style.less`. This file is meant to aggregate all LESS stylesheets for the compiler, so it doesn't contain anything but `@import` statements.

So to add your own stylesheets create the `.less` files in the `src/main/less` directory, then add the corresponding `@import` statement to the `style.less`. Again for complex projects you can of course separate your stylesheet files by organizing them into subdirectories.

## Customization

This setup relies on browser caching capabilities when creating monolithic JS and CSS files: so that on first page load they will be cached and subsequent requests will be served from that cached version.

However, if you need customized builds for certain scenarios (for example for mobile) you can easily extend the basic build with your custom profiles.

For example to create this previously mentioned mobile version, create a `mobile.js` (similar to `app.js` to collect dependencies for mobile) in the `src/main/javascript` directory, then add the following `execution` section to the `exec-maven-plugin` in the `pom.xml`:

```XML
<execution>
  <id>JAM compile (mobile)</id>
  <phase>generate-sources</phase>
  <goals>
    <goal>exec</goal>
  </goals>
  <configuration>
    <workingDirectory>${basedir}/src/main/javascript</workingDirectory>
    <executable>${nodejs.directory}/node</executable>
    <arguments>
      <argument>${nodejs.directory}/node_modules/jamjs/bin/jam.js</argument>
      <argument>compile</argument>
      <argument>-i</argument>
      <argument>mobile.js</argument>
      <argument>${basedir}/src/main/webapp/resources/js/mobile.js</argument>
    </arguments>
  </configuration>
</execution>
```

For LESS you can do a similar thing: add a `mobile.less` to the `src/main/less` directory, and add the following `execution` section to the `pom.xml`:

```XML
<execution>
  <id>LESS compile (mobile)</id>
  <phase>generate-sources</phase>
  <goals>
    <goal>exec</goal>
  </goals>
  <configuration>
    <workingDirectory>${basedir}/src/main/less</workingDirectory>
    <executable>${nodejs.directory}/node</executable>
    <arguments>
      <argument>${nodejs.directory}/node_modules/less/bin/lessc</argument>
      <argument>-x</argument>
      <argument>mobile.less</argument>
      <argument>${basedir}/src/main/webapp/resources/css/mobile.css</argument>
    </arguments>
  </configuration>
</execution>
```
