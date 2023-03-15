# The Page Object Model in Theory

*Maintaining a robust and large test suite involves the use of good software design practices. When it comes to UI testing, one of the most tried-and-true patterns is the Page Object Model pattern.*

We're going to learn about one of the most important and standard design patterns for UI testsuites. It's called the Page Object Model, and we'll often also see it abbreviated to POM or "Pom".

Before we talk about what the Page Object Model is, let's first discuss the problems that it aims to solve. Here's a snippet of test code that we wrote recently:

    from appium.webdriver.common.mobileby import MobileBy
    from selenium.webdriver.support.wait import WebDriverWait
    from selenium.webdriver.support import expected_conditions as EC


    def test_echo_box_displays_message_back(driver):
        wait = WebDriverWait(driver, 10)
        wait.until(EC.presence_of_element_located(
            (MobileBy.ACCESSIBILITY_ID, 'Echo Box'))).click()
        wait.until(EC.presence_of_element_located(
            (MobileBy.ACCESSIBILITY_ID, 'messageInput'))).send_keys('Hello')
        driver.find_element(MobileBy.ACCESSIBILITY_ID, 'messageSaveBtn').click()
        saved = driver.find_element(MobileBy.ACCESSIBILITY_ID, 'savedMessage').text
        assert saved == 'Hello'
        driver.back()

        wait.until(EC.presence_of_element_located(
            (MobileBy.ACCESSIBILITY_ID, 'Echo Box'))).click()
        saved = driver.find_element(MobileBy.ACCESSIBILITY_ID, 'savedMessage').text
        assert saved == 'Hello'

This code is basically a straightforward use of the Appium API, with a couple assertions thrown in, tucked inside a test method so that we can run it with Pytest. But there are a number of ways in which this code can be improved. From a big picture perspective, one big drawback of this code is that it's hard to tell by looking at it what user behavior it's trying to encode. I have to read very closely which UI elements are being interacted with, and then infer the user behavior back out of those interactions. This is a bit backwards! When we think as humans about writing and reading tests, we think first about user behavior on a high level. What is the user attempting to do? We use words like "log in" or "add an item to the shopping cart".

What I'm getting at is that users don't really *intend* to use UI elements. They have to use UI elements because that's how app works, but their goal in using your app is not to tap buttons or enter text. Their goal is to accomplish a certain task, and for better or worse, your UI elements are the ones they have to use to do it. It would be nice if we could somehow write our tests at a level that reflected user intent and user experience, because tests written like that would be extremely transparent in the sense of making it very obvious what they are trying to test.

The basic observation, then, is that when it comes to automating user-like behavior, there are really two levels: the level of user intention, namely what the user sees themselves as doing, and the level of implementation, meaning the precise and specific steps that happen to be necessary for the user to do what they want. In the language of software development, these are two separate concerns. One concern could be summarized as **what** the user is trying to do, and the other concern could be summarized as **how** the user is trying to do it. In the code that we have right now, these two concerns are not separated. The "what" and the "how" are all mixed together. So this is one problem: we need a better separation of concerns to help organize our code and make it understandable, and to make it reflect the larger reality we are trying to model with our test.

We actually have another separation of concerns problem here too. This test takes place over two separate views, but nothing in our code indicates this. Another issue with this code is duplication. In two separate places we find and interact with the 'Echo Box' element and the <code>savedMessage</code> element. What if our app developer makes some change to the app such that these elements have different selectors? We would then need to hunt down every place in our testsuite where the current selectors are set, and update them. In our case it's just two instances, but imagine that this were a more complex feature with many different testcases all involving the same elements, potentially even spread over multiple files. It could be a significant pain to find and update all those instances.

There are other kinds of duplication too, like the fact that we have this pattern where we call <code>wait.until</code> and then inside it immediately use <code>EC.presence_of_element_located</code>. It all gets rather verbose and makes it hard to see what we're actually doing. Finally, in some cases it's just not clear why we are doing what we are doing here. Why are we using <code>driver.back()</code> for example? Is it to navigate back a view, or is it to trigger some other behavior that might be associated with the back button? This is the kind of thing we might want a comment in our code to discuss.

But thankfully, there's a way to address all these issues by using the Page Object Model. So let's now talk about what it is!

First off, let's talk about what the words mean. What is a "Page Object?" Well in this case, "page" refers to a webpage, and that's because this pattern was established as part of web-based UI testing, before mobile apps were around. So the UIs that we had to deal with were all located on different "pages". I still use this term "page" even though with mobile apps we're not dealing with pages, because "Page Object Model" is the name of the pattern that you'll hear everywhere.

OK, and what about "Object"? This means making a sort of model of a given page as a software object, in other words a Class for the purposes of Python. So the basic idea here is that we construct a model for each page or view, that will have certain responsibilities. Then we will *use* these objects in our test code, which will have other responsibilities.

What are the responsibilities of the page object?

First, the object is responsible for knowing all about the UI elements for a given page or view. In fact, it's not only responsible for knowing all about these, it's responsible for *preventing* anyone else from knowing about them. This is a pattern in software development known as "encapsulation". The idea is that low-level details about UI elements on a specific page are not important to anyone outside the page, so why give access to them?

Instead the page object is responsible for exposing high-level actions that a user could take on a given page. Which actions should be exposed is of course a judgment call, but let's look at a few examples. On a login page, what are the actions a user could take? It might be tempting to suggest that one action the user could take would be something like "click the login button." It's true that that's a user action, but not on the level that I'm talking about. A user never goes to a login page and thinks, "today, I want to click the login button." No, instead, they think, "I'd like to log in." So logging in is an action that you could take on the login page. Many login pages also give you the ability to request a new password, so that's another example of a user action.

Page objects are also responsible for making data available that might be necessary for the purpose of understanding what is happening on the page. Just like a user could read text or use their eyes to understand what's happening in the app, the page object can create methods or properties that return high-level information about what's happening. Again, the key here is the "high-level" descriptor.

As an example of high-level information, imagine you log into a bank web app and are at your dashboard page. One of the information retrieval methods that the page object for this page might expose would be a method that gets the current account balance. This is something a user would want to do, and it would also be useful for verification during the course of your test!

So, to summarize, the page object model is basically a model where objects expose high-level user actions to the outside world, while encapsulating knowledge about how to implement those actions using specific UI elements, that it doesn't need to share with anyone else.

How does a page object expose high-level user actions or data? As methods or properties of a class, of course! And just like any other class methods, these can take parameters. So we can think of a "login" action as actually a method that takes two parameters: a username and a password. Again, when we think of high-level user actions, we are not considering the "how", we're just considering the "what" in an abstract way. A login action conceptually has two pieces of information required to complete it: the username and password. And so these are the parameters that the login method should take.

So given that the page object knows how to implement high-level user actions and expose those actions as methods, what are the responsibilities of the testcases, then?

The first responsibility is to instantiate and use page objects in order to construct user flows at a high level. Testcases should *never* need to know about UI elements, or access them directly. In fact, testcases should never, or should rarely, need to use the WebDriver session object at all! The responsibility of the test case is to define a user flow using high-level user actions, and certainly users don't care or know about "drivers".

The second responsibility of the testcase is to make assertions on the state of the app to prove that things are working. These could be implicit assertions, as when we move through the various app screens or pages. If an element didn't exist, let's say, the test would fail. So merely moving through a certain flow makes some assertions. Assertions can also be explicit, where we retrieve some information from the view and use the <code>assert</code> keyword to make sure things are as we expect.

What does all of this get us? It helps to implement exactly the kind of separation of concerns we wanted. When using the Page Object Model correctly, your testcase code becomes highly readable and reusable, and your automation code gets encapsulated in the natural groupings of page objects. Duplication is discouraged because different tests just use the same page object actions which are exposed, and because the page objects themselves treat UI element selectors as data, and can just put them at the top of their file.

We'll be learning how to take regular old code and refactor it into a respectable Page Object Model based test framework.


