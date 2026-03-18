# Robot Framework - Complete Guide with Real-Time Examples

## Table of Contents
1. [Introduction & Installation](#1-introduction--installation)
2. [Test Case Structure](#2-test-case-structure)
3. [Variables](#3-variables)
4. [Keywords](#4-keywords)
5. [Libraries](#5-libraries)
6. [Control Flow](#6-control-flow)
7. [Built-in Keywords](#7-built-in-keywords)
8. [Test Setup & Teardown](#8-test-setup--teardown)
9. [Data-Driven Testing](#9-data-driven-testing)
10. [Resource Files](#10-resource-files)
11. [Tags](#11-tags)
12. [Reporting](#12-reporting)

---

## 1. Introduction & Installation

### What is Robot Framework?

**Robot Framework** is a generic, open-source automation framework for:
- Test Automation (Web, API, Mobile, Desktop)
- Robotic Process Automation (RPA)
- Acceptance Test-Driven Development (ATDD)

### Key Features:
- **Keyword-driven**: Tests written in human-readable format
- **Tabular syntax**: Easy to read and write
- **Extensible**: Supports Python and Java libraries
- **Rich ecosystem**: SeleniumLibrary, RequestsLibrary, DatabaseLibrary, etc.

### Installation

```bash
# Install Robot Framework
pip install robotframework

# Verify installation
robot --version

# Install popular libraries
pip install robotframework-seleniumlibrary      # Web testing
pip install robotframework-requests             # API testing
pip install robotframework-databaselibrary      # Database testing
pip install robotframework-sshlibrary           # SSH automation
```

### Running Tests

```bash
# Run a test file
robot test_example.robot

# Run with specific tags
robot --include smoke test_example.robot

# Run and generate custom report
robot --outputdir results --name "API Tests" test_example.robot

# Run with variable override
robot --variable BROWSER:chrome test_example.robot
```

---

## 2. Test Case Structure

### Basic Test Case Anatomy

```robot
*** Settings ***
Documentation     This section contains metadata and imports
Library           SeleniumLibrary
Resource          resources/common.robot
Suite Setup       Open Browser To Login Page
Suite Teardown    Close All Browsers

*** Variables ***
${URL}            https://example.com
${BROWSER}        chrome
${USERNAME}       testuser@example.com
${PASSWORD}       SecurePass123

*** Test Cases ***
User Login Test
    [Documentation]    Verify user can login with valid credentials
    [Tags]    login    smoke
    [Setup]    Clear Browser Cache
    [Teardown]    Capture Screenshot On Failure
    
    Go To    ${URL}
    Input Text    id=username    ${USERNAME}
    Input Text    id=password    ${PASSWORD}
    Click Button    xpath=//button[@type='submit']
    Wait Until Page Contains    Dashboard    timeout=5s
    Page Should Contain    Welcome ${USERNAME}

*** Keywords ***
Clear Browser Cache
    Execute Javascript    window.localStorage.clear()
    Execute Javascript    window.sessionStorage.clear()
```

### Sections Explained:

#### 1. ***** Settings *****
- Import libraries, resources
- Define suite/test setup/teardown
- Set test timeout
- Add documentation and metadata

#### 2. ***** Variables *****
- Store reusable data
- Can be scalar, list, or dictionary

#### 3. ***** Test Cases *****
- Actual test scenarios
- Each test case is a block of keywords

#### 4. ***** Keywords *****
- Custom reusable keywords
- Built from built-in or library keywords

---

## 3. Variables

### Types of Variables

#### A. Scalar Variables (Single Value)

```robot
*** Variables ***
${STRING_VAR}        Hello World
${NUMBER_VAR}        42
${FLOAT_VAR}         3.14
${BOOL_VAR}          ${True}
${NONE_VAR}          ${None}

*** Test Cases ***
Scalar Variable Example
    Log    ${STRING_VAR}              # Output: Hello World
    Log    ${NUMBER_VAR}               # Output: 42
    Should Be Equal As Numbers    ${NUMBER_VAR}    42
```

**Real-Time Example: E-commerce Login**
```robot
*** Variables ***
${LOGIN_URL}          https://shop.example.com/login
${VALID_EMAIL}        customer@test.com
${VALID_PASSWORD}     Test@12345
${TIMEOUT}            10s

*** Test Cases ***
Valid User Login
    Open Browser    ${LOGIN_URL}    chrome
    Input Text    id=email    ${VALID_EMAIL}
    Input Text    id=password    ${VALID_PASSWORD}
    Click Button    Login
    Wait Until Page Contains    My Account    ${TIMEOUT}
    Close Browser
```

---

#### B. List Variables (Multiple Values)

```robot
*** Variables ***
@{BROWSERS}           chrome    firefox    edge
@{VALID_USERS}        user1@test.com    user2@test.com    user3@test.com
@{SEARCH_TERMS}       laptop    smartphone    headphones

*** Test Cases ***
Test With List Variables
    Log Many    @{BROWSERS}                    # Logs: chrome, firefox, edge
    Log    ${BROWSERS}[0]                      # Logs: chrome (first item)
    Log    ${BROWSERS}[-1]                     # Logs: edge (last item)
    ${count}=    Get Length    ${BROWSERS}     # Returns: 3
    Log    ${count}
```

**Real-Time Example: Multi-Browser Testing**
```robot
*** Variables ***
@{BROWSERS}           chrome    firefox    edge
@{TEST_URLS}          https://google.com    https://amazon.com    https://github.com

*** Test Cases ***
Test Homepage On Multiple Browsers
    FOR    ${browser}    IN    @{BROWSERS}
        Open Browser    https://example.com    ${browser}
        Title Should Be    Example Domain
        Close Browser
    END

Test Multiple URLs
    Open Browser    about:blank    chrome
    FOR    ${url}    IN    @{TEST_URLS}
        Go To    ${url}
        Log    Testing: ${url}
        Capture Page Screenshot
    END
    Close Browser
```

---

#### C. Dictionary Variables (Key-Value Pairs)

```robot
*** Variables ***
&{USER1}              username=john.doe    password=Pass123    role=admin
&{USER2}              username=jane.smith  password=Test456    role=user
&{API_HEADERS}        Content-Type=application/json    Authorization=Bearer token123

*** Test Cases ***
Dictionary Variable Example
    Log    ${USER1.username}           # Output: john.doe
    Log    ${USER1}[password]          # Output: Pass123
    Log Many    &{USER1}                # Logs all key-value pairs
    
    Should Be Equal    ${USER1.role}    admin
```

**Real-Time Example: API Testing with Different Users**
```robot
*** Settings ***
Library    RequestsLibrary

*** Variables ***
&{ADMIN_USER}         username=admin@company.com    password=Admin@123    role=admin
&{REGULAR_USER}       username=user@company.com     password=User@123     role=user
&{API_CONFIG}         base_url=https://api.example.com    timeout=30    verify=${True}

*** Test Cases ***
Admin Can Access Admin Panel
    Create Session    api    ${API_CONFIG.base_url}    verify=${API_CONFIG.verify}
    ${auth}=    Create List    ${ADMIN_USER.username}    ${ADMIN_USER.password}
    
    ${response}=    GET On Session    api    /admin/dashboard    auth=${auth}
    Should Be Equal As Numbers    ${response.status_code}    200
    Should Contain    ${response.text}    Admin Dashboard

Regular User Cannot Access Admin Panel
    Create Session    api    ${API_CONFIG.base_url}
    ${auth}=    Create List    ${REGULAR_USER.username}    ${REGULAR_USER.password}
    
    ${response}=    GET On Session    api    /admin/dashboard    auth=${auth}    expected_status=403
    Should Be Equal As Numbers    ${response.status_code}    403
```

---

#### D. Environment Variables

```robot
*** Variables ***
${DB_HOST}            %{DATABASE_HOST=localhost}    # Default: localhost
${DB_PORT}            %{DATABASE_PORT=5432}
${API_KEY}            %{API_KEY}                    # Required env var

*** Test Cases ***
Use Environment Variables
    Log    Database Host: ${DB_HOST}
    Log    API Key: ${API_KEY}
```

**Real-Time Example: Multi-Environment Configuration**
```robot
*** Variables ***
${ENV}                %{ENVIRONMENT=dev}                      # dev, staging, prod
${BASE_URL}           %{BASE_URL=https://dev.example.com}
${DB_CONNECTION}      %{DB_URL=postgresql://localhost:5432/testdb}

*** Test Cases ***
Test On Current Environment
    Log    Running tests on: ${ENV}
    Open Browser    ${BASE_URL}    chrome
    Title Should Contain    Example - ${ENV}
    Close Browser
```

**Running with environment variables:**
```bash
# Linux/Mac
export ENVIRONMENT=staging
export BASE_URL=https://staging.example.com
robot tests.robot

# Or inline
ENVIRONMENT=prod BASE_URL=https://example.com robot tests.robot

# Windows
set ENVIRONMENT=staging
robot tests.robot
```

---

#### E. Variable Assignment During Test

```robot
*** Test Cases ***
Dynamic Variable Assignment
    ${current_time}=    Get Time
    Log    Current time: ${current_time}
    
    ${random_num}=    Evaluate    random.randint(1, 100)
    Log    Random number: ${random_num}
    
    ${page_title}=    Get Title
    Log    Page title: ${page_title}
    
    ${element_text}=    Get Text    xpath=//h1
    Should Contain    ${element_text}    Welcome
```

**Real-Time Example: Shopping Cart Test**
```robot
*** Settings ***
Library    SeleniumLibrary

*** Variables ***
${PRODUCT_URL}        https://shop.example.com/product/12345

*** Test Cases ***
Add Product To Cart And Verify
    Open Browser    ${PRODUCT_URL}    chrome
    
    # Get product details before adding to cart
    ${product_name}=    Get Text    css=.product-title
    ${product_price}=   Get Text    css=.product-price
    
    Log    Adding ${product_name} for ${product_price} to cart
    
    # Add to cart
    Click Button    Add to Cart
    
    # Navigate to cart
    Click Link    View Cart
    
    # Verify product in cart
    ${cart_product_name}=    Get Text    css=.cart-item-name
    ${cart_product_price}=   Get Text    css=.cart-item-price
    
    Should Be Equal    ${cart_product_name}    ${product_name}
    Should Be Equal    ${cart_product_price}   ${product_price}
    
    Close Browser
```

---

#### F. Variable Scopes

```robot
*** Variables ***
${SUITE_VAR}          Suite Level Variable

*** Test Cases ***
Test Case Variables
    ${test_var}=    Set Variable    Test Level Variable
    Log    ${test_var}
    Log    ${SUITE_VAR}
    Custom Keyword With Variables

*** Keywords ***
Custom Keyword With Variables
    ${keyword_var}=    Set Variable    Keyword Level Variable
    Log    ${keyword_var}
    Log    ${SUITE_VAR}
    
    # Set suite variable (accessible everywhere)
    Set Suite Variable    ${SUITE_VAR}    Modified Suite Variable
    
    # Set test variable (accessible in current test)
    Set Test Variable    ${NEW_TEST_VAR}    New Test Variable
    
    # Set global variable (accessible everywhere, all suites)
    Set Global Variable    ${GLOBAL_VAR}    Global Variable
```

**Real-Time Example: Session Management**
```robot
*** Variables ***
${SESSION_TOKEN}      ${EMPTY}

*** Test Cases ***
Login And Store Session
    # Login and get token
    ${token}=    User Logs In    admin@test.com    Admin@123
    Set Suite Variable    ${SESSION_TOKEN}    ${token}
    Log    Session token stored: ${SESSION_TOKEN}

Use Session Token For API Calls
    # Use the token from previous test
    Should Not Be Empty    ${SESSION_TOKEN}
    ${response}=    Make Authenticated Request    /api/user/profile    ${SESSION_TOKEN}
    Should Be Equal As Numbers    ${response.status_code}    200

*** Keywords ***
User Logs In
    [Arguments]    ${email}    ${password}
    ${response}=    POST Request    /api/login    {"email":"${email}","password":"${password}"}
    ${token}=    Get From Dictionary    ${response.json()}    token
    [Return]    ${token}
```

---

## 4. Keywords

### Types of Keywords

#### A. Built-in Keywords (BuiltIn Library)

Always available without importing. Includes: Log, Should Be Equal, Set Variable, etc.

```robot
*** Test Cases ***
Built-in Keywords Example
    Log    This is a log message                    # Console output
    Log To Console    This appears in console       # Direct console
    
    ${result}=    Set Variable    Hello World
    Should Be Equal    ${result}    Hello World
    
    Should Be True    5 > 3
    Should Contain    Hello World    World
    
    Sleep    2s                                      # Wait 2 seconds
```

---

#### B. Library Keywords

From imported libraries like SeleniumLibrary, RequestsLibrary, etc.

```robot
*** Settings ***
Library    SeleniumLibrary
Library    String

*** Test Cases ***
Library Keywords Example
    # SeleniumLibrary keywords
    Open Browser    https://google.com    chrome
    Input Text    name=q    Robot Framework
    Press Keys    name=q    ENTER
    Close Browser
    
    # String library keywords
    ${upper}=    Convert To Uppercase    hello
    ${lower}=    Convert To Lowercase    WORLD
    ${replaced}=    Replace String    Hello World    World    Robot
```

---

#### C. User-defined Keywords

Custom keywords created by you.

```robot
*** Keywords ***
Login To Application
    [Documentation]    Custom keyword to login
    [Arguments]    ${username}    ${password}
    
    Open Browser    https://app.example.com/login    chrome
    Input Text    id=username    ${username}
    Input Text    id=password    ${password}
    Click Button    Login
    Wait Until Page Contains    Dashboard

*** Test Cases ***
User Can Login
    Login To Application    testuser@example.com    Pass123
    Close Browser
```

**Real-Time Example: Reusable E-commerce Keywords**
```robot
*** Settings ***
Library    SeleniumLibrary

*** Variables ***
${SHOP_URL}           https://shop.example.com
${BROWSER}            chrome

*** Test Cases ***
Complete Purchase Flow
    Open Shop Website
    Search For Product    laptop
    Add First Product To Cart
    Proceed To Checkout
    Fill Shipping Information    John Doe    123 Main St    New York    10001
    Complete Payment    4111111111111111    12/25    123
    Verify Order Confirmation
    [Teardown]    Close Browser

*** Keywords ***
Open Shop Website
    [Documentation]    Opens the e-commerce website
    Open Browser    ${SHOP_URL}    ${BROWSER}
    Maximize Browser Window
    Wait Until Page Contains Element    css=.header-logo    timeout=10s

Search For Product
    [Arguments]    ${search_term}
    [Documentation]    Searches for a product using search bar
    Input Text    id=searchBox    ${search_term}
    Click Button    css=button.search-btn
    Wait Until Page Contains Element    css=.product-list    timeout=10s
    Page Should Contain    Search Results

Add First Product To Cart
    [Documentation]    Adds the first product from search results to cart
    ${first_product}=    Get Text    css=.product-list .product-item:first-child .product-name
    Log    Adding product: ${first_product}
    Click Element    css=.product-list .product-item:first-child .add-to-cart-btn
    Wait Until Page Contains    Added to Cart    timeout=5s

Proceed To Checkout
    [Documentation]    Navigates to checkout page
    Click Element    css=.cart-icon
    Wait Until Page Contains Element    css=.cart-items    timeout=5s
    Click Button    Proceed to Checkout
    Wait Until Page Contains    Shipping Information    timeout=10s

Fill Shipping Information
    [Arguments]    ${name}    ${address}    ${city}    ${zipcode}
    [Documentation]    Fills shipping form with provided details
    Input Text    id=fullName    ${name}
    Input Text    id=address    ${address}
    Input Text    id=city    ${city}
    Input Text    id=zipCode    ${zipcode}
    Click Button    Continue to Payment

Complete Payment
    [Arguments]    ${card_number}    ${expiry}    ${cvv}
    [Documentation]    Completes payment with credit card details
    Input Text    id=cardNumber    ${card_number}
    Input Text    id=expiryDate    ${expiry}
    Input Text    id=cvv    ${cvv}
    Click Button    Place Order
    Wait Until Page Contains    Order Confirmation    timeout=15s

Verify Order Confirmation
    [Documentation]    Verifies order was placed successfully
    Page Should Contain    Thank you for your order
    ${order_number}=    Get Text    css=.order-number
    Log    Order Number: ${order_number}
    Set Suite Variable    ${ORDER_NUMBER}    ${order_number}
    Capture Page Screenshot    order_confirmation.png
```

---

#### D. Keywords with Arguments

```robot
*** Keywords ***
Add Two Numbers
    [Arguments]    ${num1}    ${num2}
    ${sum}=    Evaluate    ${num1} + ${num2}
    [Return]    ${sum}

Login With Credentials
    [Arguments]    ${username}    ${password}    ${remember_me}=${False}
    Input Text    id=username    ${username}
    Input Text    id=password    ${password}
    Run Keyword If    ${remember_me}    Select Checkbox    id=rememberMe
    Click Button    Login

*** Test Cases ***
Test With Arguments
    ${result}=    Add Two Numbers    10    20
    Should Be Equal As Numbers    ${result}    30
    
    Login With Credentials    user@test.com    Pass123                    # remember_me=False
    Login With Credentials    user@test.com    Pass123    remember_me=${True}
```

---

#### E. Keywords with Different Return Types

```robot
*** Keywords ***
Return Single Value
    [Return]    Hello World

Return Multiple Values
    [Return]    First    Second    Third

Return Dictionary
    &{user}=    Create Dictionary    name=John    age=30    city=NYC
    [Return]    &{user}

*** Test Cases ***
Test Return Values
    ${value}=    Return Single Value
    Log    ${value}
    
    ${first}    ${second}    ${third}=    Return Multiple Values
    Log Many    ${first}    ${second}    ${third}
    
    &{user_data}=    Return Dictionary
    Log    Name: ${user_data.name}, Age: ${user_data.age}
```

**Real-Time Example: API Response Parsing**
```robot
*** Settings ***
Library    RequestsLibrary
Library    Collections

*** Keywords ***
Get User Profile
    [Arguments]    ${user_id}
    [Documentation]    Fetches user profile and returns parsed data
    
    Create Session    api    https://api.example.com
    ${response}=    GET On Session    api    /users/${user_id}
    
    ${name}=    Get From Dictionary    ${response.json()}    name
    ${email}=    Get From Dictionary    ${response.json()}    email
    ${age}=    Get From Dictionary    ${response.json()}    age
    
    [Return]    ${name}    ${email}    ${age}

Get All Users
    [Documentation]    Returns list of all users
    Create Session    api    https://api.example.com
    ${response}=    GET On Session    api    /users
    ${users}=    Set Variable    ${response.json()}
    [Return]    ${users}

*** Test Cases ***
Verify User Profile
    ${name}    ${email}    ${age}=    Get User Profile    12345
    Log    User: ${name}, Email: ${email}, Age: ${age}
    Should Contain    ${email}    @
    Should Be True    ${age} >= 18

Process Multiple Users
    @{all_users}=    Get All Users
    ${user_count}=    Get Length    ${all_users}
    Log    Total users: ${user_count}
    
    FOR    ${user}    IN    @{all_users}
        Log    Processing user: ${user}[name]
        Should Not Be Empty    ${user}[email]
    END
```

---

## 5. Libraries

### A. BuiltIn Library (Default)

Always available, no import needed.

```robot
*** Test Cases ***
BuiltIn Library Examples
    # String operations
    ${text}=    Set Variable    Hello World
    Should Contain    ${text}    World
    Should Start With    ${text}    Hello
    Should End With    ${text}    World
    
    # Number operations
    Should Be Equal As Numbers    10    10
    Should Be True    5 > 3
    
    # List operations
    @{list}=    Create List    apple    banana    cherry
    ${length}=    Get Length    ${list}
    Should Be Equal As Numbers    ${length}    3
    
    # Dictionary operations
    &{dict}=    Create Dictionary    key1=value1    key2=value2
    Dictionary Should Contain Key    ${dict}    key1
    
    # Control flow
    Run Keyword If    ${True}    Log    Condition is true
    Run Keyword Unless    ${False}    Log    Condition is false
```

---

### B. SeleniumLibrary (Web Testing)

```robot
*** Settings ***
Library    SeleniumLibrary

*** Test Cases ***
Web Testing Example
    # Browser operations
    Open Browser    https://example.com    chrome
    Maximize Browser Window
    Set Window Size    1920    1080
    
    # Navigation
    Go To    https://another-site.com
    Go Back
    Reload Page
    
    # Element interaction
    Click Element    id=submitBtn
    Click Link    Sign Up
    Input Text    name=username    testuser
    Input Password    id=password    secret123
    Select From List By Label    id=country    United States
    Select Checkbox    id=terms
    
    # Waits
    Wait Until Page Contains    Welcome    timeout=10s
    Wait Until Element Is Visible    css=.header    timeout=5s
    Wait Until Element Is Enabled    id=submitBtn
    
    # Assertions
    Page Should Contain    Dashboard
    Element Should Be Visible    xpath=//button[@type='submit']
    Element Text Should Be    id=title    Welcome
    
    # Capture
    Capture Page Screenshot    page.png
    Capture Element Screenshot    id=logo    logo.png
    
    Close Browser
```

**Real-Time Example: Complete Web Test**
```robot
*** Settings ***
Library    SeleniumLibrary
Suite Setup    Open Browser To Homepage
Suite Teardown    Close All Browsers

*** Variables ***
${URL}                https://demo.opencart.com
${BROWSER}            chrome
${SEARCH_TERM}        MacBook
${EMAIL}              test${RANDOM}@example.com
${PASSWORD}           Test@12345

*** Test Cases ***
User Registration Flow
    [Tags]    registration    smoke
    Navigate To Registration Page
    Fill Registration Form    John    Doe    ${EMAIL}    ${PASSWORD}
    Submit Registration Form
    Verify Registration Success

Product Search And Add To Cart
    [Tags]    search    cart
    Search For Product    ${SEARCH_TERM}
    Verify Search Results    ${SEARCH_TERM}
    Add First Product To Cart
    Verify Product In Cart

Complete Checkout Process
    [Tags]    checkout    critical
    [Setup]    Login As User    ${EMAIL}    ${PASSWORD}
    Navigate To Cart
    Proceed To Checkout
    Fill Billing Details
    Confirm Order
    Verify Order Placed
    [Teardown]    Logout

*** Keywords ***
Open Browser To Homepage
    Open Browser    ${URL}    ${BROWSER}
    Maximize Browser Window
    Set Selenium Speed    0.5s
    Set Selenium Timeout    10s

Navigate To Registration Page
    Click Element    xpath=//a[text()='My Account']
    Click Link    Register

Fill Registration Form
    [Arguments]    ${firstname}    ${lastname}    ${email}    ${password}
    Input Text    id=input-firstname    ${firstname}
    Input Text    id=input-lastname    ${lastname}
    Input Text    id=input-email    ${email}
    Input Text    id=input-telephone    1234567890
    Input Password    id=input-password    ${password}
    Input Password    id=input-confirm    ${password}

Submit Registration Form
    Select Checkbox    name=agree
    Click Button    xpath=//input[@value='Continue']

Verify Registration Success
    Wait Until Page Contains    Your Account Has Been Created    timeout=10s
    Page Should Contain    Congratulations

Search For Product
    [Arguments]    ${product_name}
    Input Text    name=search    ${product_name}
    Click Button    css=.btn-default.btn-lg

Verify Search Results
    [Arguments]    ${expected_text}
    Wait Until Page Contains Element    css=.product-layout    timeout=10s
    Page Should Contain    ${expected_text}

Add First Product To Cart
    Click Element    xpath=(//button[@data-original-title='Add to Cart'])[1]
    Wait Until Page Contains    Success    timeout=5s

Verify Product In Cart
    Click Element    id=cart
    Wait Until Element Is Visible    css=.table-responsive    timeout=5s
    Page Should Contain    ${SEARCH_TERM}
```

---

### C. RequestsLibrary (API Testing)

```robot
*** Settings ***
Library    RequestsLibrary
Library    Collections

*** Variables ***
${BASE_URL}           https://jsonplaceholder.typicode.com

*** Test Cases ***
GET Request Example
    Create Session    api    ${BASE_URL}
    ${response}=    GET On Session    api    /posts/1
    Should Be Equal As Numbers    ${response.status_code}    200
    ${body}=    Set Variable    ${response.json()}
    Log    ${body}
    Dictionary Should Contain Key    ${body}    title

POST Request Example
    Create Session    api    ${BASE_URL}
    &{data}=    Create Dictionary    title=Test Post    body=Test Content    userId=1
    ${response}=    POST On Session    api    /posts    json=${data}
    Should Be Equal As Numbers    ${response.status_code}    201
    ${response_body}=    Set Variable    ${response.json()}
    Should Be Equal    ${response_body}[title]    Test Post

PUT Request Example
    Create Session    api    ${BASE_URL}
    &{data}=    Create Dictionary    title=Updated Title    body=Updated Content    userId=1    id=1
    ${response}=    PUT On Session    api    /posts/1    json=${data}
    Should Be Equal As Numbers    ${response.status_code}    200

DELETE Request Example
    Create Session    api    ${BASE_URL}
    ${response}=    DELETE On Session    api    /posts/1
    Should Be Equal As Numbers    ${response.status_code}    200
```

**Real-Time Example: Complete API Test Suite**
```robot
*** Settings ***
Library    RequestsLibrary
Library    Collections
Library    String

*** Variables ***
${API_URL}            https://reqres.in/api
${VALID_EMAIL}        eve.holt@reqres.in
${VALID_PASSWORD}     cityslicka
${AUTH_TOKEN}         ${EMPTY}

*** Test Cases ***
User Registration API
    [Tags]    api    registration
    &{payload}=    Create Dictionary    email=${VALID_EMAIL}    password=${VALID_PASSWORD}
    
    Create Session    reqres    ${API_URL}
    ${response}=    POST On Session    reqres    /register    json=${payload}
    
    Should Be Equal As Numbers    ${response.status_code}    200
    ${body}=    Set Variable    ${response.json()}
    Dictionary Should Contain Key    ${body}    token
    
    ${token}=    Get From Dictionary    ${body}    token
    Set Suite Variable    ${AUTH_TOKEN}    ${token}
    Log    Auth Token: ${AUTH_TOKEN}

Get User List
    [Tags]    api    users
    Create Session    reqres    ${API_URL}
    ${response}=    GET On Session    reqres    /users?page=2
    
    Should Be Equal As Numbers    ${response.status_code}    200
    ${body}=    Set Variable    ${response.json()}
    
    # Verify response structure
    Dictionary Should Contain Key    ${body}    data
    Dictionary Should Contain Key    ${body}    page
    
    # Verify page number
    ${page}=    Get From Dictionary    ${body}    page
    Should Be Equal As Numbers    ${page}    2
    
    # Verify users list
    ${users}=    Get From Dictionary    ${body}    data
    ${user_count}=    Get Length    ${users}
    Should Be True    ${user_count} > 0
    
    # Log first user
    ${first_user}=    Get From List    ${users}    0
    Log    First User: ${first_user}[email]

Get Single User
    [Tags]    api    users
    Create Session    reqres    ${API_URL}
    ${user_id}=    Set Variable    2
    
    ${response}=    GET On Session    reqres    /users/${user_id}
    Should Be Equal As Numbers    ${response.status_code}    200
    
    ${body}=    Set Variable    ${response.json()}
    ${user_data}=    Get From Dictionary    ${body}    data
    
    # Verify user data structure
    Dictionary Should Contain Key    ${user_data}    id
    Dictionary Should Contain Key    ${user_data}    email
    Dictionary Should Contain Key    ${user_data}    first_name
    Dictionary Should Contain Key    ${user_data}    last_name
    
    # Verify user ID
    ${id}=    Get From Dictionary    ${user_data}    id
    Should Be Equal As Numbers    ${id}    ${user_id}

Create New User
    [Tags]    api    crud
    &{user_payload}=    Create Dictionary
    ...    name=John Doe
    ...    job=Software Engineer
    
    Create Session    reqres    ${API_URL}
    ${response}=    POST On Session    reqres    /users    json=${user_payload}
    
    Should Be Equal As Numbers    ${response.status_code}    201
    ${body}=    Set Variable    ${response.json()}
    
    # Verify created user
    Should Be Equal    ${body}[name]    John Doe
    Should Be Equal    ${body}[job]    Software Engineer
    Dictionary Should Contain Key    ${body}    id
    Dictionary Should Contain Key    ${body}    createdAt
    
    ${new_user_id}=    Get From Dictionary    ${body}    id
    Set Test Variable    ${NEW_USER_ID}    ${new_user_id}
    Log    Created User ID: ${NEW_USER_ID}

Update User
    [Tags]    api    crud
    &{update_payload}=    Create Dictionary
    ...    name=John Doe Updated
    ...    job=Senior Engineer
    
    Create Session    reqres    ${API_URL}
    ${response}=    PUT On Session    reqres    /users/2    json=${update_payload}
    
    Should Be Equal As Numbers    ${response.status_code}    200
    ${body}=    Set Variable    ${response.json()}
    
    Should Be Equal    ${body}[name]    John Doe Updated
    Should Be Equal    ${body}[job]    Senior Engineer
    Dictionary Should Contain Key    ${body}    updatedAt

Delete User
    [Tags]    api    crud
    Create Session    reqres    ${API_URL}
    ${response}=    DELETE On Session    reqres    /users/2
    Should Be Equal As Numbers    ${response.status_code}    204

User Not Found
    [Tags]    api    negative
    Create Session    reqres    ${API_URL}
    ${response}=    GET On Session    reqres    /users/999    expected_status=404
    Should Be Equal As Numbers    ${response.status_code}    404

*** Keywords ***
Verify Response Time
    [Arguments]    ${response}    ${max_time}=2
    ${elapsed}=    Evaluate    ${response.elapsed.total_seconds()}
    Should Be True    ${elapsed} < ${max_time}    Response time ${elapsed}s exceeds ${max_time}s
```

---

### D. DatabaseLibrary (Database Testing)

```robot
*** Settings ***
Library    DatabaseLibrary

*** Variables ***
${DB_MODULE}          psycopg2
${DB_NAME}            testdb
${DB_USER}            admin
${DB_PASS}            password
${DB_HOST}            localhost
${DB_PORT}            5432

*** Test Cases ***
Database Operations
    Connect To Database    ${DB_MODULE}    ${DB_NAME}    ${DB_USER}    ${DB_PASS}    ${DB_HOST}    ${DB_PORT}
    
    # Query execution
    ${result}=    Query    SELECT * FROM users WHERE id=1
    Log    ${result}
    
    # Row count
    ${count}=    Row Count    SELECT * FROM users
    Log    Total users: ${count}
    
    # Check if exists
    ${exists}=    Check If Exists In Database    SELECT * FROM users WHERE email='test@example.com'
    Should Be True    ${exists}
    
    # Execute SQL
    Execute Sql String    INSERT INTO users (name, email) VALUES ('John', 'john@test.com')
    
    Disconnect From Database
```

**Real-Time Example: Database Validation**
```robot
*** Settings ***
Library    DatabaseLibrary
Library    Collections

*** Variables ***
${DB_MODULE}          pymysql
${DB_NAME}            ecommerce_db
${DB_USER}            test_user
${DB_PASS}            test_pass
${DB_HOST}            localhost
${DB_PORT}            3306

*** Test Cases ***
Verify User Registration In Database
    [Documentation]    Verify new user is correctly inserted into database
    Connect To Database    ${DB_MODULE}    ${DB_NAME}    ${DB_USER}    ${DB_PASS}    ${DB_HOST}    ${DB_PORT}
    
    ${email}=    Set Variable    newuser@test.com
    
    # Check user doesn't exist
    ${count_before}=    Row Count    SELECT * FROM users WHERE email='${email}'
    Should Be Equal As Numbers    ${count_before}    0
    
    # Insert user (simulating registration)
    Execute Sql String    INSERT INTO users (name, email, password, created_at) VALUES ('New User', '${email}', 'hashed_password', NOW())
    
    # Verify user was inserted
    ${count_after}=    Row Count    SELECT * FROM users WHERE email='${email}'
    Should Be Equal As Numbers    ${count_after}    1
    
    # Verify user details
    @{user_data}=    Query    SELECT name, email, created_at FROM users WHERE email='${email}'
    ${user}=    Get From List    ${user_data}    0
    Should Be Equal    ${user}[0]    New User
    Should Be Equal    ${user}[1]    ${email}
    
    # Cleanup
    Execute Sql String    DELETE FROM users WHERE email='${email}'
    
    Disconnect From Database

Verify Order Details
    Connect To Database    ${DB_MODULE}    ${DB_NAME}    ${DB_USER}    ${DB_PASS}    ${DB_HOST}    ${DB_PORT}
    
    ${order_id}=    Set Variable    12345
    
    # Get order information
    @{order}=    Query    SELECT order_id, user_id, total_amount, status FROM orders WHERE order_id=${order_id}
    Log Many    @{order}
    
    ${order_data}=    Get From List    ${order}    0
    ${total_amount}=    Set Variable    ${order_data}[2]
    ${status}=    Set Variable    ${order_data}[3]
    
    Should Be Equal    ${status}    completed
    Should Be True    ${total_amount} > 0
    
    # Verify order items
    ${item_count}=    Row Count    SELECT * FROM order_items WHERE order_id=${order_id}
    Should Be True    ${item_count} > 0
    
    Disconnect From Database
```

---

### E. Collections Library

```robot
*** Settings ***
Library    Collections

*** Test Cases ***
List Operations
    @{fruits}=    Create List    apple    banana    cherry
    
    Append To List    ${fruits}    orange
    ${length}=    Get Length    ${fruits}
    Should Be Equal As Numbers    ${length}    4
    
    ${first}=    Get From List    ${fruits}    0
    Should Be Equal    ${first}    apple
    
    List Should Contain Value    ${fruits}    banana
    
    Remove From List    ${fruits}    2
    
Dictionary Operations
    &{person}=    Create Dictionary    name=John    age=30    city=NYC
    
    Set To Dictionary    ${person}    country=USA
    
    ${name}=    Get From Dictionary    ${person}    name
    Should Be Equal    ${name}    John
    
    Dictionary Should Contain Key    ${person}    age
    
    ${keys}=    Get Dictionary Keys    ${person}
    ${values}=    Get Dictionary Values    ${person}
    
    Log Many    @{keys}
    Log Many    @{values}
```

---

## 6. Control Flow

### A. FOR Loops

#### Simple FOR Loop
```robot
*** Test Cases ***
Simple FOR Loop
    FOR    ${i}    IN RANGE    5
        Log    Iteration: ${i}
    END

FOR Loop With List
    @{browsers}=    Create List    chrome    firefox    edge
    FOR    ${browser}    IN    @{browsers}
        Log    Testing on: ${browser}
    END

FOR Loop With Index
    @{items}=    Create List    apple    banana    cherry
    FOR    ${index}    ${item}    IN ENUMERATE    @{items}
        Log    Index ${index}: ${item}
    END
```

**Real-Time Example: Multiple Browser Testing**
```robot
*** Settings ***
Library    SeleniumLibrary

*** Variables ***
@{BROWSERS}           chrome    firefox    edge
@{TEST_URLS}          https://google.com    https://github.com    https://stackoverflow.com

*** Test Cases ***
Test Multiple Browsers
    FOR    ${browser}    IN    @{BROWSERS}
        Open Browser    https://example.com    ${browser}
        Title Should Contain    Example
        ${title}=    Get Title
        Log    ${browser} - Page Title: ${title}
        Close Browser
    END

Test Multiple URLs On Chrome
    Open Browser    about:blank    chrome
    FOR    ${url}    IN    @{TEST_URLS}
        Go To    ${url}
        ${title}=    Get Title
        Log    URL: ${url} - Title: ${title}
        Capture Page Screenshot    screenshot_${url}.png
    END
    Close Browser

Test Matrix: Multiple Browsers and URLs
    FOR    ${browser}    IN    @{BROWSERS}
        Open Browser    about:blank    ${browser}
        FOR    ${url}    IN    @{TEST_URLS}
            Go To    ${url}
            ${title}=    Get Title
            Log    [${browser}] ${url} - ${title}
        END
        Close Browser
    END
```

---

#### FOR Loop with EXIT and CONTINUE

```robot
*** Test Cases ***
FOR Loop With EXIT
    FOR    ${i}    IN RANGE    10
        Log    Number: ${i}
        Exit For Loop If    ${i} == 5
    END

FOR Loop With CONTINUE
    FOR    ${i}    IN RANGE    10
        Continue For Loop If    ${i} % 2 == 0    # Skip even numbers
        Log    Odd number: ${i}
    END
```

**Real-Time Example: Find First Available Product**
```robot
*** Settings ***
Library    SeleniumLibrary

*** Test Cases ***
Find And Purchase First Available Product
    Open Browser    https://shop.example.com/products    chrome
    
    ${purchased}=    Set Variable    ${False}
    
    FOR    ${i}    IN RANGE    1    11    # Check first 10 products
        ${xpath}=    Set Variable    (//div[@class='product'])[${i}]
        ${available}=    Run Keyword And Return Status    
        ...    Element Should Be Visible    ${xpath}//span[@class='in-stock']
        
        Run Keyword If    not ${available}    Continue For Loop
        
        # Product is available, purchase it
        Click Element    ${xpath}//button[@class='add-to-cart']
        Wait Until Page Contains    Added to Cart
        ${purchased}=    Set Variable    ${True}
        Exit For Loop
    END
    
    Should Be True    ${purchased}    No products were available
    Close Browser
```

---

### B. IF/ELSE Conditions

```robot
*** Test Cases ***
Simple IF Condition
    ${age}=    Set Variable    25
    IF    ${age} >= 18
        Log    Adult
    ELSE
        Log    Minor
    END

Multiple Conditions
    ${score}=    Set Variable    85
    IF    ${score} >= 90
        Log    Grade: A
    ELSE IF    ${score} >= 80
        Log    Grade: B
    ELSE IF    ${score} >= 70
        Log    Grade: C
    ELSE
        Log    Grade: F
    END

IF With String Comparison
    ${status}=    Set Variable    active
    IF    '${status}' == 'active'
        Log    Status is active
    END
```

**Real-Time Example: Dynamic Test Execution**
```robot
*** Settings ***
Library    SeleniumLibrary

*** Variables ***
${ENVIRONMENT}        staging
${INCLUDE_SLOW_TESTS}    ${False}

*** Test Cases ***
Environment-Specific Login Test
    ${url}=    Get Environment URL    ${ENVIRONMENT}
    ${username}=    Get Environment Username    ${ENVIRONMENT}
    
    Open Browser    ${url}    chrome
    Login With Credentials    ${username}    TestPass123
    Verify Login Success
    Close Browser

Conditional Test Execution
    IF    '${ENVIRONMENT}' == 'production'
        Log    Skipping destructive tests in production
        Pass Execution    Production environment detected
    END
    
    # Destructive tests run only in non-production
    Delete Test Data
    Create Test Orders
    Verify Data Cleanup

Performance Test Based On Flag
    IF    ${INCLUDE_SLOW_TESTS}
        Run Keyword    Execute Load Test
        Run Keyword    Execute Stress Test
    ELSE
        Log    Slow tests skipped
    END

*** Keywords ***
Get Environment URL
    [Arguments]    ${env}
    IF    '${env}' == 'production'
        [Return]    https://app.example.com
    ELSE IF    '${env}' == 'staging'
        [Return]    https://staging.example.com
    ELSE
        [Return]    https://dev.example.com
    END

Get Environment Username
    [Arguments]    ${env}
    IF    '${env}' == 'production'
        [Return]    prod_user@example.com
    ELSE
        [Return]    test_user@example.com
    END
```

---

### C. WHILE Loops

```robot
*** Test Cases ***
Simple WHILE Loop
    ${counter}=    Set Variable    0
    WHILE    ${counter} < 5
        Log    Counter: ${counter}
        ${counter}=    Evaluate    ${counter} + 1
    END

WHILE Loop With Condition
    ${attempts}=    Set Variable    0
    ${max_attempts}=    Set Variable    3
    ${success}=    Set Variable    ${False}
    
    WHILE    ${attempts} < ${max_attempts} and not ${success}
        ${attempts}=    Evaluate    ${attempts} + 1
        ${success}=    Attempt Operation
        IF    not ${success}
            Log    Attempt ${attempts} failed, retrying...
            Sleep    2s
        END
    END
```

**Real-Time Example: Retry Mechanism**
```robot
*** Settings ***
Library    SeleniumLibrary
Library    RequestsLibrary

*** Test Cases ***
Wait For Element With Retry
    Open Browser    https://example.com/dynamic-content    chrome
    
    ${max_wait}=    Set Variable    30
    ${elapsed}=    Set Variable    0
    ${found}=    Set Variable    ${False}
    
    WHILE    ${elapsed} < ${max_wait} and not ${found}
        ${found}=    Run Keyword And Return Status    
        ...    Element Should Be Visible    id=dynamic-element
        
        IF    not ${found}
            Sleep    1s
            ${elapsed}=    Evaluate    ${elapsed} + 1
            Reload Page
        END
    END
    
    Should Be True    ${found}    Element not found within ${max_wait} seconds
    Close Browser

API Retry On Failure
    ${max_retries}=    Set Variable    5
    ${retry_count}=    Set Variable    0
    ${success}=    Set Variable    ${False}
    
    WHILE    ${retry_count} < ${max_retries} and not ${success}
        ${retry_count}=    Evaluate    ${retry_count} + 1
        Log    Attempt ${retry_count} of ${max_retries}
        
        TRY
            Create Session    api    https://api.example.com
            ${response}=    GET On Session    api    /data    timeout=5
            ${success}=    Set Variable    ${True}
            Log    API call successful
        EXCEPT
            Log    API call failed, retrying...
            Sleep    2s
        END
    END
    
    Should Be True    ${success}    API call failed after ${max_retries} attempts
```

---

### D. TRY/EXCEPT (Error Handling)

```robot
*** Test Cases ***
Basic Try-Except
    TRY
        ${result}=    Divide    10    0
    EXCEPT
        Log    Division by zero error handled
    END

Try-Except With Specific Error
    TRY
        Open Browser    invalid_url    chrome
    EXCEPT    WebDriverException
        Log    WebDriver exception occurred
    FINALLY
        Log    Cleanup operations
    END

Try-Except With Variable
    TRY
        Click Element    id=non-existent
    EXCEPT    AS    ${error}
        Log    Error occurred: ${error}
    END
```

**Real-Time Example: Robust Test Execution**
```robot
*** Settings ***
Library    SeleniumLibrary
Library    RequestsLibrary

*** Test Cases ***
Robust Login Test With Error Handling
    [Documentation]    Login test with comprehensive error handling
    
    TRY
        Open Browser    https://example.com    chrome
        Maximize Browser Window
        
        TRY
            Wait Until Element Is Visible    id=username    timeout=5s
        EXCEPT
            Log    Login form not visible, trying refresh
            Reload Page
            Wait Until Element Is Visible    id=username    timeout=5s
        END
        
        Input Text    id=username    testuser@example.com
        Input Password    id=password    TestPass123
        Click Button    Login
        
        TRY
            Wait Until Page Contains    Dashboard    timeout=10s
            Log    Login successful
        EXCEPT
            ${error_msg}=    Get Text    css=.error-message
            Log    Login failed: ${error_msg}
            Capture Page Screenshot    login_failure.png
            Fail    Login unsuccessful: ${error_msg}
        END
        
    EXCEPT    AS    ${error}
        Log    Test failed with error: ${error}
        Capture Page Screenshot    test_error.png
        Fail    Test execution failed
    FINALLY
        Close Browser
    END

API Test With Fallback
    TRY
        Create Session    primary    https://api.example.com
        ${response}=    GET On Session    primary    /users    timeout=5
        Log    Primary API successful
    EXCEPT
        Log    Primary API failed, trying backup
        TRY
            Create Session    backup    https://backup-api.example.com
            ${response}=    GET On Session    backup    /users    timeout=5
            Log    Backup API successful
        EXCEPT    AS    ${backup_error}
            Log    Both APIs failed: ${backup_error}
            Fail    All API endpoints are down
        END
    END
    
    Should Be Equal As Numbers    ${response.status_code}    200
```

---

## 7. Built-in Keywords (Common Ones)

### Logging Keywords

```robot
*** Test Cases ***
Logging Examples
    Log    Simple log message                        # Info level
    Log    Debug message    level=DEBUG               # Debug level
    Log    Warning message    level=WARN              # Warning level
    Log    Error message    ERROR                     # Error level
    
    Log To Console    This appears in console
    Log Many    First    Second    Third              # Log multiple values
    
    Comment    This is a comment in logs
```

---

### String Keywords

```robot
*** Test Cases ***
String Operations
    ${upper}=    Convert To Uppercase    hello world
    Should Be Equal    ${upper}    HELLO WORLD
    
    ${lower}=    Convert To Lowercase    HELLO WORLD
    Should Be Equal    ${lower}    hello world
    
    Should Contain    Robot Framework    Framework
    Should Not Contain    Robot Framework    Java
    
    Should Start With    Robot Framework    Robot
    Should End With    Robot Framework    Framework
    
    ${replaced}=    Replace String    Hello World    World    Robot
    Should Be Equal    ${replaced}    Hello Robot
    
    ${stripped}=    Strip String    ${SPACE}${SPACE}text${SPACE}${SPACE}
    Should Be Equal    ${stripped}    text
```

---

### Number Keywords

```robot
*** Test Cases ***
Number Operations
    Should Be Equal As Numbers    10    10
    Should Not Be Equal As Numbers    10    20
    
    Should Be True    5 > 3
    Should Be True    10 < 20
    Should Be True    5 >= 5
    
    ${num}=    Convert To Integer    42
    ${float}=    Convert To Number    3.14
    
    ${result}=    Evaluate    10 + 20
    Should Be Equal As Numbers    ${result}    30
```

---

### Wait Keywords

```robot
*** Test Cases ***
Wait Examples
    Sleep    2s                                    # Wait 2 seconds
    Sleep    500ms                                 # Wait 500 milliseconds
    
    Wait Until Keyword Succeeds    30s    2s      # Retry every 2s for 30s max
    ...    Element Should Be Visible    id=dynamic-element
```

---

### Run Keyword Variants

```robot
*** Test Cases ***
Conditional Keyword Execution
    Run Keyword If    ${True}    Log    Condition is true
    Run Keyword Unless    ${False}    Log    Condition is not false
    
    ${status}=    Run Keyword And Return Status    Element Should Be Visible    id=element
    Run Keyword If    ${status}    Log    Element found
    
    Run Keyword And Ignore Error    Click Element    id=optional-button
    
    ${result}=    Run Keyword And Continue On Failure    Should Be Equal    1    2
```

---

## 8. Test Setup & Teardown

### Test-Level Setup/Teardown

```robot
*** Test Cases ***
Test With Setup And Teardown
    [Setup]    Log    Starting test
    [Teardown]    Log    Ending test
    
    Log    Test body execution

Test With Keyword Setup/Teardown
    [Setup]    Open Browser    https://example.com    chrome
    [Teardown]    Close Browser
    
    Page Should Contain    Example Domain
```

---

### Suite-Level Setup/Teardown

```robot
*** Settings ***
Suite Setup    Log    Suite starting
Suite Teardown    Log    Suite ending

*** Test Cases ***
First Test
    Log    First test

Second Test
    Log    Second test
```

**Real-Time Example: Database Test Suite**
```robot
*** Settings ***
Library    DatabaseLibrary
Suite Setup    Setup Test Database
Suite Teardown    Cleanup Test Database

*** Variables ***
${DB_MODULE}          pymysql
${DB_NAME}            test_db
${DB_USER}            test_user
${DB_PASS}            test_pass
${DB_HOST}            localhost
${DB_PORT}            3306

*** Test Cases ***
Test User Creation
    [Setup]    Begin Transaction
    [Teardown]    Rollback Transaction
    
    Execute Sql String    INSERT INTO users (name, email) VALUES ('Test User', 'test@example.com')
    ${count}=    Row Count    SELECT * FROM users WHERE email='test@example.com'
    Should Be Equal As Numbers    ${count}    1

Test Order Creation
    [Setup]    Begin Transaction
    [Teardown]    Rollback Transaction
    
    Execute Sql String    INSERT INTO orders (user_id, total) VALUES (1, 100.00)
    ${count}=    Row Count    SELECT * FROM orders WHERE user_id=1
    Should Be True    ${count} >= 1

*** Keywords ***
Setup Test Database
    Connect To Database    ${DB_MODULE}    ${DB_NAME}    ${DB_USER}    ${DB_PASS}    ${DB_HOST}    ${DB_PORT}
    Execute Sql String    CREATE TABLE IF NOT EXISTS users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100), email VARCHAR(100))
    Execute Sql String    CREATE TABLE IF NOT EXISTS orders (id INT AUTO_INCREMENT PRIMARY KEY, user_id INT, total DECIMAL(10,2))
    Log    Test database ready

Cleanup Test Database
    Execute Sql String    DROP TABLE IF EXISTS users
    Execute Sql String    DROP TABLE IF EXISTS orders
    Disconnect From Database
    Log    Test database cleaned up

Begin Transaction
    Execute Sql String    START TRANSACTION

Rollback Transaction
    Execute Sql String    ROLLBACK
```

---

## 9. Data-Driven Testing

### Template-Based Testing

```robot
*** Settings ***
Test Template    Login With Invalid Credentials

*** Test Cases ***                USERNAME              PASSWORD
Empty Username                    ${EMPTY}              validpass
Empty Password                    validuser             ${EMPTY}
Both Empty                        ${EMPTY}              ${EMPTY}
Invalid Username                  invalid@test.com      validpass
Invalid Password                  valid@test.com        wrongpass
Special Characters                admin'OR'1'='1        anything

*** Keywords ***
Login With Invalid Credentials
    [Arguments]    ${username}    ${password}
    Open Browser    https://example.com/login    chrome
    Input Text    id=username    ${username}
    Input Text    id=password    ${password}
    Click Button    Login
    Page Should Contain    Invalid credentials
    Close Browser
```

**Real-Time Example: Product Search Testing**
```robot
*** Settings ***
Library    SeleniumLibrary
Test Template    Search Product And Verify Results

*** Variables ***
${URL}                https://amazon.com

*** Test Cases ***                SEARCH_TERM           EXPECTED_RESULT
Search Laptop                     laptop                Laptop
Search Phone                      iphone                iPhone
Search Book                       python book           Python
Search Electronics                headphones            Headphones
Search With Spaces                smart watch           Smart Watch

*** Keywords ***
Search Product And Verify Results
    [Arguments]    ${search_term}    ${expected}
    [Documentation]    Searches for product and verifies results contain expected term
    
    Open Browser    ${URL}    chrome
    Maximize Browser Window
    
    Input Text    id=twotabsearchtextbox    ${search_term}
    Click Button    id=nav-search-submit-button
    
    Wait Until Page Contains Element    css=.s-main-slot    timeout=10s
    Page Should Contain    ${expected}
    
    ${results}=    Get Element Count    css=.s-main-slot .s-result-item
    Should Be True    ${results} > 0
    
    Capture Page Screenshot    search_${search_term}.png
    
    Close Browser
```

---

### Using Test Data from External Files

**data.csv:**
```csv
username,password,expected_result
valid@test.com,Pass123,success
invalid@test.com,Pass123,failure
valid@test.com,wrongpass,failure
<script>alert(1)</script>,Pass123,failure
```

**test.robot:**
```robot
*** Settings ***
Library    SeleniumLibrary
Library    CSVLibrary

*** Variables ***
${DATA_FILE}          data.csv

*** Test Cases ***
Data-Driven Login Test
    @{data}=    Read CSV File    ${DATA_FILE}
    
    FOR    ${row}    IN    @{data}
        ${username}=    Get From List    ${row}    0
        ${password}=    Get From List    ${row}    1
        ${expected}=    Get From List    ${row}    2
        
        Test Login    ${username}    ${password}    ${expected}
    END

*** Keywords ***
Test Login
    [Arguments]    ${username}    ${password}    ${expected_result}
    
    Open Browser    https://example.com/login    chrome
    Input Text    id=username    ${username}
    Input Text    id=password    ${password}
    Click Button    Login
    
    IF    '${expected_result}' == 'success'
        Page Should Contain    Dashboard
    ELSE
        Page Should Contain    Invalid credentials
    END
    
    Close Browser
```

---

## 10. Resource Files

Resource files contain reusable keywords, variables, and settings.

**resources/common_keywords.robot:**
```robot
*** Settings ***
Library    SeleniumLibrary

*** Variables ***
${BROWSER}            chrome
${TIMEOUT}            10s

*** Keywords ***
Open Application
    [Arguments]    ${url}
    Open Browser    ${url}    ${BROWSER}
    Maximize Browser Window
    Set Selenium Timeout    ${TIMEOUT}

Close Application
    Close All Browsers

Wait For Element
    [Arguments]    ${locator}
    Wait Until Element Is Visible    ${locator}    timeout=${TIMEOUT}

Safe Click
    [Arguments]    ${locator}
    Wait For Element    ${locator}
    Click Element    ${locator}
```

**tests/login_test.robot:**
```robot
*** Settings ***
Resource    ../resources/common_keywords.robot

*** Test Cases ***
User Login Test
    Open Application    https://example.com
    Safe Click    id=loginBtn
    Close Application
```

**Real-Time Example: Page Object Model**

**resources/pages/login_page.robot:**
```robot
*** Settings ***
Library    SeleniumLibrary

*** Variables ***
${LOGIN_URL}              https://example.com/login
${USERNAME_FIELD}         id=username
${PASSWORD_FIELD}         id=password
${LOGIN_BUTTON}           xpath=//button[@type='submit']
${ERROR_MESSAGE}          css=.error-msg

*** Keywords ***
Go To Login Page
    Go To    ${LOGIN_URL}
    Wait Until Page Contains Element    ${USERNAME_FIELD}

Enter Username
    [Arguments]    ${username}
    Input Text    ${USERNAME_FIELD}    ${username}

Enter Password
    [Arguments]    ${password}
    Input Password    ${PASSWORD_FIELD}    ${password}

Click Login Button
    Click Button    ${LOGIN_BUTTON}

Verify Login Success
    Wait Until Page Contains    Dashboard    timeout=10s

Verify Login Failure
    Wait Until Element Is Visible    ${ERROR_MESSAGE}    timeout=5s
    ${error}=    Get Text    ${ERROR_MESSAGE}
    [Return]    ${error}
```

**resources/pages/dashboard_page.robot:**
```robot
*** Settings ***
Library    SeleniumLibrary

*** Variables ***
${WELCOME_MESSAGE}        css=.welcome-msg
${LOGOUT_BUTTON}          id=logoutBtn
${PROFILE_LINK}           xpath=//a[text()='Profile']

*** Keywords ***
Verify Dashboard Loaded
    Wait Until Page Contains Element    ${WELCOME_MESSAGE}    timeout=10s

Get Welcome Message
    ${message}=    Get Text    ${WELCOME_MESSAGE}
    [Return]    ${message}

Click Profile
    Click Element    ${PROFILE_LINK}

Logout
    Click Button    ${LOGOUT_BUTTON}
    Wait Until Page Contains Element    id=username    timeout=5s
```

**tests/complete_flow_test.robot:**
```robot
*** Settings ***
Library    SeleniumLibrary
Resource    ../resources/pages/login_page.robot
Resource    ../resources/pages/dashboard_page.robot

*** Variables ***
${BROWSER}                chrome
${VALID_USER}             testuser@example.com
${VALID_PASS}             Test@12345

*** Test Cases ***
Complete User Flow
    [Setup]    Open Browser    ${LOGIN_URL}    ${BROWSER}
    [Teardown]    Close Browser
    
    # Login
    Go To Login Page
    Enter Username    ${VALID_USER}
    Enter Password    ${VALID_PASS}
    Click Login Button
    Verify Login Success
    
    # Dashboard
    Verify Dashboard Loaded
    ${welcome}=    Get Welcome Message
    Should Contain    ${welcome}    ${VALID_USER}
    
    # Profile
    Click Profile
    Wait Until Page Contains    Profile Information
    
    # Logout
    Logout
```

---

## 11. Tags

Tags are used to categorize and selectively run tests.

```robot
*** Test Cases ***
Login Test
    [Tags]    smoke    login    critical
    # Test steps

Registration Test
    [Tags]    regression    user-management
    # Test steps

Admin Dashboard Test
    [Tags]    admin    slow
    # Test steps

Payment Test
    [Tags]    smoke    critical    payment
    # Test steps
```

### Running Tests with Tags

```bash
# Run tests with specific tag
robot --include smoke tests.robot

# Run tests with multiple tags (OR condition)
robot --include smokeORcritical tests.robot

# Run tests with multiple tags (AND condition)
robot --include smokeANDcritical tests.robot

# Exclude tests with specific tag
robot --exclude slow tests.robot

# Combine include and exclude
robot --include smoke --exclude admin tests.robot

# Complex tag expressions
robot --include "smokeANDcritical" --exclude "adminORslow" tests.robot
```

**Real-Time Example: Tagged Test Suite**
```robot
*** Settings ***
Library    SeleniumLibrary

*** Variables ***
${URL}                https://example.com
${BROWSER}            chrome

*** Test Cases ***
Homepage Load Test
    [Tags]    smoke    fast    ui
    [Documentation]    Verify homepage loads correctly
    Open Browser    ${URL}    ${BROWSER}
    Title Should Contain    Example
    Close Browser

User Registration
    [Tags]    regression    user-management    medium
    [Documentation]    Complete user registration flow
    Open Browser    ${URL}/register    ${BROWSER}
    Fill Registration Form
    Submit Form
    Verify Registration Success
    Close Browser

User Login
    [Tags]    smoke    critical    login    fast
    [Documentation]    User login with valid credentials
    Open Browser    ${URL}/login    ${BROWSER}
    Login With Valid Credentials
    Verify Dashboard Loaded
    Close Browser

Password Reset
    [Tags]    regression    user-management    medium
    [Documentation]    Password reset functionality
    Open Browser    ${URL}/forgot-password    ${BROWSER}
    Request Password Reset
    Verify Email Sent
    Close Browser

Admin Panel Access
    [Tags]    admin    critical    slow
    [Documentation]    Admin can access admin panel
    Open Browser    ${URL}/admin    ${BROWSER}
    Login As Admin
    Verify Admin Dashboard
    Close Browser

Load Test
    [Tags]    performance    slow    manual
    [Documentation]    Load testing with multiple users
    Run Load Test
    Analyze Results

Database Migration Test
    [Tags]    database    slow    deployment
    [Documentation]    Verify database migration
    Run Migration Scripts
    Verify Database Schema
    Verify Data Integrity

API Health Check
    [Tags]    smoke    api    fast    critical
    [Documentation]    Verify API is responding
    ${response}=    GET Request    /api/health
    Should Be Equal As Numbers    ${response.status_code}    200
```

### Running Strategy:
```bash
# Smoke tests (quick sanity check)
robot --include smoke tests.robot

# Critical tests only
robot --include critical tests.robot

# All tests except slow ones
robot --exclude slow tests.robot

# Regression suite (all except smoke and performance)
robot --exclude smokeORperformance tests.robot

# UI tests only
robot --include ui tests.robot

# Fast smoke and critical tests
robot --include "smokeORcritical" --include fast tests.robot
```

---

## 12. Reporting

Robot Framework generates three reports after execution:

1. **log.html** - Detailed execution log
2. **report.html** - High-level test report
3. **output.xml** - Machine-readable XML

### Custom Report Configuration

```bash
# Custom output directory
robot --outputdir results tests.robot

# Custom report name
robot --name "Smoke Tests" tests.robot

# Custom file names
robot --output custom_output.xml --log custom_log.html --report custom_report.html tests.robot

# Set log level
robot --loglevel DEBUG tests.robot

# Disable specific reports
robot --log NONE tests.robot                # Disable log.html
robot --report NONE tests.robot             # Disable report.html

# Timestamp in results
robot --timestampoutputs tests.robot        # Adds timestamp to output files
```

### Screenshots and Logging

```robot
*** Settings ***
Library    SeleniumLibrary

*** Test Cases ***
Test With Screenshots
    Open Browser    https://example.com    chrome
    
    # Manual screenshot
    Capture Page Screenshot    homepage.png
    
    # Screenshot on failure (in teardown)
    [Teardown]    Run Keyword If Test Failed    Capture Page Screenshot

Test With Custom Logging
    Log    This is info level
    Log    This is debug level    level=DEBUG
    Log    This is warning    level=WARN
    Log    This is error    level=ERROR
    
    # HTML logging
    Log    <b>Bold text</b>    html=True
    
    # Log to console
    Log To Console    This appears in console
```

**Real-Time Example: Comprehensive Logging**
```robot
*** Settings ***
Library    SeleniumLibrary
Library    OperatingSystem
Suite Setup    Create Results Directory
Suite Teardown    Generate Summary Report

*** Variables ***
${RESULTS_DIR}        ${CURDIR}/results
${SCREENSHOT_DIR}     ${RESULTS_DIR}/screenshots
${LOG_FILE}           ${RESULTS_DIR}/test_log.txt

*** Test Cases ***
Test With Detailed Logging
    [Documentation]    Test with screenshots and detailed logs
    [Tags]    demo
    
    Log    Starting test execution    INFO
    Append To File    ${LOG_FILE}    [${DATETIME}] Test started\n
    
    Open Browser    https://example.com    chrome
    Maximize Browser Window
    
    # Capture initial state
    Capture Page Screenshot    ${SCREENSHOT_DIR}/step1_homepage.png
    Log    <img src="screenshots/step1_homepage.png" width="800px">    html=True
    
    # Perform actions
    ${title}=    Get Title
    Log    Page Title: ${title}
    Append To File    ${LOG_FILE}    [${DATETIME}] Page title: ${title}\n
    
    # Capture final state
    Capture Page Screenshot    ${SCREENSHOT_DIR}/step2_final.png
    
    Close Browser
    
    Log    Test completed successfully    INFO
    Append To File    ${LOG_FILE}    [${DATETIME}] Test completed\n

*** Keywords ***
Create Results Directory
    Create Directory    ${RESULTS_DIR}
    Create Directory    ${SCREENSHOT_DIR}
    Create File    ${LOG_FILE}    Test Execution Log\n${'='*50}\n

Generate Summary Report
    ${log_content}=    Get File    ${LOG_FILE}
    Log    ${log_content}
    Log To Console    \nTest execution completed. Check ${RESULTS_DIR} for results.
```

---

## Summary: Complete Real-Time Test Suite

Here's a comprehensive example combining all concepts:

**test_suite.robot:**
```robot
*** Settings ***
Documentation     Complete E-commerce Test Suite
Library           SeleniumLibrary
Library           RequestsLibrary
Library           Collections
Resource          resources/common_keywords.robot
Resource          resources/pages/login_page.robot
Resource          resources/pages/product_page.robot

Suite Setup       Initialize Test Environment
Suite Teardown    Cleanup Test Environment
Test Setup        Open Browser To Homepage
Test Teardown     Close Browser And Capture On Failure

*** Variables ***
${BASE_URL}           https://shop.example.com
${BROWSER}            chrome
${TIMEOUT}            10s
${VALID_USER}         testuser@example.com
${VALID_PASS}         Test@12345

*** Test Cases ***
User Can Register
    [Tags]    smoke    registration    critical
    [Documentation]    Verify new user can register successfully
    
    Navigate To Registration Page
    ${random_email}=    Generate Random Email
    Fill Registration Form    John    Doe    ${random_email}    ${VALID_PASS}
    Submit Registration
    Verify Registration Success
    
    # Store for later tests
    Set Suite Variable    ${NEW_USER_EMAIL}    ${random_email}

User Can Login
    [Tags]    smoke    login    critical    fast
    [Documentation]    Verify user can login with valid credentials
    
    Navigate To Login Page
    Enter Credentials    ${VALID_USER}    ${VALID_PASS}
    Click Login Button
    Verify User Logged In    ${VALID_USER}

Search And Add Product To Cart
    [Tags]    regression    cart    medium
    [Documentation]    Search for product and add to cart
    [Setup]    Login As User    ${VALID_USER}    ${VALID_PASS}
    
    ${product_name}=    Search And Select First Product    laptop
    Add To Cart
    Verify Product In Cart    ${product_name}

Complete Purchase Flow
    [Tags]    critical    checkout    e2e    slow
    [Documentation]    Complete end-to-end purchase flow
    [Setup]    Login As User    ${VALID_USER}    ${VALID_PASS}
    
    # Add products
    Search And Add Product    laptop
    Search And Add Product    mouse
    
    # Checkout
    Navigate To Cart
    Proceed To Checkout
    Fill Shipping Details    John Doe    123 Main St    New York    10001
    Select Payment Method    Credit Card
    Enter Payment Details    4111111111111111    12/25    123
    Place Order
    
    # Verify
    ${order_number}=    Get Order Number
    Verify Order Confirmation    ${order_number}
    
    # API verification
    Verify Order In Backend    ${order_number}

API - Get All Products
    [Tags]    api    regression    fast
    [Documentation]    Verify API returns product list
    
    Create Session    api    ${BASE_URL}/api
    ${response}=    GET On Session    api    /products
    
    Should Be Equal As Numbers    ${response.status_code}    200
    ${products}=    Set Variable    ${response.json()}
    ${count}=    Get Length    ${products}
    Should Be True    ${count} > 0

API - Create And Delete Product
    [Tags]    api    admin    regression
    [Documentation]    Admin can create and delete products via API
    
    ${auth_token}=    Get Admin Auth Token
    &{headers}=    Create Dictionary    Authorization=Bearer ${auth_token}
    
    # Create product
    &{product}=    Create Dictionary    name=Test Product    price=99.99    stock=10
    Create Session    api    ${BASE_URL}/api    headers=${headers}
    ${response}=    POST On Session    api    /products    json=${product}
    Should Be Equal As Numbers    ${response.status_code}    201
    
    ${product_id}=    Get From Dictionary    ${response.json()}    id
    
    # Delete product
    ${response}=    DELETE On Session    api    /products/${product_id}
    Should Be Equal As Numbers    ${response.status_code}    204

*** Keywords ***
Initialize Test Environment
    Log    Initializing test environment
    Set Selenium Speed    0.3s
    Set Selenium Timeout    ${TIMEOUT}

Cleanup Test Environment
    Log    Cleaning up test environment
    Close All Browsers

Open Browser To Homepage
    Open Browser    ${BASE_URL}    ${BROWSER}
    Maximize Browser Window

Close Browser And Capture On Failure
    Run Keyword If Test Failed    Capture Page Screenshot
    Close Browser

Generate Random Email
    ${timestamp}=    Get Time    epoch
    ${email}=    Set Variable    testuser${timestamp}@example.com
    [Return]    ${email}

Login As User
    [Arguments]    ${email}    ${password}
    Navigate To Login Page
    Enter Credentials    ${email}    ${password}
    Click Login Button
    Verify User Logged In    ${email}

Search And Add Product
    [Arguments]    ${search_term}
    ${product}=    Search And Select First Product    ${search_term}
    Add To Cart
    [Return]    ${product}

Verify Order In Backend
    [Arguments]    ${order_number}
    Create Session    api    ${BASE_URL}/api
    ${response}=    GET On Session    api    /orders/${order_number}
    Should Be Equal As Numbers    ${response.status_code}    200
    ${order}=    Set Variable    ${response.json()}
    Should Be Equal    ${order}[order_number]    ${order_number}
    Should Be Equal    ${order}[status]    confirmed
```

---

## Running The Tests

```bash
# Run all tests
robot test_suite.robot

# Run smoke tests only
robot --include smoke test_suite.robot

# Run critical tests, exclude slow ones
robot --include critical --exclude slow test_suite.robot

# Run with custom report
robot --outputdir results --name "E-commerce Tests" test_suite.robot

# Run in parallel (requires pabot)
pabot --processes 4 test_suite.robot

# Run with different browser
robot --variable BROWSER:firefox test_suite.robot

# Run with different environment
robot --variable BASE_URL:https://staging.example.com test_suite.robot
```

---

This completes the comprehensive Robot Framework guide! Each concept is explained with real-time examples showing practical usage in various testing scenarios.
