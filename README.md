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

The basic observation, then, is that when it comes to automating user-like behavior, there are really two levels: the level of user intention, namely what the user sees themselves as doing, and the level of implementation, meaning the precise and specific steps that happen to be necessary for the user to do what they want. In the language of software development, these are two separate concerns. One concern could be summarized as what the user is trying to do, and the other concern could be summarized as how the user is trying to do it. In the code that we have on this screen right now, these two concerns are not separated. The "what" and the "how" are all mixed together. So this is one problem: we need a better separation of concerns to help organize our code and make it understandable, and to make it reflect the larger reality we are trying to model with our test.









