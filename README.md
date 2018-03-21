### Info

The [__Angular Protractor__](https://github.com/angular/protractor) is a very popular Selenium Web Driver based project.
However being a Javascript testing tool, __Protractor__ enforces one to (re)write the whole test suite in Javascript which some may find not acceptable solution.

This project and the sibling [C# Protractor Client Framework](https://github.com/sergueik/powershell_selenium/tree/master/csharp/protractor-net) tries to make such conversion unnecessary.

On the other hand __Angular 1.x__  a.k.a. __AngularJS__ offered unique features: `ng-repeat` and `ng_model`,
which __Protractor__ takes advantage of by providing unique MVC-style [locator strategies](http://www.protractortest.org/#/api?view=ProtractorBy)

The __jProtractor__ project makes these, plus few extra, locator extensions available to Java:


```javascript
findBindings = function(binding, exactMatch, using, rootSelector)
```
finds a list of elements in the page by their `angular` binding
```javascript
findByButtonText = function(searchText, using)
```
finds buttons by textual content
```javascript
findByCssContainingText = function(cssSelector, searchText, using)
```
finds elements by css selector and textual content
```javascript
findByModel = function(model, using, rootSelector)
```
finds elements by model name
```javascript
findByOptions = function(options, using)
```
finds elements by options
```javascript
findByPartialButtonText = function(searchText, using)
```
finds button(s) by textual content fragment
```javascript
findAllRepeaterRows = function(using, repeater)
```
finds an array of elements matching a row within an `ng-repeat`
```javascript
findRepeaterColumn = function(repeater, exact, binding, using, rootSelector)
```
finds the elements in a column of an `ng-repeat`
```javascript
findRepeaterElement = function(repeater, exact, index, binding, using, rootSelector)
```
finds an element within an `ng-repeat` by its row and column.
```javascript
findSelectedOption = function(model, using)
```
finds the selected option elements by model name (Angular 1.x).
```javascript
findSelectedRepeaterOption = function(repeater, using)
```
finds selected option elements in the select implemented via repeater without a model.
```javascript
TestForAngular = function(attempts)
```
tests whether the angular global variable is present on a page
```javascript
waitForAngular = function(rootSelector, callback)
```
waits until Angular has finished rendering
```javascript
return angular.element(element).scope().$eval(expression)
```
evaluates an Angular expression in the context of a given element.

These are 
implemented in a form of Javascript snippets, one file per method, 
borrowed from __Protractor__  project's [clientsidescripts.js](https://github.com/angular/protractor/blob/master/lib/clientsidescripts.js)
and can be found in the `src/main/java/resources` directory:
```bash
binding.js
buttonText.js
cssContainingText.js
evaluate.js
getLocationAbsUrl.js
model.js
options.js
partialButtonText.js
repeater.js
repeaterColumn.js
repeaterElement.js
repeaterRows.js
resumeAngularBootstrap.js
selectedOption.js
selectedRepeaterOption.js
testForAngular.js
waitForAngular.js
```
Many __AngularJS__-specific locators [aren't any longer supported](https://stackoverflow.com/questions/36201691/protractor-angular-2-failed-unknown-error-angular-is-not-defined) by __Angular 2__.

The newest Protractor [`By.deepCss` (Shadow DOM)](http://www.protractortest.org/#/api?view=ProtractorBy.prototype.deepCss) and [flexible `By.addLocator`](http://www.protractortest.org/#/api?view=ProtractorBy.prototype.addLocator) are not yet supported.

The __E2E_Tests__ section of the [document](https://angular.io/guide/upgrade) covers the migration.

The standard pure Java Selenium locators are also supported.

### Example Tests
There is a big number of examples in  [`src/test/java/com/jprotractor/integration`](https://github.com/sergueik/jProtractor/tree/master/src/test/java/com/jprotractor/integration) directory.

For desktop browser testing, run a Selenium node and Selenium hub on port 4444 and
```java
  @BeforeClass
  public static void setup() throws IOException {
      DesiredCapabilities capabilities =   new DesiredCapabilities("firefox", "", Platform.ANY);
      FirefoxProfile profile = new ProfilesIni().getProfile("default");
      capabilities.setCapability("firefox_profile", profile);
      seleniumDriver = new RemoteWebDriver(new URL("http://127.0.0.1:4444/wd/hub"), capabilities);
      ngDriver = new NgWebDriver(seleniumDriver);
  }

  @Before
  public void beforeEach() {
	  String baseUrl = "http://www.way2automation.com/angularjs-protractor/banking";
	  ngDriver.navigate().to(baseUrl);
  }

  @Test
  public void testCustomerLogin() throws Exception {
	  NgWebElement element = ngDriver.findElement(NgBy.buttonText("Customer Login"));
	  highlight(element, 100);
	  element.click();
	  element = ngDriver.findElement(NgBy.input("custId"));
	  assertThat(element.getAttribute("id"), equalTo("userSelect"));
	  Enumeration<WebElement> elements = Collections.enumeration(ngDriver.findElements(NgBy.repeater("cust in Customers")));

	  while (elements.hasMoreElements()){
      WebElement next_element = elements.nextElement();
      if (next_element.getText().indexOf("Harry Potter") >= 0 ){
      	System.err.println(next_element.getText());
      	next_element.click();
      }
	  }
	  NgWebElement login_element = ngDriver.findElement(NgBy.buttonText("Login"));
	  assertTrue(login_element.isEnabled());	
	  login_element.click();
	  assertThat(ngDriver.findElement(NgBy.binding("user")).getText(),containsString("Harry"));
	
	  NgWebElement account_number_element = ngDriver.findElement(NgBy.binding("accountNo"));
	  assertThat(account_number_element, notNullValue());
	  assertTrue(account_number_element.getText().matches("^\\d+$"));
  }
```
for CI build replace the Setup () with
```java
  @BeforeClass
  public static void setup() throws IOException {
	  seleniumDriver = new PhantomJSDriver();
	  ngDriver = new NgWebDriver(seleniumDriver);
  }
```
### Tests

The project contains over 50 tests execrising various scenarios with popular web sites like and also with static AngularJS sample pages checked in to `src/test/resources`. To prevent running all these tests during package generation, these tests are currenly converted to integration tests. To run these tests use the comand:
```cmd
mvn integration-test
```

### Note
PhantomJs allows loading Angular samples from `file://` content, you need to allow some additional options if the test page loads external content:

```java
  DesiredCapabilities capabilities = new DesiredCapabilities("phantomjs", "", Platform.ANY);
  capabilities.setCapability(PhantomJSDriverService.PHANTOMJS_CLI_ARGS, new String[] {
    "--web-security=false",
    "--ssl-protocol=any",
    "--ignore-ssl-errors=true",
    "--local-to-remote-url-access=true", // prevent local file test XMLHttpRequest Exception 101
    "--webdriver-loglevel=INFO" // set to DEBUG for a really verbose console output
  });
  seleniumDriver = new PhantomJSDriver(capabilities);

  seleniumDriver.manage().window().setSize(new Dimension(width , height ));
  seleniumDriver.manage().timeouts()
    .pageLoadTimeout(50, TimeUnit.SECONDS)
    .implicitlyWait(implicitWait, TimeUnit.SECONDS)
    .setScriptTimeout(10, TimeUnit.SECONDS);
  wait = new WebDriverWait(seleniumDriver, flexibleWait );
  wait.pollingEvery(pollingInterval,TimeUnit.MILLISECONDS);
  actions = new Actions(seleniumDriver);
  ngDriver = new NgWebDriver(seleniumDriver);
  localFile = "local_file.htm";
  URI uri = NgByIntegrationTest.class.getClassLoader().getResource(localFile).toURI();
  ngDriver.navigate().to(uri);
  WebElement element = ngDriver.findElement(NgBy.repeater("item in items"));
  assertThat(element, notNullValue());

```
Certain tests ( e.g. involving `NgBy.selectedOption()` ) currently fail under [travis](https://travis-ci.org/) CI build.

### Selenum Version compatibility

|                      |              |
|----------------------|--------------|
| SELENIUM_VERSION     | __3.x__ ,  __2.53.1__ |
| FIREFOX_VERSION      | __45.0.1__   |
| CHROME_VERSION       | __56.0.X__   |
| CHROMEDRIVER_VERSION | __2.29__     |

|                      |              |
|----------------------|--------------|
| SELENIUM_VERSION     | __3.2.0__    |
| FIREFOX_VERSION      | __52.0__     |
| GECKODRIVER_VERSION  | __0.15__     |
| CHROME_VERSION       | __57.0.X__   |
| CHROMEDRIVER_VERSION | __2.29__     |



### Building the jar

You can build the `jprotractor.jar` from the source by cloning the repository
```bash
git clone https://github.com/sergueik/jProtractor
```
and running the following in console:

#### Windows (jdk1.7.0_65, 32 bit)
```cmd
set M2=c:\java\apache-maven-3.2.1\bin
set M2_HOME=c:\java\apache-maven-3.2.1
set MAVEN_OPTS=-Xms256m -Xmx512m
set JAVA_VERSION=1.8.0_112
set JAVA_HOME=c:\java\jdk%JAVA_VERSION%
PATH=%JAVA_HOME%\bin;%PATH%;%M2%
REM
REM move %USERPROFILE%\.M2 %USERPROFILE%\.M2.MOVED
REM rd /s/q %USERPROFILE%\.M2
set TRAVIS=true
mvn -Dmaven.test.skip=true clean package
```
#### Linux
```bash
export TRAVIS=true
mvn -Dmaven.test.skip=true clean package
```

The project contains substantial number of 'integration tests' and by default maven will run all, which will take quite some time,
also some of the tests could fail in your environment.
After a test failure, maven will not package the jar.

Alternatively you can temporarily remove the `src/test/java` directory from the project:
```bash
rm -f -r src/test/java
mvn clean package
```

The jar will be in the `target` folder:
```cmd
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ jprotractor ---
[INFO] Building jar: C:\developer\sergueik\jProtractor\target\jprotractor-1.2-SNAPSHOT.jar
```
You can install it into your local `.m2` repository as  explained below

### Keyword-Driven Frameworks

For example of [Keyword-Driven Framework](http://toolsqa.com/selenium-webdriver/keyword-driven-framework/introduction/), incorporating the Selenium Protractor methods see [sergueik\keyword_driven_framework](https://github.com/sergueik/keyword_driven_framework). It contains both __jProtractor__ and a similar Java-based Protractor Client Library wrapper ptoject, [paul-hammant/ngWebDriver](https://github.com/paul-hammant/ngWebDriver).

### Using with existing Java projects

#### Maven
  * Copy `target\jprotractor-1.2-SNAPSHOT.jar` to your project `src/main/resources`:

```bash
+---src
    +---main
            +---java
            |   +---com
            |       +---mycompany
            |           +---app
            +---resources

```
  * Add reference to the project `pom.xml` (a sample project is checked in)
```xml
<properties>
  <jprotractor.version>1.2-SNAPSHOT</jprotractor.version>
</properties>
```
```xml
<dependencies>
<dependency>
     <groupId>com.jprotractor</groupId>
     <artifactId>jprotractor</artifactId>
     <version>${jprotractor.version}</version>
     <scope>system</scope>
     <systemPath>${project.basedir}/src/main/resources/jprotractor-${jprotractor.version}.jar</systemPath>
</dependency>
</dependencies>
```
  * Add reference to the code:
```java
import com.jprotractor.NgBy;
import com.jprotractor.NgWebDriver;
import com.jprotractor.NgWebElement;
```

#### Ant

* Copy the `target\jprotractor-1.2-SNAPSHOT.jar`  in the same location oher dependency jars, e.g. `c:\java\selenium`,
* Use the bolierplate `build.xml` (a sample project is checked in) or merge with your existing build file(s):
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<project name="example" basedir=".">
  <property name="build.dir" value="${basedir}/build"/>
  <property name="selenium.jars" value="c:/java/selenium"/>
  <property name="src.dir" value="${basedir}/src"/>
  <target name="loadTestNG" depends="setClassPath">
    <taskdef resource="testngtasks" classpath="${test.classpath}"/>
  </target>
  <target name="setClassPath">
    <path id="classpath_jars">
      <pathelement path="${basedir}/"/>
      <fileset dir="${selenium.jars}" includes="*.jar"/>
    </path>
    <pathconvert pathsep=";" property="test.classpath" refid="classpath_jars"/>
  </target>
  <target name="clean">
    <delete dir="${build.dir}"/>
  </target>
  <target name="compile" depends="clean,setClassPath,loadTestNG">
    <mkdir dir="${build.dir}"/>
    <javac destdir="${build.dir}" srcdir="${src.dir}">
      <classpath refid="classpath_jars"/>
    </javac>
  </target>
  <target name="test" depends="compile">
    <testng classpath="${test.classpath};${build.dir}">
      <xmlfileset dir="${basedir}" includes="testng.xml"/>
    </testng>
  </target>
</project>
```
* Add reference to the code:
```java
import com.jprotractor.NgBy;
import com.jprotractor.NgWebDriver;
import com.jprotractor.NgWebElement;
```

### Related Projects
  - [Protractor-jvm](https://github.com/F1tZ81/Protractor-jvm)
  - [ngWebDriver](https://github.com/paul-hammant/ngWebDriver)
  - [angular/protractor](https://github.com/angular/protractor)
  - [bbaia/protractor-net](https://github.com/bbaia/protractor-net)
  - [sergueik/protractor-net](https://github.com/sergueik/powershell_selenium/tree/master/csharp/protractor-net)

### Authors
[Carlos Alexandro Becker](caarlos0@gmail.com)
[Serguei Kouzmine](kouzmine_serguei@yahoo.com)
