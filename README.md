# GTM Guidelines

These guidelines are a collection of things any [Google Tag Manager (GTM)](https://www.google.com/analytics/tag-manager/) user should be aware of and understand before actually implementing tracking using the tool.
While marketed and understood by many as a simple WYSIWYG tracking editor anyone can use, GTM is a powerful tool which gives its users unrestricted access to the entire website and its data and functionality. With great power comes great responsibility and these guidelines are supposed to help navigate the jungle that can be tracking implementation. 

## JavaScript

JavaScript in Google Tag Manager can interfere with your website's JavaScript in unintended and unexpected ways. To make sure that GTM implementations have as little impact on smooth operation of your site, follow these basic guidelines:

### Isolate custom JavaScript

JS in GTM should not leak into the global variable space. Always make sure to limit the scope of your implementations by wrapping your code in an anonymous function

    // good
    (function() {
        var foo = 'bar'
    })()

    // bad
    var foo = 'bar'

If you need to persist variable information for the duration of the pageview or you need to access the same variable from multiple tag executions, use a dataLayer variable.

    // 1st tag
    (function() {
        dataLayer.push({
            foo: 'bar'
        })
    })()

    // 2nd tag
    console.log({{dl.foo}})

### Reuse code with function variables

Instead of a simple string or integer, you can return a function as the result of a GTM variable of the _Custom JavaScript_ type.

    // Custom JavaScript variable: js.double
    function () {
        var doubleSize = function(number) {
            return number * 2
        }
        return doubleSize
    }

Your anonymous outer function returns another function, in this example `doubleSize`.
You can then use {{js.double}} as a function in other JavaScript based GTM tags and variables:

    {{js.double}}(2)
    // Output: 4
	
### Adhere to the browser support guideline of your website

Know which browsers your website / your company officially supports. Unfortunately there are still plenty of JavaScript features that semi-popular browsers do not support. Implementing these anyway without fallbacks or programmatically checking for browser support beforehand can break your tracking or even website functionality itself.

### Do not use Custom JS where built-in solutions suffice

Custom JavaScript is more prone to errors than the solutions that Google Tag Manager supports out of the box. Use the built-in tag templates instead of JavaScript where you can.

* If you're asked to implement some tracking code, check if there's a GTM-native tag template
* Use lookup or regex tables instead of if…else constructs

### Document all functions

If you write your own functions, document them [using JSDoc3 syntax](http://usejsdoc.org/about-getting-started.html).

	/**
	 * Increases the supplied number by one
	 * @param  {number} number The number to increase
	 * @return {number}        The number increased by 1
	 */
	function increaseByOne(number) {
		return number + 1;
	}

### Use JS error reporting

It is difficult to make every piece of code work flawlessly for every user on every device.
You can't test your GTM implementation with every possible setup. To find bugs which happen only under circumstances which you didn't test for, use a JavaScript error reporting tool, e. g. Sentry or New Relic.

It will detect whenever a JavaScript error occurs and save it to a database where you can evaluate the root cause of the issue.

Do not use your tracking tool for JavaScript error detection, they have nothing to do with your business.

### Do not use console.log in production

Do not use `console.log` on production systems. It needlessly clutters both your code and the users' browser console.
Feel free to use it for debugging purposes during development in preview mode, though.

## Security

### Use subresource integrity

If you implement 3rd party scripts, use subresource integrity hashes to make sure the integrated resource does not change without you noticing.
See https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity for more information.

### Ask for a Content Security Policy (CSP)

A Content Security Policy allows you to define which hosts the browser may load scripts, fonts, stylesheets etc. from. See https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP for more details.
This has significant technical impact on your entire website and frontend development so all other developers need to be onboard with this. But it can prevent third party scripts from going rogue and suddenly executing JavaScript you didn't approve.

### Ask a(nother) developer

If you are not _entirely_ sure what you are doing, consult with one or more professional JavaScript developers who are in charge or coding the JavaScript on your website.

## dataLayer

### Do not use tool-specific dataLayer naming and structure

Make sure the names of dataLayer variables are always as descriptive as possible.
Do not use tool specific keys, even if it may seem convenient. Separating tool logic from the website is one of the most important goals of using Google Tag Manager.

    // good
    dataLayer.push({
        loginStatus: true
    })

    // bad
    dataLayer.push({
        dimension17: 1
    })

Use Custom JavaScript variables to preprocess data for individual tools if necessary.

### Do not push personally identifiable information (PII) to dataLayer

Information that can be used by third parties to identify a person or a device should not be used in dataLayer.
That includes email and IP adresses.

Your proprietary user ID can be okay if only you can trace it back to an individual. Nevertheless you should consult with a privacy expert first.

## Access management

Give container publishing access to as few people as possible. Everyone who doesn't _absolutely_ need it should have read access.
Do quarterly checks if users can be moved to a more restrictive acces level or removed entirely.

## Workflow

### Workspaces

Always keep the "Default workspace" empty for fastlane emergencies. Should sudden need for a bugfix, you shouldn't have to
a) sacrifice work that's already done just because you don't have an empty workspace left
b) go through workspaces with existing changes because you need to make sure what they do before publishing your bugfix

Use one workspace per implementation, do not solve different problems in one workspace.

### Use constants for all configurations

As soon as more than one tag uses the same configuration value (e. g. a tracking account ID), create a GTM variable of type "Constant" for it and refer to it from the tag.

### Activate only the auto variables you need

Activate only the ones you actively use. The values of inactive auto variables do not need to be calculated which saves resources.

### Version names and notes

* Name every version. The changes of the version should be intuitively clear from just the name. Focus on the _business impact_ of the change rather than the technical details. If you use a ticket or task system like JIRA, include the ticket ID in the version name.
* Write version notes for every version. The notes should answer
  * Who requested the change?
  * What was the tracking behavior before the release, what's it like now?
  * Were there website changes that this GTM implementation depends on? If so, what are they?

### Folders

Use folders to group your elements by the goal they are achieving.

| Type          | Description                                                                 |
| :------------ | :-------------------------------------------------------------------------- |
| Ad Tech       | Advertising technology, 3rd party traffic partners like Adform or Criteo    |
| Web Analytics | Web analytics software like Google Analytics or Webtrekk                    |
| Utilities     | Helper functions and little scripts that can be used for different purposes |

## Naming

### Tags

### Trigger

`Type` `Description`

| Type       | Description                                                                                                                                   |
| :--------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| Conversion | A user action with immediate positive business impact                                                                                         |
| Pageview   | A pageview. By your definition, so it's not relevant whether it's an actual full pageload or a new screen inside your single page application |
| Event      | Any custom GTM event that does not match one of the types listed above                                                                        |
| State      | Describes a certain state of your website and is usually used as an exception rather than a trigger                                           |

### Variables

`type.[tool.]name`

`type` refers to the different kinds of Google Tag Manager variables available. Since a variable's type is not immediately visible when looking at a reference like `{{myVariable}}`, it's hard to identify where the data is coming from.

| Variable type             | Type value  |
| :------------------------ | :---------- |
| HTTP Referrer             |  ref        |
| URL                       | url         |
| 1st Party Cookie          | cookie      |
| Custom JavaScript         | js          |
| Data Layer Variable       | dl          |
| JavaScript Variable       | global      |
| Auto-Event Variable       | auto        |
| DOM Element               | dom         |
| Element Visibility        | vis         |
| Constant                  | const       |
| Custom Event              | custom      |
| Environment Name          | env         |
| Google Analytics Settings | settings    |
| Lookup Table              | lookup      |
| Random Number             | rnd         |
| RegEx Table               | regexlookup |
| Container ID              | gtm         |
| Container Version Number  | gtm         |
| Debug Mode                | debug       |

`tool` is an optional part for variables which are (and will be) only relevant to one tool, e. g. a specific 3rd-party tool ID as a constant a data reformatting script which reformats data specifically for one tag.
A `tool` in this case can be either a product like `adform`, `hotjar`, `kissmetrics` or a universal concept like Google Analytics Enhanced eCommerce which has the common shorthand `eec`.

`name` is a descriptive name for this variable. Keep it short but in combination with `type` and `tool` it should be immediately understandable what it refers to.

#### Examples for variables

* An Adform publisher ID: `const.adform.id`
* A JavaScript reformatting script which provides product data in a specific format, relevant only to Yahoo: `js.yahoo.productData`
