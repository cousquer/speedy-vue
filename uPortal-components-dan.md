# Vue Portal Components
# Contents
    1. Step by step guide for vue.js components

### Step by Step guide for vue.js component
1. Prerequisites
2. Generating the vue app
3. Editing the vue app
4. Assembling and deploying the vue app
5. Adding the component into uPortal
#### Prerequisites
1. Node
    - If you don't have node installed
    ```
        # Install nvm:

        $ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash

        # Install the latest LTS version of node (as of time of publication this is 10.15.1):

        $ nvm install node

    ```
2. Vue CLI
    - If you dont have vue cli installed:
    ```
        # Install vue cli:
        $ npm install --global @vue/cli
    ```
3. Maven
    - Use the appropriate package manager for your OS (These instructions were tested with maven version 3.6)
4. Gradle
    - Use the appropriate package manager for your OS (These instructions were tested with gradle 5.2)
#### Generating the Vue app
```
    # replace {component-name} with the desired name for the component
    $ vue create {component-name} --default
    $ cd {component-name}
    # Install dependencies
    $ npm install --save-dev @babel/{cli,plugin-transform-runtime,preset-env}

```
#### Editing the vue app
1. In the root directory, create a gradle.properties file, with the following content:
    ```
    group=org.webjars.npm
    ```
2. Copy build.gradle file from @uportal directory of uPortal-web-components project:
    https://github.com/uPortal-contrib/uPortal-web-components/blob/master/%40uportal/build.gradle
    Remove the subprojects line and its enclosing brackets from build.gradle file.
3. Add the gradlew wrapper to the project
    ```
    $ gradle wrapper --gradle-version=5.1.1
    ```
4. Rename generated HelloWorld file:
    | Old Name                                   | New Name                      |
    | ----------------------------------------- | -------------------------------- |
    | src/components/HelloWorld.vue| src/components/{component-name}.vue|
5. Replace imports. In the main App.js file (src/App).
    | Old Import                                   | New Import                      |
    | ----------------------------------------- | -------------------------------- |
    | import HelloWorld from './components/HelloWorld.vue'| import HelloWorld from './components/{component-name}.vue'|
6. Editing the package.json file
    6.1 First edit the sripts:
```diff
- "build": "vue-cli-service build",
+ "prebuild": "babel node_modules/@vue/web-component-wrapper/dist/vue-wc-wrapper.js -o node_modules/@vue/web-component-wrapper/dist/vue-wc-wrapper.js",
+ "build": "vue-cli-service build --name {component-name} --target wc src/components/{component-name}.vue",
```

    NOTE: the --name {component-name} must have a hyphen, for example speedy-vue.
    6.2 Add these top level declarations:
    ```
    "main": "dist/{component-name}.js",
    "source": "src/components/{component-name}.vue",
    ```
7. Replace the content of babel.config.js file with this:
    ```
    module.exports = {
    presets: ['@babel/preset-env'],
        plugins: [['@babel/plugin-transform-runtime', { useESModules: true }]]
    };
    ```
#### Assembling the vue app
        # Packs the component
        $ npm run build
        # Publish to the local maven repo
        $ ./gradlew install

#### Adding the component into uPortal
1. Make a copy of the portlet definition file and rename:
    | Old File                                  | New File                      |
    | ----------------------------------------- | -------------------------------- |
    | data/quickstart/portlet-definition/admin-dashboard.portlet-definition.xml| data/quickstart/portlet-definition/{component-name}.portlet-definition.xml|
2. In the copied file make update these fields:
    ```xml
    <title>{component-name}</title>
        <name>{component-name}</name>
        <fname>{component-name}</fname>
    <desc>My new sample Vue component</desc>
    ```
3. To add chrome:
    | Old parameter value | New Parameter value                      |
    | ----------------------------------------- | -------------------------------- |
    | `<parameter><name>chromeStyle</name><value>no-chrome</value></parameter>`| `<parameter><name>showChrome</name><value>true</value></parameter>`|

4. Replace the CDATA section of the file with this:
    ```
    <![CDATA[
            <script src="https://unpkg.com/vue"></script>
            <script type="text/javascript" src="/resource-server/webjars/uportal__{component-name}/dist/{component-min-name}.min.js"></script>
            <{component-name}></{component-name}>
        ]]>
    ```
    ```
        # to find the component min name

        $ cd ~/.m2/repository/org/webjars/npm/uportal__{component-name}/0.1.0-SNAPSHOT/

        # this will give you the right name of the min.js file {component-min-name}

        $ jar tvf uportal__{whatever you found there}-0.1.0-SNAPSHOT.jar | grep min.js

    ```
5. Add the webjar as a dependency:
    In overlays/resource-server/build.gradle, add the follwing runtime dependency:
            runtime "org.webjars.npm:uportal__{component-name}:0.1.0-SNAPSHOT@jar"
