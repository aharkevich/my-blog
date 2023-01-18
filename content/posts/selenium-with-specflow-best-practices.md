---
title: "Selenium with Specflow best practices .Net"
date: 2023-01-09T23:28:34+01:00
tags: ["Selenium", "Specflow", ".Net", "Best Practices", "Tests Automation"]

showWordCount : true
showHeadingAnchors : true
showTaxonomies : true
showEdit: false 
sharingLinks: false
showLikes: false
---

If you want to cover your application with stable, fast, effective and easy to maintain automated tests, you should follow few simple rules, that will make your life easier.

In this article I will tell you about Selenium and Specflow great features and practices based on my E2E tests development experience.

### Locators
Do **NOT** use XPath

XPath locator with the absolute path to the DOM element will lead you to the errors with the slightest change in the HTML structure.

Using of relative path in XPath locator will solve this problem, however, it will work slower, then other types of locators.

```
driver.findElement(By.xpath("/html/body/div[7]/div[3]) :x:
```

Such locators are also hard to read and maintain.

What locators **should** be used in this case?

The good practice is to use the following locators: 
1. By html element identifier
```
driver.findElement(By.Id("loginForm")) :white_check_mark:
```
2. By CSS class
```
driver.findElement(By.ClassName("myTable")) :white_check_mark:
```
3. By CSS selector
```
driver.findElement(By.CssSelector("[data-qa='object-id']") :white_check_mark:
```
As you may have noticed, I've used `data-qa` attribute selector. Using of such attributes is a great approach. You will not depend on any ids or classes, that can be change during development, you will define your own attributes just for your locators.

Even if you need to find the element by some text, you can avoid the use of XPath with nested locators.
```
drive.FindElements(By.TagName("a")).First(e => e.Text == "Car") :white_check_mark:
```

P.S. Feel free to use remaining locators like `By.TagName` or `By.Name`, but be careful, you can have more than one DOM element with the specified name on the page.

**Conclusion**: try to avoid the use of XPath as long as possible :smile:

### Waits
Do **NOT** use Thread.Sleep

You've made a click on the button and waiting for some element to appear before content verification. How to deal with it?

The simplest solution you can think of, is to use `Thread.Sleep`, but this idea is not as good as you might think.

For example, you are calling `Thread.Sleep` for 10 seconds, but what happens when your element only takes 2-3 seconds to load? Right, you are waiting for remaining 7-8 seconds to continue tests execution. So, `Thread.Sleep` will lead you to increased tests execution time in cases, where elements are loaded earlier than we expect. Also, in general, the use of `Thread.Sleep` is considered to be a bad practice.

What can be used instead of `Thread.Sleep`?

Selenium provides us with powerful **Wait** mechanisms, that make our code more flexible. There are several types of them.

#### Implicit Wait

Unlike `Thread.Sleep`, implicit wait doesn't wait for complete time duration. If an element is found before specified wait duration, it will continue execution of your next line of the code immediately.

Implicit wait is useful, when you need to apply wait mechanism once for all elements specified in your test script.

```
driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);

// Your locators
```

#### Explicit Wait

Explicit wait is waiting for some condition to occur. Unlike implicit wait that is used for all elements in the test script, explicit wait works only on the particular element.

```
var wait = new WebDriverWait(driver, timeout: TimeSpan.FromSeconds(10));

wait.Until(driver => driver.FindElement(By.Id("loginForm")))
```

#### Fluent Wait

As well as maximum timeout time, in the fluent wait you are setting the value of how often the condition should be evaluated.

You can also configure the wait to ignore specific types of exceptions. 
```
var wait = new WebDriverWait(driver, timeout: TimeSpan.FromSeconds(10))
{
    PollingInterval = TimeSpan.FromMilliseconds(2000)
};

wait.IgnoreExceptionTypes(typeof(ElementClickInterceptedException));
wait.Until(driver => driver.FindElement(By.Id("loginButton")))
```
As you can see, all types of waits are similar to each other. If you need to wait for particular DOM element to appear, feel free to use explicit or fluent wait. If you need to wait for all elements on the page, you can use implicit wait.

**Important:** Do not use different types of waits at the same time. It will lead you to unpredictable wait times.

### Hooks

Specflow hooks allow you to run automated scripts at specific time, e.g., after each scenario or before each test step. Specflow hooks, combined with Selenium is a powerful tool that can make your tests more stable and reliable.

In this section I will talk about few most useful hooks.

#### Browser session management

**DO** finish your browser session after each test run.

This simple hook will help you to keep your client storage clean before each test run.

```
[AfterScenario]
internal static void FinishBrowserSession(IWebDriver driver)
{
    driver.Quit();
}
```

Such approach is very useful in case when you need to login under different users. Having this hook frees you from adding logout logic and prevent you from possible flakiness caused by old/not updated client storage values.

#### Test failures management

When you have a lot of E2E tests, it's not uncommon for some test to fail on the one of your environments. Having logs will make it easier to find the issue, but what if you will have few screenshots of your browser right in the moment of the failure? Yeah, these pictures will be your real salvation, especially if your page has lots of dynamically loaded elements.

Selenium provides us with mechanism of making page screenshots. Combination of this mechanism with Specflow hook will make your logging much more useful.

```
[AfterStep]
[BeforeStep]
internal static void AfterStep(IWebDriver driver)
{
    var screenShot = ((ITakesScreenshot)driver).GetScreenshot();
    var path = "Full path to the image";

    screenShot.SaveAsFile(path, ScreenshotImageFormat.Png);

    // Store screenshots references, e.g., in scenario or feature context for further processing
}
```

In the example above, we are making screenshots and store them before and after each step.

With current implementation you will have tons of images that will take up a lot of space on the disk. Let's add one more hook to fix this.

```
[AfterScenario]
internal static void AfterScenario(TestContext testContext)
{
    // If scenario passed, let's remove all the screenshots
    if (testContext.CurrentTestOutcome == UnitTestOutcome.Passed)
    {
        // Retrieve screenshots references from the context
        ...

        // Remove screenshots from the disk
        foreach (var screenShot in screenShots)
        {
            File.Delete(screenShot);
        }
    }

    // If scenario failed, we keep screenshots on the disk
}
```

P.S. You can also use hooks to setup/clean some data before/after each test run.

### Conclusion

Combination of right Selenium and Specflow approaches ensures, that your application works as expected.