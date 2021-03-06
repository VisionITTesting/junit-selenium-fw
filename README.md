# junit-selenium-fw

Junit Based Automation Framework for Beginners

#### 1. Branch: ```first-test-case-without-pom```
> Learning Objective:
>> * Basic Maven Project <br>
>> * POM File <br>
>> * Basic Selenium Test

* Create a Maven Project
* Add below dependency in the POM file:
```xml
 <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>3.141.59</version>
        </dependency>
    </dependencies>
```
* Create package under ```test.java``` with name as ```testcases```
* Create a Test Class: ```test.java.testcases.RunTest```
* Add test case to invoke the browser and navigate to the amazon.in web site
* Note that ```@Test``` annotation is used to invoke the test cases. This annotation is provided to us by Junit library which we have added.
* Also Note annotation ```@Before``` ```@After```. 
* All the annotation which has ```@Before``` annotated method will execute before every test.
* All the annotation which has ```@After``` annotated method will execute after every test.

```java
import org.junit.After;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import java.util.concurrent.TimeUnit;

public class TC_ClassFile1 {

    WebDriver driver;
    String base_url = "https://amazon.in/";
    int implicit_wait_timeout_in_sec = 20;

    //This method will execute before every Test method
    //Since, we need to invoke Browser for every test case, we will use this annotation to have driver init steps
    @Before 
    public void set_up(){
        driver = new ChromeDriver();
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(implicit_wait_timeout_in_sec, TimeUnit.SECONDS);
    }

    @Test
    public void t_01_check_website_is_working(){
        driver.get(base_url);
        String expected = "amazon";
        String actual =driver.getTitle();
        Assert.assertEquals("Page Title validation",expected,actual);
    }
    
    //This method will execute after the end of each @Test annotated method.
    @After
    public void clean_up(){
        driver.quit();
    }
}
```

#### 2. Branch: ```2-test-base-second-test-case```
> Learning Objective:
>> * Inheritance: Test Base Class <br>
>> * Explicit Wait

* In this next enhancement we are making using of inheritance concept i.e. use of ```TestBase``` Class
* All the reusable component can be shifted to this class and all the TestCase Class will inherit this ```TestBase``` class.
* So you can make use of ```TestBase``` Class whenever you wish to create a new Test Class.
* In this case we have two test class and both inheriting the ```TestBase``` class.

```java
//Test Base class
package base;

import org.junit.After;
import org.openqa.selenium.WebDriver;

public class TestBase {
    protected WebDriver driver; //this should never be static, if made static parallel exec of classes not possible
    protected final static String base_url = "https://amazon.in";
    protected final static int implicit_wait_timeout_in_sec = 20;
}
```

* Please notice ```extends``` keyword in Test Classes
* This is how we can make use of methods and variables in the Parent Class which in this case is ```TestBase``` class.
* Driver is declared in the Base Class and can be used in Child Classes. 
* This brings reusability and maintainability of code.
```java
public class TestCases_1 extends TestBase {

    @Before
    public void set_up(){
        driver = new ChromeDriver();
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(implicit_wait_timeout_in_sec, TimeUnit.SECONDS);
    }
```

```java
public class TestCases_2 extends TestBase {

    @Before
    public void set_up(){
        driver = new ChromeDriver();
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(implicit_wait_timeout_in_sec, TimeUnit.SECONDS);
    }
```
* Add another test case for search.
* Notice the Sync i.e. web driver wait ```WebDriverWait webDriverWait = new WebDriverWait(driver,20);```
* After a product is searched, we need to instruct the Selenium to wait for specific condition before proceeding to the next step.
* This is called as "Explicit Wait" i.e. we are explicitly instructing the java script to wait for Element to be clickable.

```java
    @Test
    public void t_02_search_for_product() {
        driver.get(base_url);
        String expected = "Online Shopping site in India: Shop Online for Mobiles, Books, Watches, Shoes and More - Amazon.in";
        String actual =driver.getTitle();
        Assert.assertEquals("Page Title validation",expected,actual);

        //Wait and Search for product
        WebDriverWait webDriverWait = new WebDriverWait(driver,20);
        WebElement elementSearchBox = webDriverWait.until(ExpectedConditions.elementToBeClickable(By.id("twotabsearchtextbox")));

        elementSearchBox.sendKeys("Laptop");
        driver.findElement(By.xpath("//input[@value='Go']")).click();

        //Wait for title
        WebDriverWait webDriverWait1 = new WebDriverWait(driver,20);
        webDriverWait1.until(ExpectedConditions.titleIs("Amazon.in : Laptop"));

        //Assertion for Page Title
        Assert.assertEquals("Page Title validation","Amazon.in : Laptop", driver.getTitle());
    }
```

#### 3. Branch: ```3-move-setup-cleanup-in-base```
> Learning Objective:
>> * Inheritance: Move @Before, @After Annotation to Base Class

* Move @Before and @After in the TestBase Class since it was getting repeated in both the test classes.
* Notice that Set up and clean up methods are not repeated in classes and is being reused from the ```TestBase``` class.
```java
public class TestBase {
    protected WebDriver driver; //this should never be static, if made static parallel exec of classes not possible
    protected final static String base_url = "https://amazon.in";
    protected final static int implicit_wait_timeout_in_sec = 20;

    @Before
    public void set_up(){
        driver = new ChromeDriver();
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(implicit_wait_timeout_in_sec, TimeUnit.SECONDS);
    }

    @After
    public void clean_up(){
        driver.quit();
    }

}
```

#### 4. Branch: ```4-impl-webdriver-factory```
> Learning Objective:
>> * Web Driver factory implementation

* Web Driver factory is implemented to handle Web Driver object initialization.
* Factory method is used a centralized way of creating driver object.
* It comes handy to manage cross browser automation.
* ```WebDriverFactory``` class is added under ```core``` pacakage.
* ```getWebDriverForBrowser``` can be used to get the browser. It accepts argument as "Browse". Pass chrome or mozilla to get the required driver object.

```java
public class WebDriverFactory {

    private static WebDriver driver=null;
    public static WebDriver getWebDriverForBrowser(String browser) throws Exception {
        switch(browser.toLowerCase()){
            case "chrome":
                driver = new ChromeDriver();
                break;
            case "firefox":
                driver = new FirefoxDriver();
                break;
            default:
                throw new Exception("No such browser is implemented.Browser name sent: " + browser);
        }

        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(20, TimeUnit.SECONDS);
        return driver;
    }
}

```

* Use WebDriver Factory to get the Driver object instead of ```driver = new ChromeDriver()```

```java
public class TestBase {
    protected WebDriver driver; //this should never be static, if made static parallel exec of classes not possible
    protected final static String base_url = "https://amazon.in";

    @Before
    public void set_up(){
        try{
            //Use WebDriver Factory to get the Driver object instead of driver = new ChromeDriver()
            driver = WebDriverFactory.getWebDriverForBrowser("chrome");
        }catch(Exception e){
            e.printStackTrace();
            Assert.fail("Incorrect Browser Sent. Check the Stack Trace");
        }
    }

    @After
    public void clean_up(){
        driver.quit();
    }
}
```
#### 5. Branch: ```5-send-browser-info-from-mvn-cmd-line```
> Learning Objective:
>> * Send Browser information from maven command line <br>
>> * ```mvn clean test -Dbrowser=chrome``` <br>
>> * ```mvn clean test -Dbrowser=firefox ``` 

* Same test case should be capable of executing in multiple browsers.
* But to run the test case on different browser, code should not be modified in any way.
* Browser setting should be configurable from outside of the Framework. 
* One of the way to pass browser argument is via command line argument.
* In below code, method ```getBrowserName``` is created to pick or sense the command line argument.
* If there is any argument being sent from command line, for example, ```mvn clean test -Dbrowser=chrome```, it will be captured in the java environment variable.
* Enviroment Variable ```browser``` will store the value as ```chrome``` and can be used with-in the code.
* This statement, ```String browserSentFromCmd = System.getProperty("browser");``` stores the "chrome" sent from cmd line and saves it in the variable ```browserSentFromCmd```
```java
public class TestBase {
    protected WebDriver driver; //this should never be static, if made static parallel exec of classes not possible
    protected final static String base_url = "https://amazon.in";

    private String getBrowserName(){
        String browserDefault = "chrome"; //Set by default
        String browserSentFromCmd = System.getProperty("browser");
        //mvn clean install -Dbrowser=safari
        //browserSentFromCmd = safari
        if (browserSentFromCmd==null){
            return browserDefault;
        }else{
            return browserSentFromCmd;
        }
    }

    @Before
    public void set_up(){
        String browser = getBrowserName();
        try{
            driver = WebDriverFactory.getWebDriverForBrowser(browser);
        }catch(Exception e){
            e.printStackTrace();
            Assert.fail("Browser Initialization failed. Check the Stack Trace. " + e.getMessage());
        }
    }

    @After
    public void clean_up(){
        driver.quit();
    }
}
```

