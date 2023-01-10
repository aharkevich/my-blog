---
title: "Selenium best practices with .Net"
date: 2023-01-09T23:28:34+01:00
draft: true
tags: ["Selenium", ".Net", "Best Practices", "Tests Automation"]

showWordCount : true
showHeadingAnchors : true
showTaxonomies : true
showEdit: false 
sharingLinks: false
---

If you want to cover your application with stable, fast, effective and easy to maintain automated Selenium tests, you should follow few simple rules that will make your life easier.

### Locators
Do **NOT** use XPath.

XPath locator with the absolute path to the DOM element will lead you to the errors with the slightest change in the HTML structure.

Using of relative path in XPath locator will solve this problem, however, it will work slower, then other types of locators.

```
driver.findElement(By.xpath("/html/body/div[7]/div[3]) :x:
```

As you can see, this locator is really hard to read and maintain in the future.

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

Even if you need to find the element by some text, you can avoid the use of XPath using nested locators.
```
drive.FindElements(By.TagName("a")).First(e => e.Text == "Car") :white_check_mark:
```

P.S. Feel free to use remaining locators like `By.TagName` or `By.Name`, but be careful, you can have more than one element with the specified name on the page.

**Conclusion**: try to avoid the use of XPath as long as possible :smile:

### Waits

To be continued....