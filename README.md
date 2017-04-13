
Info
====

Being  a JS testing tool, [Angular Protractor](https://github.com/angular/protractor) enforces one to (re)write the web page tests in Javascript - this project (and the sibling [C# Protractor Client Framework](https://github.com/sergueik/powershell_selenium/tree/master/csharp/protractor-net)) tries to make such switch unnecessary.
On the other hand Protractor offers some [locator strategies](https://github.com/angular/protractor/blob/master/lib/clientsidescripts.js) that take advantage of Angular's features to testers - this project tries to keep these available.


Currently supported Angular Proractor methods:
```bash
binding.js
buttonText.js
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
testForAngular.js
waitForAngular.js
```


Building
========
Windows (jdk1.7.0_65, 32 bit)
-----------------------------
The following commands compile the project in console.
```cmd
set M2=c:\java\apache-maven-3.2.1\bin
set M2_HOME=c:\java\apache-maven-3.2.1
set MAVEN_OPTS=-Xms256m -Xmx512m
set JAVA_HOME=c:\java\jdk1.7.0_65
set JAVA_VERSION=1.7.0_65
PATH=%JAVA_HOME%\bin;%PATH%;%M2%
REM
REM move %USERPROFILE%\.M2 %USERPROFILE%\.M2.MOVED
REM rd /s/q %USERPROFILE%\.M2
set TRAVIS=true
mvn clean package
```
Linux
-----
```bash
export TRAVIS=true
mvn clean package
```

Using with existing Java projects
=================================

Maven
-----

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

Ant
---

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

Example Test
============

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


Note
----
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

Selenum Version compatibility
============================

|                      |              |
|----------------------|--------------|
| SELENIUM_VERSION     | __2.53.1__   |
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



Related Projects 
================
  - [Protractor-jvm](https://github.com/F1tZ81/Protractor-jvm)
  - [ngWebDriver](https://github.com/paul-hammant/ngWebDriver)
  - [angular/protractor](https://github.com/angular/protractor) 
  - [bbaia/protractor-net](https://github.com/bbaia/protractor-net)
  - [sergueik/protractor-net](https://github.com/sergueik/powershell_selenium/tree/master/csharp/protractor-net)


Authors
-------
[Serguei Kouzmine](kouzmine_serguei@yahoo.com)

[Carlos Alexandro Becker](caarlos0@gmail.com)
