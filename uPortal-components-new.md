# Develop Vue Web Components for uPortal

## Step by Step guide for vue.js component
1. [Prerequisites](#1-prerequisites)
    - [Node.js](#node.js)
    - [Vue CLI](https://cli.vuejs.org/)
    - [Maven](https://maven.apache.org/)
    - [Gradle](https://gradle.org/)
2. [Generate the Vue app](#2-generate-the-vue-app)
3. [Edit the Vue app](#3-edit-the-vue-app)
4. [Assemble and deploy the Vue app](#4-assemble-and-deploy-the-vue-app)
5. [Add the component into uPortal](#5-add-the-component-into-uportal)

## 1. Prerequisites
#### Node.js
If you don't have node installed there are several ways to do it. One
way is to use [Node Version Manager](https://github.com/creationix/nvm)
(nvm):

``` bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

Install the latest Long Term Support (LTS) version of node (currently 10.15.1):

``` bash
nvm install node
```

#### Vue CLI
If you dont have Vue cli installed:
``` bash
npm install --global @vue/cli
```

#### Maven
Use the appropriate package manager for your OS. These instructions were
tested with maven version 3.6.

#### Gradle
Use the appropriate package manager for your OS. These instructions were
tested with gradle 5.2.

## 2. Generate the Vue app
Replace `{component-name}` with the desired name for the component.

``` bash
vue create {component-name} --default
```

Install dependencies for legacy browser support in the newly generated app:
``` bash
cd {component-name}

npm install --save-dev @babel/{cli,plugin-transform-runtime,preset-env}
```


## 3. Edit the Vue app
In the root directory, create a **gradle.properties** file, with the
following content:
```
group=org.webjars.npm
```


Copy build.gradle file from @uportal directory of uPortal-web-components
project:

https://github.com/uPortal-contrib/uPortal-web-components/blob/master/%40uportal/build.gradle

Remove the subprojects line and its enclosing brackets from build.gradle
file. For example:

``` diff
- subprojects {

    apply plugin: 'java'
    apply plugin: 'maven'

    def jsonFile = file("${projectDir}/package.json")
    def parsedJson = new groovy.json.JsonSlurper().parseText(jsonFile.text)
    project.version = parsedJson.version + '-SNAPSHOT'

    jar {
        baseName "uportal__${project.name}"
        from '.'
        exclude 'build'
        exclude 'node_modules'
        into "META-INF/resources/webjars/uportal__${project.name}/${project.version}"
    }

- }
```

or

``` gradle
    apply plugin: 'java'
    apply plugin: 'maven'

    def jsonFile = file("${projectDir}/package.json")
    def parsedJson = new groovy.json.JsonSlurper().parseText(jsonFile.text)
    project.version = parsedJson.version + '-SNAPSHOT'

    jar {
        baseName "uportal__${project.name}"
        from '.'
        exclude 'build'
        exclude 'node_modules'
        into "META-INF/resources/webjars/uportal__${project.name}/${project.version}"
    }
```


## Got to here


Change these:

``` diff
- "build": "vue-cli-service build",
+ "prebuild": "babel node_modules/@vue/web-component-wrapper/dist/vue-wc-wrapper.js -o node_modules/@vue/web-component-wrapper/dist/vue-wc-wrapper.js",
+ "build": "vue-cli-service build --name {component-name} --target wc src/components/{component-name}.vue",
```

Javascript sample:

``` Javascript
  var Button = function (element, options) {
    this.$element  = $(element)
    this.options   = $.extend({}, Button.DEFAULTS, options)
    this.isLoading = false
  }
```

Replace:
```
```
With:


More stuff here:

```
    1. In the root directory, create a gradle.properties file, with the following content:

    group=org.webjars.npm

    2. Copy build.gradle file from @uportal directory of uPortal-web-components project:

		https://github.com/uPortal-contrib/uPortal-web-components/blob/master/%40uportal/build.gradle

		Remove the subprojects line and its enclosing brackets from build.gradle file.

    3. Add the gradlew wrapper to the project

    $ gradle wrapper --gradle-version=5.1.1

    4. Rename generated HelloWorld file:

    rename src/components/HelloWorld.vue => src/components/{component-name}.vue

    5. Rename imports. In the main App file.

    # In src/App

    import HelloWorld from './components/HelloWorld.vue' => import HelloWorld from './components/{component-name}.vue'

5. Editing the package.json file

    # First edit the sripts:

    - "build": "vue-cli-service build",
    + "prebuild": "babel node_modules/@vue/web-component-wrapper/dist/vue-wc-wrapper.js -o node_modules/@vue/web-component-wrapper/dist/vue-wc-wrapper.js",
    + "build": "vue-cli-service build --name {component-name} --target wc src/components/{component-name}.vue",

	# NOTE: the --name {component-name} must have a hyphen, for example speedy-vue.

    # Add these top level declarations:

    + "main": "dist/{component-name}.js",
    + "source": "src/components/{component-name}.vue",

6. Replace the content of babel.config.js file with this:

    module.exports = {
    presets: ['@babel/preset-env'],
        plugins: [['@babel/plugin-transform-runtime', { useESModules: true }]]
    };



```

#### 4. Assemble and deploy the Vue app

```
    # Packs the component

    $ npm run build

    # Publish to the local maven repo

    $ ./gradlew install
```

#### 5. Add the component into uPortal

```
    1. Make a copy of the portlet definition file and rename:

     data/quickstart/portlet-definition/admin-dashboard.portlet-definition.xml =>  data/quickstart/portlet-definition/{component-name}.portlet-definition.xml

    2. In the copied file make the follwing changes to these fields:

        <title>{component-name}</title>
        <name>{component-name}</name>
        <fname>{component-name}</fname>
        <desc>My new sample Vue component</desc>

    3. change
		<parameter>                        =>         <parameter>
			<name>chromeStyle</name>   =>            <name>showChrome</name>
			<value>no-chrome</value>   =>            <value>true</value>
		</parameter>                       =>         </parameter>

    4. Replace the CDATA section of the file with this:

    <![CDATA[
           <script src="https://unpkg.com/vue"></script>
           <script type="text/javascript" src="/resource-server/webjars/uportal__{component-name}/dist/{component-min-name}.min.js"></script>
            <{component-name}></{component-name}>
       ]]>


	***to find the component min name
	    $ cd ~/.m2/repository/org/webjars/npm/uportal__{component-name}/0.1.0-SNAPSHOT/

	    $ jar tvf uportal__{whatever you found there}-0.1.0-SNAPSHOT.jar | grep min.js
	this will give you the right name of the min.js file {component-min-name}

    5. Add the webjar as a dependency:

        In overlays/resource-server/build.gradle, add the follwing runtime dependency:

             runtime "org.webjars.npm:uportal__{component-name}:0.1.0-SNAPSHOT@jar"

```

# Appendix
## Node.js installation
### Mac OS X
1. [MacPorts](https://www.macports.org/)
2. [HomeBrew](https://brew.sh/)
3. macOS installer (.pkg) from [Node.js website](https://nodejs.org/)

#### With MacPorts
```
sudo port list | grep node
sudo port install nodejs10
```
#### With Homebrew
```
brew search node
brew install node
```




