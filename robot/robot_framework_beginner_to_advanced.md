# Robot Framework - Complete Beginner to Advanced Guide

## Table of Contents
1. [Installation & Setup](#installation--setup)
2. [Basic Structure](#basic-structure)
3. [Text Fields - Complete Example](#text-fields---complete-example)
4. [Dropdowns - Complete Example](#dropdowns---complete-example)
5. [Radio Buttons - Complete Example](#radio-buttons---complete-example)
6. [Checkboxes - Complete Example](#checkboxes---complete-example)
7. [Mouse Actions - Complete Example](#mouse-actions---complete-example)
8. [Waits and Timing - Complete Example](#waits-and-timing---complete-example)
9. [Alerts and Popups - Complete Example](#alerts-and-popups---complete-example)
10. [Frames - Complete Example](#frames---complete-example)
11. [Complete Real Project Example](#complete-real-project-example)

---

## Installation & Setup

```bash
# Install Python (if not already installed)
# Download from https://www.python.org/downloads/

# Install Robot Framework
pip install robotframework

# Install Selenium Library for web testing
pip install robotframework-seleniumlibrary

# Install browser drivers
# For Chrome: Download ChromeDriver from https://chromedriver.chromium.org/
# Or use webdriver-manager
pip install webdrivermanager
webdrivermanager firefox chrome

# Verify installation
robot --version
```

---

## Basic Structure

```robot
*** Settings ***
# This section imports libraries and sets up initial configurations
# Library = External code that provides keywords/functions we can use
Library           SeleniumLibrary

# Suite Setup = Runs ONCE before all tests in this file
# Suite Teardown = Runs ONCE after all tests in this file
Suite Setup       Open Browser    https://www.example.com    chrome
Suite Teardown    Close Browser

# Test Setup = Runs before EACH test case
# Test Teardown = Runs after EACH test case
Test Setup        Log    Starting test
Test Teardown     Log    Test completed

*** Variables ***
# This section defines variables that can be reused
# ${variable_name} = Scalar variable (single value)
# @{variable_name} = List variable (multiple values)
# &{variable_name} = Dictionary variable (key-value pairs)

${URL}            https://www.example.com
${BROWSER}        chrome
${USERNAME}       testuser@example.com

*** Test Cases ***
# This section contains your actual test cases
# Each test case is a sequence of keywords (actions)

My First Test
    [Documentation]    This describes what the test does
    [Tags]    smoke    regression    # Tags help organize and filter tests
    Log    Hello from Robot Framework    # Log prints to console
    Log    This is my first test
    
My Second Test
    Log    Running second test
    Should Be Equal    Hello    Hello    # Assertion - checks if two values are equal

*** Keywords ***
# This section defines custom keywords (reusable functions)
# You create your own keywords to make tests more readable

My Custom Keyword
    [Documentation]    This is a reusable function
    Log    This is a custom keyword
    Log    You can call this from any test case
```

### How to Run Tests
```bash
# Run all tests in a file
robot test_file.robot

# Run tests with specific tag
robot --include smoke test_file.robot

# Run and generate report
robot --outputdir results test_file.robot

# The output will be in:
# - log.html (detailed execution log)
# - report.html (test results summary)
# - output.xml (machine-readable results)
```

---

## Text Fields - Complete Example

```robot
*** Settings ***
Library           SeleniumLibrary

*** Variables ***
# Using a free test automation practice website
${URL}            https://testautomationpractice.blogspot.com/
${BROWSER}        chrome

*** Test Cases ***
Complete Text Field Example
    [Documentation]    This test demonstrates all text field operations
    [Tags]    textfields    beginner
    
    # STEP 1: Open browser and navigate to website
    Open Browser    ${URL}    ${BROWSER}
    # Open Browser = Opens a new browser window
    # ${URL} = The website to visit
    # ${BROWSER} = Which browser to use (chrome, firefox, edge)
    
    # STEP 2: Maximize browser window so all elements are visible
    Maximize Browser Window
    # Maximizes the browser to full screen
    
    # STEP 3: Wait for page to load completely
    Sleep    2s
    # Sleep = Pause execution for specified time
    # 2s = 2 seconds
    
    # STEP 4: Input text into name field
    Input Text    id=name    John Doe
    # Input Text = Types text into an input field
    # id=name = Locator (how to find the element on page)
    # John Doe = The text to type
    
    # STEP 5: Input text into email field
    Input Text    id=email    john.doe@example.com
    # Using ID locator again - most reliable way to find elements
    
    # STEP 6: Input text into phone field
    Input Text    id=phone    +1-555-0123
    
    # STEP 7: Input into textarea (multi-line text)
    Input Text    id=textarea    This is line 1\nThis is line 2\nThis is line 3
    # \n = New line character (creates line break)
    
    # STEP 8: Clear a field and enter new text
    Clear Element Text    id=name
    # Clear Element Text = Removes all text from input field
    
    Input Text    id=name    Jane Smith
    # Now enters the new name
    
    # STEP 9: Get the value from an input field
    ${name_value}=    Get Value    id=name
    # Get Value = Retrieves the current text from input field
    # ${name_value}= stores the result in a variable
    
    Log    The name field contains: ${name_value}
    # Logs the value to console/report
    
    # STEP 10: Verify the value is correct
    Should Be Equal    ${name_value}    Jane Smith
    # Should Be Equal = Assertion that checks if two values match
    # If they don't match, test fails
    
    # STEP 11: Press keyboard keys
    Press Keys    id=name    CTRL+A
    # Press Keys = Simulates keyboard key presses
    # CTRL+A = Select all text
    
    Press Keys    id=name    DELETE
    # Deletes selected text
    
    Input Text    id=name    Final Name
    
    # STEP 12: Take a screenshot for evidence
    Capture Page Screenshot    text_fields_test.png
    # Creates a screenshot file in the output directory
    
    # STEP 13: Close the browser
    Close Browser
    # Always close browser to free up resources

*** Keywords ***
# Custom keyword for reusable text input with logging
Input Text With Logging
    [Arguments]    ${locator}    ${text}
    # [Arguments] = This keyword accepts parameters
    # ${locator} = Where to type
    # ${text} = What to type
    
    Log    Entering text "${text}" into field: ${locator}
    Input Text    ${locator}    ${text}
    Log    Successfully entered text
```

### Different Locator Strategies Explained
```robot
*** Test Cases ***
Understanding Locators
    [Documentation]    Shows different ways to find elements on a webpage
    
    Open Browser    https://testautomationpractice.blogspot.com/    chrome
    Maximize Browser Window
    
    # METHOD 1: Using ID (Most Reliable)
    # Elements have id="name" in HTML
    Input Text    id=name    Test User
    # Format: id=VALUE
    
    # METHOD 2: Using NAME attribute
    # Elements have name="username" in HTML
    Input Text    name=username    TestUser123
    # Format: name=VALUE
    
    # METHOD 3: Using CSS Selector (Powerful and Fast)
    Input Text    css=#name    Test User
    # Format: css=SELECTOR
    # #name means element with id="name"
    # .classname means element with class="classname"
    
    # METHOD 4: Using XPath (Most Flexible)
    Input Text    xpath=//input[@id='name']    Test User
    # Format: xpath=XPATH_EXPRESSION
    # //input = Any input element
    # [@id='name'] = That has id attribute equal to 'name'
    
    # METHOD 5: Using Partial Text
    Click Link    partial link=Sign Up
    # Clicks link containing "Sign Up" in text
    
    # METHOD 6: Using Full Text
    Click Link    link=Click here to Sign Up
    # Clicks link with exact text
    
    Close Browser
```

---

## Dropdowns - Complete Example

```robot
*** Settings ***
Library           SeleniumLibrary

*** Variables ***
${URL}            https://testautomationpractice.blogspot.com/
${BROWSER}        chrome

*** Test Cases ***
Complete Dropdown Example
    [Documentation]    Comprehensive guide to handling dropdowns
    [Tags]    dropdown    select    beginner
    
    # STEP 1: Setup browser
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # STEP 2: Scroll to dropdown (in case it's below the fold)
    Scroll Element Into View    id=country
    # Scroll Element Into View = Scrolls page so element is visible
    # Useful when elements are at bottom of page
    
    # DROPDOWNS/SELECT ELEMENTS - Three Ways to Select
    
    # METHOD 1: Select by Visible Label (Most Common)
    Select From List By Label    id=country    United States
    # Select From List By Label = Selects option by the text you see
    # id=country = The dropdown element
    # United States = The visible text in dropdown
    # This is like clicking dropdown and clicking "United States"
    
    Log    Selected United States from country dropdown
    Sleep    1s    # Pause to see the selection
    
    # METHOD 2: Select by Value Attribute
    Select From List By Value    id=country    usa
    # Select From List By Value = Selects by the HTML value attribute
    # In HTML: <option value="usa">United States</option>
    # We select using "usa" not "United States"
    
    Log    Selected using value attribute
    Sleep    1s
    
    # METHOD 3: Select by Index (Position)
    Select From List By Index    id=country    0
    # Select From List By Index = Selects by position in list
    # 0 = First option, 1 = Second option, etc.
    # Index counting starts from 0
    
    Log    Selected first option using index
    Sleep    1s
    
    # GETTING SELECTED VALUE - Three Ways
    
    # Get the visible text of selected option
    ${selected_label}=    Get Selected List Label    id=country
    # Returns the text you see in dropdown (e.g., "United States")
    
    Log    Currently selected label: ${selected_label}
    
    # Get the value attribute of selected option
    ${selected_value}=    Get Selected List Value    id=country
    # Returns the value attribute (e.g., "usa")
    
    Log    Currently selected value: ${selected_value}
    
    # Get all selected labels (for multi-select dropdowns)
    @{selected_labels}=    Get Selected List Labels    id=country
    # Returns a list of all selected options
    # For single-select, list has one item
    # For multi-select, list can have multiple items
    
    Log Many    Selected labels:    @{selected_labels}
    
    # VERIFICATION - Check if correct option is selected
    Select From List By Label    id=country    India
    ${current_selection}=    Get Selected List Label    id=country
    Should Be Equal    ${current_selection}    India
    # Verifies that India is actually selected
    
    Log    Verification passed: India is selected
    
    # GET ALL OPTIONS from dropdown
    ${options_count}=    Get Element Count    css=#country option
    # Counts how many options are in dropdown
    
    Log    Total options in country dropdown: ${options_count}
    
    # List all available options
    @{all_options}=    Get List Items    id=country
    # Gets all option texts from dropdown as a list
    
    Log Many    Available countries:    @{all_options}
    
    # Take screenshot
    Capture Page Screenshot    dropdown_example.png
    
    Close Browser

*** Test Cases ***
Multi-Select Dropdown Example
    [Documentation]    Working with dropdowns that allow multiple selections
    [Tags]    dropdown    multiselect
    
    # Note: testautomationpractice may not have multi-select
    # This is示例代码 showing how it works
    
    Open Browser    https://www.w3schools.com/tags/tryit.asp?filename=tryhtml_select_multiple    chrome
    Maximize Browser Window
    Sleep    3s
    
    # Switch to the iframe where the example is
    Select Frame    id=iframeResult
    # Many practice sites use iframes (embedded pages)
    
    # Select multiple options at once
    Select From List By Label    id=cars    Volvo    Saab    Audi
    # For multi-select dropdowns, you can pass multiple values
    # Hold Ctrl (Windows) or Cmd (Mac) in manual testing
    
    # Get all selected items
    @{selected_items}=    Get Selected List Labels    id=cars
    Log Many    Selected cars:    @{selected_items}
    
    # Verify multiple selections
    ${count}=    Get Length    ${selected_items}
    Should Be Equal As Numbers    ${count}    3
    # Checks that exactly 3 items are selected
    
    # Unselect specific option
    Unselect From List By Label    id=cars    Saab
    # Removes Saab from selection, keeps others selected
    
    @{remaining_items}=    Get Selected List Labels    id=cars
    Log Many    Remaining selections:    @{remaining_items}
    
    # Unselect by value
    Unselect From List By Value    id=cars    volvo
    
    # Unselect by index
    Unselect From List By Index    id=cars    0
    
    Close Browser

*** Keywords ***
Select Dropdown And Verify
    [Documentation]    Custom keyword to select and verify dropdown
    [Arguments]    ${locator}    ${label}
    # Takes two parameters: dropdown locator and text to select
    
    Log    Selecting "${label}" from dropdown: ${locator}
    
    # Add wait to ensure dropdown is ready
    Wait Until Element Is Visible    ${locator}    timeout=10s
    # Waits up to 10 seconds for dropdown to appear
    
    Wait Until Element Is Enabled    ${locator}    timeout=10s
    # Waits up to 10 seconds for dropdown to be clickable
    
    # Select the option
    Select From List By Label    ${locator}    ${label}
    
    # Verify it was selected
    ${selected}=    Get Selected List Label    ${locator}
    Should Be Equal    ${selected}    ${label}
    
    Log    Successfully selected and verified: ${label}
    
    RETURN    ${selected}
    # Returns the selected value back to caller
```

### Dropdown HTML Structure Explained
```html
<!-- This is how dropdowns look in HTML -->
<select id="country" name="country">
    <option value="">-- Select Country --</option>
    <option value="usa">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="india">India</option>
    <option value="canada">Canada</option>
</select>

<!-- 
Explanation:
- <select> tag creates the dropdown
- id="country" is used in locator: id=country
- Each <option> is a choice in dropdown
- value="usa" is the VALUE attribute (used in Select From List By Value)
- "United States" is the LABEL (used in Select From List By Label)
- Index 0 = first option, Index 1 = "United States", etc.
-->
```

---

## Radio Buttons - Complete Example

```robot
*** Settings ***
Library           SeleniumLibrary

*** Variables ***
${URL}            https://testautomationpractice.blogspot.com/
${BROWSER}        chrome

*** Test Cases ***
Complete Radio Button Example
    [Documentation]    Complete guide to handling radio buttons
    [Tags]    radiobutton    beginner
    
    # STEP 1: Open browser
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # STEP 2: Scroll to radio buttons section
    Scroll Element Into View    id=male
    
    # UNDERSTANDING RADIO BUTTONS
    # - Radio buttons come in groups (same 'name' attribute)
    # - Only ONE can be selected at a time in a group
    # - Selecting one automatically unselects others in same group
    
    # SELECTING RADIO BUTTONS
    
    # Select Male radio button
    Select Radio Button    gender    male
    # Select Radio Button = Selects a radio button in a group
    # gender = The name of the radio button group
    # male = The value attribute of specific radio button to select
    
    Log    Selected Male radio button
    Sleep    1s
    
    # Select Female radio button (Male will be automatically unselected)
    Select Radio Button    gender    female
    # Selecting Female unselects Male automatically
    
    Log    Selected Female radio button (Male is now unselected)
    Sleep    1s
    
    # VERIFICATION - Check which radio button is selected
    
    Radio Button Should Be Set To    gender    female
    # Radio Button Should Be Set To = Verifies selection
    # gender = Group name
    # female = Expected selected value
    # Test fails if different value is selected
    
    Log    Verification passed: Female is selected
    
    # Select Male again
    Select Radio Button    gender    male
    Radio Button Should Be Set To    gender    male
    
    Log    Changed to Male successfully
    
    # CONDITIONAL SELECTION - Select only if not already selected
    
    ${status}=    Run Keyword And Return Status    
    ...           Radio Button Should Be Set To    gender    male
    # Run Keyword And Return Status = Returns True/False
    # Doesn't fail test, just returns result
    
    Log    Is Male selected? ${status}
    
    Run Keyword If    ${status}==False    Select Radio Button    gender    male
    # Run Keyword If = Conditional execution
    # Only selects if not already selected
    
    # GETTING THE SELECTED VALUE
    
    ${selected_value}=    Get Value    xpath=//input[@name='gender'][@checked='checked']
    # Gets value of the currently checked radio button
    # XPath finds the checked radio in gender group
    
    Log    Currently selected gender: ${selected_value}
    
    # Screenshot
    Capture Page Screenshot    radio_button_example.png
    
    Close Browser

*** Test Cases ***
Multiple Radio Button Groups Example
    [Documentation]    Handling multiple radio button groups on same page
    [Tags]    radiobutton    intermediate
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # Different radio button groups are independent
    # You can select one from each group
    
    # GROUP 1: Gender (name='gender')
    Select Radio Button    gender    male
    Radio Button Should Be Set To    gender    male
    Log    Selected: Male
    
    # GROUP 2: Let's say there's an age group (name='agegroup')
    # This is示例代码 - adjust based on actual page elements
    # Select Radio Button    agegroup    adult
    # Radio Button Should Be Set To    agegroup    adult
    # Log    Selected: Adult
    
    # Both selections remain independent
    # Selecting from one group doesn't affect other groups
    
    Close Browser

*** Test Cases ***
Click Radio Button Using Click Element
    [Documentation]    Alternative method - clicking radio button directly
    [Tags]    radiobutton    alternative
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # ALTERNATIVE METHOD: Direct click instead of Select Radio Button
    
    # Click using ID
    Click Element    id=male
    # Click Element = Clicks any element
    # Works for radio buttons by clicking them directly
    
    Sleep    1s
    
    # Click using XPath
    Click Element    xpath=//input[@id='female']
    
    Sleep    1s
    
    # Verify using element attribute
    ${is_checked}=    Get Element Attribute    id=female    checked
    # Gets the 'checked' attribute value
    # Returns 'true' if checked, 'None' if not checked
    
    Log    Female radio button checked status: ${is_checked}
    
    Should Be Equal As Strings    ${is_checked}    true
    # Verifies the attribute is 'true'
    
    Close Browser

*** Keywords ***
Select And Verify Radio Button
    [Documentation]    Reusable keyword for radio button selection
    [Arguments]    ${group_name}    ${value}    ${description}
    # ${group_name} = name attribute of radio group
    # ${value} = value to select
    # ${description} = human-readable description for logging
    
    Log    Selecting ${description} (${value}) from ${group_name} group
    
    # Select the radio button
    Select Radio Button    ${group_name}    ${value}
    
    # Verify selection
    Radio Button Should Be Set To    ${group_name}    ${value}
    
    Log    Successfully selected and verified: ${description}
    
    # Return the selected value
    RETURN    ${value}
```

### Radio Button HTML Structure Explained
```html
<!-- This is how radio buttons look in HTML -->
<input type="radio" id="male" name="gender" value="male"> Male
<input type="radio" id="female" name="gender" value="female"> Female
<input type="radio" id="other" name="gender" value="other"> Other

<!-- 
Explanation:
- type="radio" makes it a radio button
- name="gender" groups them together (all have same name)
- value="male" is used in Select Radio Button command
- id="male" can be used with Click Element command
- Only one can be selected at a time within same name group
-->

<!-- Another independent group -->
<input type="radio" id="student" name="status" value="student"> Student
<input type="radio" id="employed" name="status" value="employed"> Employed

<!-- 
These are independent:
- Different name="status" means different group
- Selecting from 'status' doesn't affect 'gender' selection
-->
```

---

## Checkboxes - Complete Example

```robot
*** Settings ***
Library           SeleniumLibrary

*** Variables ***
${URL}            https://testautomationpractice.blogspot.com/
${BROWSER}        chrome

*** Test Cases ***
Complete Checkbox Example
    [Documentation]    Complete guide to handling checkboxes
    [Tags]    checkbox    beginner
    
    # STEP 1: Open browser
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # STEP 2: Scroll to checkboxes
    Scroll Element Into View    id=sunday
    
    # UNDERSTANDING CHECKBOXES
    # - Checkboxes can be checked or unchecked independently
    # - Multiple checkboxes can be checked at same time
    # - Unlike radio buttons, selecting one doesn't affect others
    
    # SELECTING CHECKBOXES (Checking them)
    
    # Check Sunday checkbox
    Select Checkbox    id=sunday
    # Select Checkbox = Checks (ticks) a checkbox
    # id=sunday = Locator for the checkbox element
    
    Log    Checked Sunday checkbox
    Sleep    1s
    
    # Check Monday checkbox
    Select Checkbox    id=monday
    # Both Sunday AND Monday are now checked
    
    Log    Checked Monday checkbox
    Sleep    1s
    
    # Check multiple checkboxes
    Select Checkbox    id=tuesday
    Select Checkbox    id=wednesday
    Select Checkbox    id=thursday
    
    Log    Checked Tuesday, Wednesday, Thursday
    Sleep    1s
    
    # VERIFICATION - Check if checkbox is selected
    
    Checkbox Should Be Selected    id=sunday
    # Checkbox Should Be Selected = Verifies checkbox is checked
    # Test fails if checkbox is NOT checked
    
    Log    Verification passed: Sunday is checked
    
    Checkbox Should Be Selected    id=monday
    Checkbox Should Be Selected    id=tuesday
    
    Log    Multiple checkboxes verified as checked
    
    # UNSELECTING CHECKBOXES (Unchecking them)
    
    Unselect Checkbox    id=monday
    # Unselect Checkbox = Unchecks (unticks) a checkbox
    # Removes the checkmark
    
    Log    Unchecked Monday checkbox
    Sleep    1s
    
    # Verify unchecked state
    Checkbox Should Not Be Selected    id=monday
    # Checkbox Should Not Be Selected = Verifies checkbox is NOT checked
    # Test fails if checkbox IS checked
    
    Log    Verification passed: Monday is unchecked
    
    # Uncheck more checkboxes
    Unselect Checkbox    id=tuesday
    Unselect Checkbox    id=wednesday
    
    Log    Unchecked Tuesday and Wednesday
    
    # CONDITIONAL CHECKBOX HANDLING
    
    # Check only if not already checked
    ${is_checked}=    Run Keyword And Return Status    
    ...               Checkbox Should Be Selected    id=friday
    # Returns True if checked, False if not checked
    
    Log    Is Friday checked? ${is_checked}
    
    Run Keyword If    ${is_checked}==False    Select Checkbox    id=friday
    # Selects only if currently unchecked
    
    Log    Ensured Friday is checked
    
    # TOGGLING CHECKBOX STATE
    
    # Get current state
    ${current_state}=    Run Keyword And Return Status    
    ...                  Checkbox Should Be Selected    id=saturday
    
    # Toggle it (check if unchecked, uncheck if checked)
    Run Keyword If    ${current_state}==True    
    ...    Unselect Checkbox    id=saturday
    ...    ELSE    
    ...    Select Checkbox    id=saturday
    
    Log    Toggled Saturday checkbox
    
    # USING CLICK ELEMENT (Alternative Method)
    
    # Click checkbox directly (toggles state)
    Click Element    id=sunday
    # If checked, becomes unchecked; if unchecked, becomes checked
    
    Sleep    0.5s
    
    # Click again to toggle back
    Click Element    id=sunday
    
    Log    Toggled Sunday using Click Element
    
    # GETTING CHECKBOX STATE
    
    ${is_selected}=    Get Element Attribute    id=sunday    checked
    # Gets 'checked' attribute
    # Returns 'true' if checked, None if unchecked
    
    Log    Sunday checkbox state: ${is_selected}
    
    # Screenshot
    Capture Page Screenshot    checkbox_example.png
    
    Close Browser

*** Test Cases ***
Select All Checkboxes Example
    [Documentation]    Programmatically check all checkboxes
    [Tags]    checkbox    intermediate
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # Scroll to checkbox area
    Scroll Element Into View    id=sunday
    
    # Get count of all day checkboxes
    ${checkbox_count}=    Get Element Count    xpath=//input[@type='checkbox' and @id='sunday' or @id='monday' or @id='tuesday' or @id='wednesday' or @id='thursday' or @id='friday' or @id='saturday']
    # Counts how many checkboxes match the criteria
    
    Log    Found ${checkbox_count} checkboxes
    
    # Select all day checkboxes
    Select Checkbox    id=sunday
    Select Checkbox    id=monday
    Select Checkbox    id=tuesday
    Select Checkbox    id=wednesday
    Select Checkbox    id=thursday
    Select Checkbox    id=friday
    Select Checkbox    id=saturday
    
    Log    All day checkboxes are now checked
    Sleep    2s
    
    # Verify all are checked
    Checkbox Should Be Selected    id=sunday
    Checkbox Should Be Selected    id=monday
    Checkbox Should Be Selected    id=tuesday
    Checkbox Should Be Selected    id=wednesday
    Checkbox Should Be Selected    id=thursday
    Checkbox Should Be Selected    id=friday
    Checkbox Should Be Selected    id=saturday
    
    Log    Verification: All checkboxes are checked
    
    # Unselect all
    Unselect Checkbox    id=sunday
    Unselect Checkbox    id=monday
    Unselect Checkbox    id=tuesday
    Unselect Checkbox    id=wednesday
    Unselect Checkbox    id=thursday
    Unselect Checkbox    id=friday
    Unselect Checkbox    id=saturday
    
    Log    All checkboxes are now unchecked
    Sleep    2s
    
    Close Browser

*** Keywords ***
Select Checkbox Safely
    [Documentation]    Selects checkbox only if not already selected
    [Arguments]    ${locator}
    
    Log    Checking checkbox: ${locator}
    
    # Wait for element to be visible
    Wait Until Element Is Visible    ${locator}    timeout=10s
    
    # Check current state
    ${is_selected}=    Run Keyword And Return Status    
    ...                Checkbox Should Be Selected    ${locator}
    
    # Select only if not already selected
    Run Keyword If    ${is_selected}==False    
    ...    Select Checkbox    ${locator}
    ...    ELSE    
    ...    Log    Checkbox already selected, skipping
    
    # Verify it's checked now
    Checkbox Should Be Selected    ${locator}
    
    Log    Checkbox is now checked: ${locator}

Unselect Checkbox Safely
    [Documentation]    Unselects checkbox only if currently selected
    [Arguments]    ${locator}
    
    Log    Unchecking checkbox: ${locator}
    
    # Wait for element
    Wait Until Element Is Visible    ${locator}    timeout=10s
    
    # Check current state
    ${is_selected}=    Run Keyword And Return Status    
    ...                Checkbox Should Be Selected    ${locator}
    
    # Unselect only if currently selected
    Run Keyword If    ${is_selected}==True    
    ...    Unselect Checkbox    ${locator}
    ...    ELSE    
    ...    Log    Checkbox already unselected, skipping
    
    # Verify it's unchecked now
    Checkbox Should Not Be Selected    ${locator}
    
    Log    Checkbox is now unchecked: ${locator}

Toggle Checkbox
    [Documentation]    Toggles checkbox state (check if unchecked, uncheck if checked)
    [Arguments]    ${locator}
    
    Log    Toggling checkbox: ${locator}
    
    # Get current state
    ${is_selected}=    Run Keyword And Return Status    
    ...                Checkbox Should Be Selected    ${locator}
    
    # Toggle based on current state
    Run Keyword If    ${is_selected}==True    
    ...    Unselect Checkbox    ${locator}
    ...    ELSE    
    ...    Select Checkbox    ${locator}
    
    Log    Checkbox toggled: ${locator}
```

### Checkbox HTML Structure Explained
```html
<!-- This is how checkboxes look in HTML -->
<input type="checkbox" id="sunday" name="day" value="sunday"> Sunday
<input type="checkbox" id="monday" name="day" value="monday"> Monday
<input type="checkbox" id="tuesday" name="day" value="tuesday"> Tuesday

<!-- 
Explanation:
- type="checkbox" makes it a checkbox
- id="sunday" is used in locator: id=sunday
- name="day" groups them logically but doesn't restrict selection
- value="sunday" is the value sent when form is submitted
- Multiple checkboxes can be checked simultaneously
- Each checkbox is independent
-->

<!-- Checkbox can have 'checked' attribute -->
<input type="checkbox" id="terms" checked> I agree to terms

<!-- 
- checked attribute means checkbox is initially checked
- Get Element Attribute returns 'true' when checked
-->
```

---

## Mouse Actions - Complete Example

```robot
*** Settings ***
Library           SeleniumLibrary

*** Variables ***
${URL}            https://testautomationpractice.blogspot.com/
${BROWSER}        chrome
${HOVER_URL}      https://www.w3schools.com/howto/tryit.asp?filename=tryhow_css_dropdown_navbar
${DRAG_URL}       https://www.w3schools.com/html/tryit.asp?filename=tryhtml5_draganddrop

*** Test Cases ***
Complete Click Actions Example
    [Documentation]    All types of click operations
    [Tags]    click    mouse    beginner
    
    # STEP 1: Setup
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # BASIC CLICKS
    
    # Click a button
    Click Button    xpath=//button[text()='Alert']
    # Click Button = Clicks a button element
    # Works with <button> and <input type="button">
    
    Sleep    1s
    Handle Alert    ACCEPT
    # Handles the alert that appeared
    
    Log    Clicked Alert button successfully
    
    # Click a link
    Click Link    partial link=Home
    # Click Link = Clicks a hyperlink (<a> tag)
    # partial link = Matches part of the link text
    
    Sleep    2s
    Log    Clicked Home link
    
    Go Back
    # Goes back to previous page (like browser back button)
    
    Sleep    2s
    
    # Click any element (most generic)
    Click Element    id=male
    # Click Element = Clicks any HTML element
    # Can click buttons, links, divs, images, etc.
    
    Log    Clicked Male radio button
    Sleep    1s
    
    # Click at specific coordinates
    Click Element At Coordinates    css=body    100    100
    # Click Element At Coordinates = Clicks at X,Y position
    # css=body = Base element
    # 100 = X coordinate (pixels from left)
    # 100 = Y coordinate (pixels from top)
    
    Log    Clicked at coordinates (100, 100)
    
    Close Browser

*** Test Cases ***
Double Click Example
    [Documentation]    Double-clicking elements
    [Tags]    doubleclick    mouse
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # Double-click an element
    Double Click Element    id=field1
    # Double Click Element = Simulates double-clicking
    # Like clicking twice very quickly
    # Often used to select text or open items
    
    Log    Double-clicked on field1
    Sleep    1s
    
    # Select all text after double-click
    Press Keys    None    CTRL+A
    
    Close Browser

*** Test Cases ***
Right Click (Context Menu) Example
    [Documentation]    Right-click to open context menu
    [Tags]    rightclick    contextmenu    mouse
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # Right-click an element
    Open Context Menu    id=name
    # Open Context Menu = Right-clicks element
    # Opens the browser's context menu (right-click menu)
    # Same as right-clicking with mouse
    
    Log    Right-clicked on name field
    Sleep    2s
    
    # Press Escape to close context menu
    Press Keys    None    ESCAPE
    
    Close Browser

*** Test Cases ***
Mouse Hover (Mouse Over) Example
    [Documentation]    Hovering mouse over elements
    [Tags]    hover    mouseover    mouse
    
    Open Browser    ${HOVER_URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    3s
    
    # Switch to result frame
    Select Frame    id=iframeResult
    
    # Hover over dropdown menu
    Mouse Over    xpath=//div[@class='dropdown']
    # Mouse Over = Moves mouse cursor over element
    # Triggers :hover CSS effects
    # Shows hidden menus, tooltips, etc.
    
    Log    Hovering over dropdown menu
    Sleep    2s
    
    # After hovering, submenu becomes visible
    Wait Until Element Is Visible    css=.dropdown-content    timeout=5s
    # Waits for submenu to appear
    
    # Click item in submenu
    Click Link    xpath=//a[contains(text(),'Link 1')]
    
    Log    Clicked on submenu item
    Sleep    2s
    
    Close Browser

*** Test Cases ***
Drag and Drop Example
    [Documentation]    Dragging elements and dropping them
    [Tags]    drag    drop    mouse
    
    Open Browser    ${DRAG_URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    3s
    
    # Switch to result frame
    Select Frame    id=iframeResult
    Sleep    1s
    
    # Drag and drop element
    Drag And Drop    id=drag1    id=div1
    # Drag And Drop = Drags source element to target element
    # id=drag1 = Element to drag (source)
    # id=div1 = Where to drop it (target)
    # Simulates click-hold-move-release
    
    Log    Dragged image to box
    Sleep    2s
    
    # Drag it back
    Drag And Drop    id=drag1    id=div2
    
    Log    Dragged image back
    Sleep    2s
    
    Close Browser

*** Test Cases ***
Drag and Drop By Offset Example
    [Documentation]    Dragging by pixel offset
    [Tags]    drag    offset    mouse
    
    Open Browser    https://jqueryui.com/draggable/    ${BROWSER}
    Maximize Browser Window
    Sleep    3s
    
    # Switch to demo frame
    Select Frame    css=.demo-frame
    
    # Drag element by offset
    Drag And Drop By Offset    id=draggable    200    100
    # Drag And Drop By Offset = Drags element by X,Y pixels
    # id=draggable = Element to drag
    # 200 = Move 200 pixels to the right (X-axis)
    # 100 = Move 100 pixels down (Y-axis)
    
    Log    Dragged element 200px right and 100px down
    Sleep    2s
    
    # Drag back using negative offset
    Drag And Drop By Offset    id=draggable    -200    -100
    
    Log    Dragged element back to original position
    Sleep    2s
    
    Close Browser

*** Test Cases ***
Mouse Down and Mouse Up Example
    [Documentation]    Manual mouse button press and release
    [Tags]    mousedown    mouseup    mouse
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # Simulate mouse button press (hold down)
    Mouse Down    id=name
    # Mouse Down = Presses mouse button down (holds it)
    # Like clicking but not releasing
    
    Log    Mouse button pressed down on name field
    Sleep    1s
    
    # Simulate mouse button release
    Mouse Up    id=name
    # Mouse Up = Releases mouse button
    # Completes the click action
    
    Log    Mouse button released
    Sleep    1s
    
    # This is used for custom drag operations:
    # Mouse Down on source -> Mouse Over target -> Mouse Up
    
    Close Browser

*** Keywords ***
Hover And Click
    [Documentation]    Custom keyword to hover then click
    [Arguments]    ${hover_locator}    ${click_locator}
    # ${hover_locator} = Element to hover over
    # ${click_locator} = Element to click after hover
    
    Log    Hovering over: ${hover_locator}
    
    # Hover over first element
    Mouse Over    ${hover_locator}
    
    # Wait for submenu/tooltip to appear
    Sleep    1s
    
    # Click the target element
    Wait Until Element Is Visible    ${click_locator}    timeout=5s
    Click Element    ${click_locator}
    
    Log    Clicked: ${click_locator} after hovering

Custom Drag And Drop
    [Documentation]    Manual drag and drop using mouse actions
    [Arguments]    ${source}    ${target}
    
    Log    Dragging ${source} to ${target}
    
    # Scroll source into view
    Scroll Element Into View    ${source}
    
    # Press down on source
    Mouse Down    ${source}
    
    Sleep    0.5s
    
    # Move to target
    Mouse Over    ${target}
    
    Sleep    0.5s
    
    # Release on target
    Mouse Up    ${target}
    
    Log    Drag and drop completed
```

### Mouse Actions Summary Table

| Action | Keyword | Description |
|--------|---------|-------------|
| Click | `Click Element` | Single click on any element |
| Click Button | `Click Button` | Click button element specifically |
| Click Link | `Click Link` | Click hyperlink specifically |
| Double Click | `Double Click Element` | Double-click element |
| Right Click | `Open Context Menu` | Right-click (opens context menu) |
| Hover | `Mouse Over` | Move mouse over element |
| Drag & Drop | `Drag And Drop` | Drag source to target |
| Drag by Offset | `Drag And Drop By Offset` | Drag by pixel distance |
| Press Down | `Mouse Down` | Hold mouse button down |
| Release | `Mouse Up` | Release mouse button |
| Click at XY | `Click Element At Coordinates` | Click at specific position |

---

## Waits and Timing - Complete Example

```robot
*** Settings ***
Library           SeleniumLibrary

*** Variables ***
${URL}            https://testautomationpractice.blogspot.com/
${BROWSER}        chrome

*** Test Cases ***
Complete Wait Strategies Example
    [Documentation]    Comprehensive guide to waiting in Robot Framework
    [Tags]    wait    timing    synchronization
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    
    # TYPES OF WAITS IN ROBOT FRAMEWORK
    
    # 1. IMPLICIT WAIT (Global Setting)
    Set Selenium Implicit Wait    10 seconds
    # Set Selenium Implicit Wait = Default wait for ALL elements
    # Selenium waits up to 10s for elements to appear
    # Applied automatically to all element operations
    # Set once at the beginning
    
    Log    Implicit wait set to 10 seconds
    
    # 2. EXPLICIT WAIT - Wait for specific condition
    
    # Wait for element to be visible
    Wait Until Element Is Visible    id=name    timeout=10s
    # Wait Until Element Is Visible = Waits until element appears on page
    # timeout=10s = Maximum time to wait
    # Fails test if element doesn't appear within timeout
    
    Log    Element became visible
    
    # Wait for element to be enabled (clickable)
    Wait Until Element Is Enabled    id=name    timeout=10s
    # Wait Until Element Is Enabled = Waits until element is clickable
    # Some elements are visible but disabled/greyed out
    # This waits until they become enabled
    
    Log    Element is now enabled
    
    # Wait for element to disappear
    Click Button    xpath=//button[text()='Alert']
    Sleep    0.5s
    Handle Alert    ACCEPT
    
    # Wait for page to contain specific text
    Wait Until Page Contains    Alert    timeout=5s
    # Wait Until Page Contains = Waits for text to appear anywhere on page
    # Useful after submitting forms or loading data
    
    Log    Page contains expected text
    
    # Wait for element to contain text
    Wait Until Element Contains    css=h1    Automation    timeout=5s
    # Wait Until Element Contains = Waits for element to have specific text
    # Checks element's text content
    
    Log    Element contains expected text
    
    # Wait for element to NOT be visible (disappear)
    # Note: This is示例代码, adjust for actual loading spinner
    # Wait Until Element Is Not Visible    id=loadingSpinner    timeout=30s
    # Waits for loading spinners/overlays to disappear
    
    # 3. STATIC WAIT (Sleep) - Pause execution
    Sleep    2 seconds
    # Sleep = Pauses for exact duration
    # NOT recommended except for debugging
    # Use explicit waits instead when possible
    
    Log    Slept for 2 seconds
    
    # Can use different time units
    Sleep    2s        # 2 seconds
    Sleep    500ms     # 500 milliseconds
    Sleep    0.5s      # 0.5 seconds (half second)
    
    # 4. WAIT UNTIL KEYWORD SUCCEEDS - Retry mechanism
    Wait Until Keyword Succeeds    30s    2s    Element Should Be Visible    id=name
    # Wait Until Keyword Succeeds = Retries keyword until it passes
    # 30s = Maximum total time to try
    # 2s = Wait between each retry
    # Element Should Be Visible = The keyword to retry
    # Retries every 2s for up to 30s total
    
    Log    Keyword succeeded within retry period
    
    # 5. SET SELENIUM SPEED - Delay between each operation
    Set Selenium Speed    0.5 seconds
    # Set Selenium Speed = Adds delay between every Selenium command
    # 0.5 seconds = Wait half second after each action
    # Useful for debugging or demo purposes
    # Slows down entire test execution
    
    Log    Selenium speed set to 0.5 seconds
    
    # Now every action will have 0.5s delay
    Input Text    id=name    Test User
    Input Text    id=email    test@example.com
    
    # Reset to no delay
    Set Selenium Speed    0 seconds
    
    Log    Selenium speed reset to 0 seconds
    
    # 6. CUSTOM WAIT CONDITIONS
    
    # Wait for element count
    Wait Until Keyword Succeeds    10s    1s    
    ...    Should Be True    ${element_count} > 0
    ...    element_count=${Get Element Count xpath=//input}
    
    # Wait for page to load completely
    Wait For Condition    return document.readyState == 'complete'    timeout=30s
    # Wait For Condition = Waits for JavaScript condition to be true
    # Executes JavaScript and waits for it to return true
    
    Log    Page loaded completely
    
    Close Browser

*** Test Cases ***
Real World Wait Examples
    [Documentation]    Practical waiting scenarios
    [Tags]    wait    realworld
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Set Selenium Implicit Wait    10s
    
    # SCENARIO 1: Wait for AJAX content to load
    Log    Waiting for dynamic content
    
    # Click button that triggers AJAX
    # Wait Until Element Is Visible    id=loadButton    timeout=5s
    # Click Button    id=loadButton
    
    # Wait for loading spinner to appear then disappear
    # Wait Until Element Is Visible    id=spinner    timeout=5s
    # Wait Until Element Is Not Visible    id=spinner    timeout=30s
    
    # Wait for results to appear
    # Wait Until Element Is Visible    css=.results    timeout=10s
    
    Log    Dynamic content loaded
    
    # SCENARIO 2: Wait for element attribute to change
    # Sometimes need to wait for attribute like 'class' or 'disable' to change
    
    Wait Until Keyword Succeeds    10s    1s    
    ...    Element Should Not Be Visible    css=.loading-overlay
    
    Log    Overlay disappeared
    
    # SCENARIO 3: Wait for dropdown to populate
    # After selecting country, wait for state dropdown to populate
    
    # Select From List By Label    id=country    United States
    # Sleep    1s    # Wait for states to load
    # Wait Until Element Is Enabled    id=state    timeout=10s
    # ${state_count}=    Get Element Count    css=#state option
    # Should Be True    ${state_count} > 1    # More than just placeholder
    
    # SCENARIO 4: Wait for file download to complete
    # This requires custom logic - Robot Framework doesn't track downloads directly
    # Usually check if file exists in download folder
    
    # ${download_complete}=    Wait Until Keyword Succeeds    60s    5s
    # ...    OperatingSystem.File Should Exist    ${DOWNLOAD_DIR}/report.pdf
    
    Log    All real-world scenarios demonstrated
    
    Close Browser

*** Keywords ***
Wait For Element And Click
    [Documentation]    Waits for element then clicks it safely
    [Arguments]    ${locator}    ${timeout}=10s
    
    Log    Waiting for element: ${locator}
    
    # Wait for element to be visible
    Wait Until Element Is Visible    ${locator}    timeout=${timeout}
    
    # Wait for element to be enabled
    Wait Until Element Is Enabled    ${locator}    timeout=${timeout}
    
    # Scroll element into view
    Scroll Element Into View    ${locator}
    
    # Click the element
    Click Element    ${locator}
    
    Log    Successfully clicked: ${locator}

Wait For Text To Appear
    [Documentation]    Waits for specific text on page
    [Arguments]    ${text}    ${timeout}=10s
    
    Log    Waiting for text: ${text}
    
    Wait Until Page Contains    ${text}    timeout=${timeout}
    
    Log    Text appeared: ${text}

Wait For Loading To Complete
    [Documentation]    Waits for common loading indicators to disappear
    [Arguments]    ${timeout}=30s
    
    Log    Waiting for page to finish loading
    
    # Wait for common loading indicators (adjust locators for your app)
    ${spinner_present}=    Run Keyword And Return Status    
    ...    Element Should Be Visible    css=.spinner
    
    Run Keyword If    ${spinner_present}    
    ...    Wait Until Element Is Not Visible    css=.spinner    timeout=${timeout}
    
    ${overlay_present}=    Run Keyword And Return Status    
    ...    Element Should Be Visible    css=.loading-overlay
    
    Run Keyword If    ${overlay_present}    
    ...    Wait Until Element Is Not Visible    css=.loading-overlay    timeout=${timeout}
    
    # Wait for page ready state
    Wait For Condition    return document.readyState == 'complete'    timeout=${timeout}
    
    Log    Loading completed
```

### Wait Strategy Decision Guide

```
When to use which wait:

1. Implicit Wait (Set Selenium Implicit Wait)
   - Set ONCE at test start
   - Applies globally to all elements
   - Good baseline waiting strategy

2. Explicit Waits (Wait Until...)
   - For specific conditions
   - Loading spinners to disappear
   - Dynamic content to appear
   - Buttons to become enabled
   - Most commonly used

3. Sleep
   - AVOID if possible
   - Only for debugging
   - Only when no other wait works
   - Makes tests slower and flaky

4. Wait Until Keyword Succeeds
   - For flaky elements
   - When element appears/disappears repeatedly
   - When you need retry logic

5. Set Selenium Speed
   - Only for debugging
   - Only for demos/presentations
   - Slows down everything
```

---

## Alerts and Popups - Complete Example

```robot
*** Settings ***
Library           SeleniumLibrary

*** Variables ***
${URL}            https://testautomationpractice.blogspot.com/
${BROWSER}        chrome

*** Test Cases ***
Complete Alert Handling Example
    [Documentation]    Complete guide to handling JavaScript alerts
    [Tags]    alert    popup    beginner
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # JAVASCRIPT ALERTS - Three types:
    # 1. Alert Box - Just OK button
    # 2. Confirm Box - OK and Cancel buttons
    # 3. Prompt Box - Input field with OK and Cancel
    
    # TYPE 1: ALERT BOX (Simple notification with OK button)
    
    # Trigger alert
    Click Button    xpath=//button[text()='Alert']
    # This opens a JavaScript alert
    
    Sleep    1s    # See the alert
    
    # Accept alert (Click OK)
    Handle Alert    ACCEPT
    # Handle Alert = Handles JavaScript alert/confirm/prompt
    # ACCEPT = Clicks OK button
    # Closes the alert
    
    Log    Accepted alert successfully
    Sleep    1s
    
    # TYPE 2: CONFIRM BOX (Has OK and Cancel buttons)
    
    # Trigger confirm dialog (if exists on page)
    # Click Button    xpath=//button[text()='Confirm Box']
    # Sleep    1s
    
    # Accept confirm (Click OK)
    # Handle Alert    ACCEPT
    # Log    Clicked OK on confirm dialog
    # Sleep    1s
    
    # Trigger again to test Cancel
    # Click Button    xpath=//button[text()='Confirm Box']
    # Sleep    1s
    
    # Dismiss confirm (Click Cancel)
    # Handle Alert    DISMISS
    # Handle Alert DISMISS = Clicks Cancel button
    # Rejects/cancels the action
    
    # Log    Clicked Cancel on confirm dialog
    # Sleep    1s
    
    # TYPE 3: GET ALERT TEXT (Read message before closing)
    
    Click Button    xpath=//button[text()='Alert']
    Sleep    1s
    
    # Get alert text without closing it
    ${alert_message}=    Handle Alert    LEAVE
    # Handle Alert LEAVE = Gets alert text without clicking OK/Cancel
    # Returns the alert message
    # Alert remains open
    
    Log    Alert message is: ${alert_message}
    
    # Now accept it to close
    Handle Alert    ACCEPT
    
    Log    Got alert text and accepted
    Sleep    1s
    
    # ALTERNATIVE: Get text and accept in two steps
    Click Button    xpath=//button[text()='Alert']
    Sleep    1s
    
    Alert Should Be Present    action=ACCEPT
    # Alert Should Be Present = Checks if alert is present
    # action=ACCEPT = Also accepts the alert
    # Can use action=DISMISS or action=LEAVE
    
    Log    Verified alert presence and accepted
    Sleep    1s
    
    # TYPE 4: PROMPT BOX (Alert with input field)
    
    # Note: testautomationpractice might not have prompt
    # This is示例代码 for prompt handling
    
    # Click Button    xpath=//button[text()='Prompt']
    # Sleep    1s
    
    # Input text into prompt and accept
    # Input Text Into Alert    John Doe    action=ACCEPT
    # Input Text Into Alert = Types into prompt input field
    # John Doe = Text to enter
    # action=ACCEPT = Then clicks OK
    
    # Log    Entered text into prompt and accepted
    
    # VERIFY ALERT TEXT
    Click Button    xpath=//button[text()='Alert']
    Sleep    1s
    
    # Verify alert contains expected text
    Alert Should Be Present    I am an alert box!    action=ACCEPT
    # First parameter = Expected alert text (optional)
    # Verifies text matches, then accepts
    
    Log    Verified alert text matches expected value
    
    Close Browser

*** Test Cases ***
Alert Timeout Example
    [Documentation]    Handling alerts with timeout
    [Tags]    alert    timeout
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # Click button that shows alert
    Click Button    xpath=//button[text()='Alert']
    
    # Wait for alert to appear (with timeout)
    Wait Until Keyword Succeeds    5s    0.5s    
    ...    Alert Should Be Present    action=LEAVE
    
    Log    Alert appeared within timeout
    
    # Accept the alert
    Handle Alert    ACCEPT
    
    Close Browser

*** Test Cases ***
No Alert Present Example
    [Documentation]    Handling case when alert is not present
    [Tags]    alert    conditional
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    2s
    
    # Check if alert is present (doesn't fail if absent)
    ${alert_present}=    Run Keyword And Return Status    
    ...    Alert Should Be Present    action=LEAVE
    # Returns True if alert present, False if not
    # Doesn't fail test
    
    Log    Is alert present? ${alert_present}
    
    # Handle conditionally
    Run Keyword If    ${alert_present}    Handle Alert    ACCEPT
    
    Log    Handled alert if it was present
    
    Close Browser

*** Keywords ***
Accept Alert If Present
    [Documentation]    Accepts alert if it exists, does nothing if not
    
    Log    Checking for alert
    
    # Check if alert is present
    ${alert_exists}=    Run Keyword And Return Status    
    ...    Alert Should Be Present    action=LEAVE
    
    # Accept if present
    Run Keyword If    ${alert_exists}    
    ...    Handle Alert    ACCEPT    
    ...    ELSE    
    ...    Log    No alert to accept
    
    Log    Alert handling completed

Get Alert Text And Accept
    [Documentation]    Gets alert text, logs it, and accepts
    
    Log    Getting alert text
    
    # Wait for alert
    Wait Until Keyword Succeeds    5s    0.5s    
    ...    Alert Should Be Present    action=LEAVE
    
    # Get text
    ${alert_text}=    Handle Alert    LEAVE
    Log    Alert says: ${alert_text}
    
    # Accept alert
    Handle Alert    ACCEPT
    Log    Alert accepted
    
    # Return the text
    RETURN    ${alert_text}

Dismiss Alert If Present
    [Documentation]    Dismisses (cancels) alert if it exists
    
    Log    Checking for alert to dismiss
    
    ${alert_exists}=    Run Keyword And Return Status    
    ...    Alert Should Be Present    action=LEAVE
    
    Run Keyword If    ${alert_exists}    
    ...    Handle Alert    DISMISS    
    ...    ELSE    
    ...    Log    No alert to dismiss
    
    Log    Alert dismissal completed
```

### Alert Actions Summary

| Action | Keyword | Description |
|--------|---------|-------------|
| Accept (OK) | `Handle Alert    ACCEPT` | Clicks OK button |
| Dismiss (Cancel) | `Handle Alert    DISMISS` | Clicks Cancel button |
| Get Text | `Handle Alert    LEAVE` | Gets alert text, keeps alert open |
| Verify & Accept | `Alert Should Be Present    action=ACCEPT` | Checks alert exists and accepts |
| Verify Text | `Alert Should Be Present    Expected Text    action=ACCEPT` | Verifies text matches |

---

## Frames - Complete Example

```robot
*** Settings ***
Library           SeleniumLibrary

*** Variables ***
${URL}            https://www.w3schools.com/html/tryit.asp?filename=tryhtml_iframe
${BROWSER}        chrome

*** Test Cases ***
Complete Frame Handling Example
    [Documentation]    Complete guide to working with iframes
    [Tags]    frame    iframe    beginner
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    3s
    
    # UNDERSTANDING FRAMES/IFRAMES
    # - Frame = A separate HTML document embedded in a page
    # - Selenium can only interact with ONE frame at a time
    # - Must "switch" to frame before accessing its elements
    # - Must "switch back" to main page to access page elements
    
    # MAIN PAGE (Outside frame)
    # We're currently on the main page
    
    # Try to click element on main page
    Click Element    id=runbtn
    # Runs the code (this button is on main page, not in frame)
    
    Log    Clicked button on main page
    Sleep    2s
    
    # SWITCH TO FRAME
    
    # Method 1: Switch by ID
    Select Frame    id=iframeResult
    # Select Frame = Switches context to frame
    # Now we can interact with elements inside this frame
    # Cannot interact with main page elements until we switch back
    
    Log    Switched to iframe with id=iframeResult
    Sleep    1s
    
    # Now interact with elements INSIDE the frame
    # The result area is inside the iframe
    ${frame_text}=    Get Text    css=body
    Log    Text inside frame: ${frame_text}
    
    # Verify we're in the frame
    Page Should Contain    HTML Tutorial
    
    Log    Successfully accessed content inside frame
    
    # SWITCH BACK TO MAIN PAGE
    
    Unselect Frame
    # Unselect Frame = Switches back to main page content
    # Now can interact with main page elements again
    # Cannot interact with frame elements
    
    Log    Switched back to main page
    Sleep    1s
    
    # Now we can interact with main page again
    Click Element    id=runbtn
    Log    Clicked main page button again
    
    Close Browser

*** Test Cases ***
Multiple Frames Example
    [Documentation]    Working with multiple frames
    [Tags]    frame    multiple
    
    Open Browser    https://www.w3schools.com/tags/tryit.asp?filename=tryhtml_iframe_frameborder_css    ${BROWSER}
    Maximize Browser Window
    Sleep    3s
    
    # Page has nested or multiple frames
    
    # Switch to first frame
    Select Frame    id=iframeResult
    Log    In first frame
    Sleep    1s
    
    # Access elements in first frame
    ${text1}=    Get Text    css=body
    Log    First frame content: ${text1}
    
    # Switch back to main
    Unselect Frame
    
    # Switch to another frame (if exists)
    # Select Frame    id=anotherFrame
    # Log    In second frame
    
    # Switch back to main again
    # Unselect Frame
    
    Close Browser

*** Test Cases ***
Nested Frames Example
    [Documentation]    Handling frames within frames
    [Tags]    frame    nested
    
    # Create a custom HTML page with nested frames for demo
    # This is示例代码
    
    Open Browser    https://www.w3schools.com/html/tryit.asp?filename=tryhtml_iframe    ${BROWSER}
    Maximize Browser Window
    Sleep    3s
    
    # Switch to outer frame
    Select Frame    id=iframeResult
    Log    Entered outer frame
    Sleep    1s
    
    # If there's a frame inside this frame
    # Select Frame    id=innerFrame
    # Log    Entered inner frame (nested)
    # Access inner frame elements here
    
    # Go back one level (to outer frame)
    # Unselect Frame would go to main page
    # To go to parent frame only:
    # Execute JavaScript    window.parent.focus()
    
    # Go back to main page
    Unselect Frame
    Log    Back to main page
    
    Close Browser

*** Test Cases ***
Frame By Different Locators Example
    [Documentation]    Different ways to locate and switch to frames
    [Tags]    frame    locators
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Sleep    3s
    
    # METHOD 1: By ID
    Select Frame    id=iframeResult
    Log    Selected frame by ID
    Page Should Contain    HTML
    Unselect Frame
    Sleep    1s
    
    # METHOD 2: By Name (if frame has name attribute)
    # Select Frame    name=resultFrame
    # Log    Selected frame by name
    # Unselect Frame
    
    # METHOD 3: By CSS Selector
    Select Frame    css=#iframeResult
    Log    Selected frame by CSS selector
    Page Should Contain    HTML
    Unselect Frame
    Sleep    1s
    
    # METHOD 4: By XPath
    Select Frame    xpath=//iframe[@id='iframeResult']
    Log    Selected frame by XPath
    Page Should Contain    HTML
    Unselect Frame
    Sleep    1s
    
    # METHOD 5: By Index (position on page)
    Select Frame    css=iframe    # Selects first iframe
    Log    Selected frame by index
    Page Should Contain    HTML
    Unselect Frame
    
    Close Browser

*** Test Cases ***
Frame Wait Example
    [Documentation]    Waiting for frame to load
    [Tags]    frame    wait
    
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    
    # Wait for frame to be available
    Wait Until Element Is Visible    id=iframeResult    timeout=10s
    # Waits for frame element to appear
    
    Log    Frame is visible and ready
    
    # Switch to frame
    Select Frame    id=iframeResult
    
    # Wait for content inside frame to load
    Wait Until Page Contains    HTML    timeout=10s
    # Waits for content inside frame
    
    Log    Frame content loaded
    
    # Access elements
    ${content}=    Get Text    css=body
    Log    Frame content: ${content}
    
    Unselect Frame
    Close Browser

*** Keywords ***
Switch To Frame And Wait
    [Documentation]    Switches to frame and waits for it to load
    [Arguments]    ${frame_locator}    ${expected_text}=${EMPTY}
    
    Log    Switching to frame: ${frame_locator}
    
    # Wait for frame to be visible
    Wait Until Element Is Visible    ${frame_locator}    timeout=10s
    
    # Switch to frame
    Select Frame    ${frame_locator}
    
    Log    Switched to frame successfully
    
    # Wait for frame content to load (if expected text provided)
    Run Keyword If    '${expected_text}'!='${EMPTY}'    
    ...    Wait Until Page Contains    ${expected_text}    timeout=10s
    
    Log    Frame content loaded

Access Element In Frame
    [Documentation]    Switches to frame, interacts with element, switches back
    [Arguments]    ${frame_locator}    ${element_locator}    ${action}=click
    
    Log    Accessing element in frame
    
    # Switch to frame
    Wait Until Element Is Visible    ${frame_locator}    timeout=10s
    Select Frame    ${frame_locator}
    
    # Wait for element in frame
    Wait Until Element Is Visible    ${element_locator}    timeout=10s
    
    # Perform action
    Run Keyword If    '${action}'=='click'    
    ...    Click Element    ${element_locator}
    ...    ELSE IF    '${action}'=='input'    
    ...    Input Text    ${element_locator}    Test Value
    ...    ELSE    
    ...    Log    No action specified
    
    Log    Performed ${action} on element: ${element_locator}
    
    # Switch back to main page
    Unselect Frame
    
    Log    Switched back to main page
```

### Frame/iFrame Concepts Explained

```html
<!-- MAIN PAGE HTML -->
<html>
  <body>
    <h1>Main Page</h1>
    <button id="mainButton">Main Button</button>
    
    <!-- This is an iframe (embedded page) -->
    <iframe id="myFrame" src="page2.html">
      <!-- Everything inside here is a SEPARATE page -->
      <h2>Inside Frame</h2>
      <button id="frameButton">Frame Button</button>
    </iframe>
    
    <button id="anotherMainButton">Another Main Button</button>
  </body>
</html>

<!--
KEY CONCEPTS:

1. Default Context = Main Page
   - Can click id=mainButton
   - Cannot click id=frameButton (it's in different context)

2. After Select Frame id=myFrame:
   - Can click id=frameButton
   - Cannot click id=mainButton (it's in different context)

3. After Unselect Frame:
   - Back to main page context
   - Can click id=mainButton again
   - Cannot click id=frameButton anymore

REMEMBER:
- Always Switch to frame before accessing its elements
- Always Switch back to main after you're done
- Can only be in ONE context at a time
-->
```

---

## Complete Real Project Example

```robot
*** Settings ***
Library           SeleniumLibrary
Library           Collections
Library           String

# Setup and Teardown for entire test suite
Suite Setup       Open Browser And Navigate
Suite Teardown    Close All Browsers

# Runs after each test case if test fails
Test Teardown     Run Keyword If Test Failed    Capture Page Screenshot

*** Variables ***
# Test Data
${URL}            https://demo.automationtesting.in/Register.html
${BROWSER}        chrome

# User Details
${FIRST_NAME}     John
${LAST_NAME}      Doe
${ADDRESS}        123 Main Street, Apartment 4B
${EMAIL}          john.doe.test@example.com
${PHONE}          1234567890
${PASSWORD}       Test@12345

# Dropdowns
${COUNTRY}        India
${SKILLS}         APIs
${COUNTRY2}       Australia

*** Test Cases ***
Complete Registration Form Test
    [Documentation]    End-to-end test filling complete registration form
    ...                Tests all UI elements: text fields, radio buttons,
    ...                checkboxes, dropdowns, date picker, file upload
    [Tags]    regression    smoke    registration
    
    Log    ========== Starting Registration Test ==========
    
    # Fill personal information
    Fill Personal Details
    
    # Fill address
    Fill Address Details
    
    # Select gender
    Select Gender
    
    # Select hobbies
    Select Hobbies
    
    # Select skills dropdown
    Select Skills From Dropdown
    
    # Select country dropdowns
    Select Country Information
    
    # Select date of birth
    # Select Date Of Birth
    
    # Set password
    Set Password Fields
    
    # Upload file (optional - may not work on all systems)
    # Upload Profile Picture
    
    # Verify all fields before submit
    Verify All Fields Are Filled
    
    # Submit form
    # Submit Registration Form
    
    Log    ========== Registration Test Completed Successfully ==========
    
    # Take final screenshot
    Capture Page Screenshot    registration_completed.png

*** Keywords ***
# ========== SETUP KEYWORDS ==========

Open Browser And Navigate
    [Documentation]    Opens browser and navigates to registration page
    
    Log    Opening browser: ${BROWSER}
    
    # Open browser
    Open Browser    ${URL}    ${BROWSER}
    
    # Maximize window
    Maximize Browser Window
    
    # Set implicit wait
    Set Selenium Implicit Wait    10 seconds
    
    # Set speed for demo (remove for actual testing)
    Set Selenium Speed    0.3 seconds
    
    # Wait for page to load
    Wait Until Page Contains Element    css=input[placeholder='First Name']    timeout=15s
    
    Log    Browser opened and page loaded successfully
    
    # Take initial screenshot
    Capture Page Screenshot    00_initial_page.png

# ========== FORM FILLING KEYWORDS ==========

Fill Personal Details
    [Documentation]    Fills first name, last name, email, phone
    
    Log    Filling personal details
    
    # First Name
    Wait Until Element Is Visible    css=input[placeholder='First Name']    timeout=10s
    Input Text    css=input[placeholder='First Name']    ${FIRST_NAME}
    Log    Entered first name: ${FIRST_NAME}
    Sleep    0.5s
    
    # Last Name
    Input Text    css=input[placeholder='Last Name']    ${LAST_NAME}
    Log    Entered last name: ${LAST_NAME}
    Sleep    0.5s
    
    # Email
    Input Text    css=input[type='email']    ${EMAIL}
    Log    Entered email: ${EMAIL}
    Sleep    0.5s
    
    # Phone
    Input Text    css=input[type='tel']    ${PHONE}
    Log    Entered phone: ${PHONE}
    Sleep    0.5s
    
    # Verify values entered
    ${entered_firstname}=    Get Value    css=input[placeholder='First Name']
    Should Be Equal    ${entered_firstname}    ${FIRST_NAME}
    
    Log    Personal details filled successfully
    Capture Page Screenshot    01_personal_details.png

Fill Address Details
    [Documentation]    Fills address textarea
    
    Log    Filling address
    
    # Scroll to address field
    Scroll Element Into View    css=textarea[ng-model='Adress']
    
    # Enter address
    Input Text    css=textarea[ng-model='Adress']    ${ADDRESS}
    Log    Entered address: ${ADDRESS}
    Sleep    0.5s
    
    Log    Address filled successfully
    Capture Page Screenshot    02_address.png

Select Gender
    [Documentation]    Selects gender radio button
    
    Log    Selecting gender
    
    # Scroll to gender section
    Scroll Element Into View    css=input[value='Male']
    
    # Select Male radio button
    Click Element    css=input[value='Male']
    # Alternative: Select Radio Button    Gender    Male
    Log    Selected gender: Male
    Sleep    0.5s
    
    # Verify selection
    ${is_checked}=    Get Element Attribute    css=input[value='Male']    checked
    Should Be Equal As Strings    ${is_checked}    true
    
    Log    Gender selected successfully
    Capture Page Screenshot    03_gender.png

Select Hobbies
    [Documentation]    Selects hobby checkboxes
    
    Log    Selecting hobbies
    
    # Scroll to hobbies
    Scroll Element Into View    id=checkbox1
    
    # Select Cricket checkbox
    Click Element    id=checkbox1
    Log    Selected hobby: Cricket
    Sleep    0.5s
    
    # Select Movies checkbox
    Click Element    id=checkbox2
    Log    Selected hobby: Movies
    Sleep    0.5s
    
    # Verify checkboxes are selected
    ${cricket_checked}=    Get Element Attribute    id=checkbox1    checked
    ${movies_checked}=    Get Element Attribute    id=checkbox2    checked
    
    Should Be Equal As Strings    ${cricket_checked}    true
    Should Be Equal As Strings    ${movies_checked}    true
    
    Log    Hobbies selected successfully
    Capture Page Screenshot    04_hobbies.png

Select Skills From Dropdown
    [Documentation]    Selects from Skills dropdown
    
    Log    Selecting skills
    
    # Scroll to skills dropdown
    Scroll Element Into View    id=Skills
    
    # Click dropdown to open it
    Click Element    id=Skills
    Sleep    0.5s
    
    # Select skill
    Select From List By Label    id=Skills    ${SKILLS}
    Log    Selected skill: ${SKILLS}
    Sleep    0.5s
    
    # Verify selection
    ${selected_skill}=    Get Selected List Label    id=Skills
    Should Be Equal    ${selected_skill}    ${SKILLS}
    
    Log    Skills selected successfully
    Capture Page Screenshot    05_skills.png

Select Country Information
    [Documentation]    Selects from country dropdowns
    
    Log    Selecting countries
    
    # Scroll to country section
    Scroll Element Into View    id=countries
    
    # Select Country dropdown (searchable dropdown)
    Click Element    css=span[role='combobox']
    Sleep    1s
    
    # Type to search
    Input Text    css=input[role='textbox']    ${COUNTRY}
    Sleep    1s
    
    # Press Enter to select
    Press Keys    css=input[role='textbox']    RETURN
    Log    Selected country: ${COUNTRY}
    Sleep    1s
    
    # Select Country (Select2 dropdown)
    Scroll Element Into View    id=countries
    Select From List By Label    id=countries    ${COUNTRY2}
    Log    Selected country: ${COUNTRY2}
    Sleep    0.5s
    
    Log    Countries selected successfully
    Capture Page Screenshot    06_countries.png

Select Date Of Birth
    [Documentation]    Selects date of birth from date picker
    
    Log    Selecting date of birth
    
    # Scroll to DOB section
    Scroll Element Into View    id=yearbox
    
    # Select Year
    Select From List By Label    id=yearbox    1990
    Log    Selected year: 1990
    Sleep    0.5s
    
    # Select Month
    Select From List By Label    css=select[placeholder='Month']    January
    Log    Selected month: January
    Sleep    0.5s
    
    # Select Day
    Select From List By Label    id=daybox    15
    Log    Selected day: 15
    Sleep    0.5s
    
    Log    Date of birth selected successfully
    Capture Page Screenshot    07_date_of_birth.png

Set Password Fields
    [Documentation]    Fills password and confirm password fields
    
    Log    Setting passwords
    
    # Scroll to password section
    Scroll Element Into View    id=firstpassword
    
    # Enter password
    Input Text    id=firstpassword    ${PASSWORD}
    Log    Entered password (hidden for security)
    Sleep    0.5s
    
    # Enter confirm password  
    Input Text    id=secondpassword    ${PASSWORD}
    Log    Entered confirm password (hidden for security)
    Sleep    0.5s
    
    # Verify passwords match (by value)
    ${pass1}=    Get Value    id=firstpassword
    ${pass2}=    Get Value    id=secondpassword
    Should Be Equal    ${pass1}    ${pass2}    Passwords do not match!
    
    Log    Passwords set successfully
    Capture Page Screenshot    08_passwords.png

Upload Profile Picture
    [Documentation]    Uploads profile picture file
    
    Log    Uploading profile picture
    
    # Note: This requires actual file path
    # Scroll to upload button
    Scroll Element Into View    id=imagesrc
    
    # Choose file (requires actual file path)
    # Choose File    id=imagesrc    ${CURDIR}/test_image.jpg
    
    Log    File upload section reached
    Capture Page Screenshot    09_file_upload.png

Verify All Fields Are Filled
    [Documentation]    Verifies all required fields have values
    
    Log    Verifying all fields are filled
    
    # Verify first name
    ${firstname}=    Get Value    css=input[placeholder='First Name']
    Should Not Be Empty    ${firstname}    First name is empty!
    
    # Verify last name
    ${lastname}=    Get Value    css=input[placeholder='Last Name']
    Should Not Be Empty    ${lastname}    Last name is empty!
    
    # Verify email
    ${email}=    Get Value    css=input[type='email']
    Should Not Be Empty    ${email}    Email is empty!
    Should Contain    ${email}    @    Email format invalid!
    
    # Verify phone
    ${phone}=    Get Value    css=input[type='tel']
    Should Not Be Empty    ${phone}    Phone is empty!
    
    # Verify address
    ${address}=    Get Value    css=textarea[ng-model='Adress']
    Should Not Be Empty    ${address}    Address is empty!
    
    # Verify gender selected
    ${male_checked}=    Get Element Attribute    css=input[value='Male']    checked
    ${female_checked}=    Get Element Attribute    css=input[value='Female']    checked
    ${any_gender}=    Evaluate    '${male_checked}' == 'true' or '${female_checked}' == 'true'
    Should Be True    ${any_gender}    No gender selected!
    
    # Verify skills selected
    ${skills}=    Get Selected List Label    id=Skills
    Should Not Be Empty    ${skills}    Skills not selected!
    
    # Verify passwords
    ${pass1}=    Get Value    id=firstpassword
    ${pass2}=    Get Value    id=secondpassword
    Should Not Be Empty    ${pass1}    Password is empty!
    Should Be Equal    ${pass1}    ${pass2}    Passwords don't match!
    
    Log    All required fields verification passed!
    Capture Page Screenshot    10_verification_complete.png

Submit Registration Form
    [Documentation]    Submits the registration form
    
    Log    Submitting registration form
    
    # Scroll to submit button
    Scroll Element Into View    id=submitbtn
    
    # Click submit
    # Click Button    id=submitbtn
    # Note: Commented out to prevent actual submission
    
    Log    Form ready for submission
    Capture Page Screenshot    11_ready_to_submit.png

# ========== UTILITY KEYWORDS ==========

Clear Form Field
    [Documentation]    Clears a form field and enters new value
    [Arguments]    ${locator}    ${new_value}
    
    Clear Element Text    ${locator}
    Input Text    ${locator}    ${new_value}
    
    ${entered_value}=    Get Value    ${locator}
    Should Be Equal    ${entered_value}    ${new_value}

Safe Click Element
    [Documentation]    Safely clicks element with waits
    [Arguments]    ${locator}
    
    Wait Until Element Is Visible    ${locator}    timeout=10s
    Wait Until Element Is Enabled    ${locator}    timeout=10s
    Scroll Element Into View    ${locator}
    Sleep    0.5s
    Click Element    ${locator}

Safe Select Dropdown
    [Documentation]    Safely selects from dropdown with verification
    [Arguments]    ${locator}    ${label}
    
    Wait Until Element Is Visible    ${locator}    timeout=10s
    Scroll Element Into View    ${locator}
    Select From List By Label    ${locator}    ${label}
    
    ${selected}=    Get Selected List Label    ${locator}
    Should Be Equal    ${selected}    ${label}
    
    Log    Selected ${label} from dropdown
```

---

## Running Tests - Command Line Examples

```bash
# Basic execution
robot registration_test.robot

# Run with specific tags
robot --include smoke registration_test.robot
robot --include regression registration_test.robot

# Run excluding certain tags
robot --exclude slow registration_test.robot

# Run specific test case
robot --test "Complete Registration Form Test" registration_test.robot

# Generate reports in specific directory
robot --outputdir results registration_test.robot

# Run in headless mode (without GUI)
robot --variable BROWSER:headlesschrome registration_test.robot

# Run with different browser
robot --variable BROWSER:firefox registration_test.robot

# Run with custom variables
robot --variable FIRST_NAME:Jane --variable LAST_NAME:Smith registration_test.robot

# Parallel execution (requires pabot)
pip install robotframework-pabot
pabot --processes 4 registration_test.robot

# Run all .robot files in directory
robot tests/

# Generate detailed log
robot --loglevel DEBUG registration_test.robot

# Continue on failure
robot --exitonfailure registration_test.robot

# Run with retry on failure
robot --rerunfailed failed.xml --output rerun.xml registration_test.robot
```

---

## Best Practices Summary

### 1. **Locator Strategy Priority**
```
1. ID (most reliable)
2. Name
3. CSS Selector (fast)
4. XPath (flexible but slower)
```

### 2. **Always Use Waits**
```robot
# Good
Wait Until Element Is Visible    id=button    timeout=10s
Click Element    id=button

# Bad  
Sleep    5s
Click Element    id=button
```

### 3. **Create Reusable Keywords**
```robot
# Instead of repeating code, create keywords
*** Keywords ***
Login As User
    [Arguments]    ${username}    ${password}
    Input Text    id=username    ${username}
    Input Text    id=password    ${password}
    Click Button    id=loginBtn
```

###4. **Use Variables for Test Data**
```robot
# Good - easy to maintain
${URL}    https://example.com
${USER}   testuser

# Bad - hardcoded everywhere
Input Text    id=url    https://example.com
```

### 5. **Add Documentation and Tags**
```robot
*** Test Cases ***
My Test
    [Documentation]    This test checks login functionality
    [Tags]    smoke    login    regression
    # Test steps here
```

### 6. **Handle Errors Gracefully**
```robot
# Check before acting
${exists}=    Run Keyword And Return Status    Element Should Be Visible    id=popup
Run Keyword If    ${exists}    Click Element    id=popup
```

### 7. **Take Screenshots for Evidence**
```robot
# In teardown
Test Teardown    Run Keyword If Test Failed    Capture Page Screenshot
```

---

## Troubleshooting Common Issues

### Issue 1: Element Not Found
```robot
# Solution: Add waits
Wait Until Element Is Visible    id=element    timeout=10s
```

### Issue 2: Element Not Clickable
```robot
# Solution: Scroll into view
Scroll Element Into View    id=element
Wait Until Element Is Enabled    id=element    timeout=10s
Click Element    id=element
```

### Issue 3: Dropdown Not Selecting
```robot
# Solution: Wait for dropdown to populate
Sleep    1s    # Wait for options to load
Select From List By Label    id=dropdown    Option1
```

### Issue 4: Stale Element
```robot
# Solution: Re-locate element
Wait Until Keyword Succeeds    10s    1s    Click Element    id=element
```

---

## Next Steps

1. **Practice on Real Websites**
   - https://testautomationpractice.blogspot.com/
   - https://demo.automationtesting.in/Register.html
   - https://www.saucedemo.com/

2. **Learn More Libraries**
   - RequestsLibrary (API testing)
   - DatabaseLibrary (Database testing)
   - AppiumLibrary (Mobile testing)

3. **Implement Page Object Model**
   - Separate page logic from tests
   - Better maintainability

4. **Set Up CI/CD Integration**
   - Jenkins, GitHub Actions
   - Automated test execution

---

## Useful Resources

- **Official Documentation**: https://robotframework.org/
- **SeleniumLibrary Docs**: https://robotframework.org/SeleniumLibrary/
- **Robot Framework User Guide**: https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html
- **Practice Sites**: 
  - https://testautomationpractice.blogspot.com/
  - https://www.saucedemo.com/
  - https://demo.automationtesting.in/

---

**END OF COMPLETE BEGINNER TO ADVANCED GUIDE**

🎉 You now have a comprehensive, copy-paste ready guide to Robot Framework!
Each section is fully independent and can be copied and run separately.
All code includes detailed comments explaining what each line does.
Practice with the provided URLs and gradually build your own test suites.

Good luck with your test automation journey! 🚀
