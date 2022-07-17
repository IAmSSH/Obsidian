# Installation
```bash
npm install cypress
```

# Opening Cypress Instance
```bash
npx cypress open
```

# Ignoring Files

In `cypress.json`, add following properties:

```json
{
	// to ignore all files inside examples directory
	"ignoreTestFiles": "**/examples/*",
	
	// set base url
	"baseUrl": "...",

	// some other useful properties
	"viewportWidth": 1920,
	"viewportHeight": 1080
}
```

# Basics
Tests have to end with `.spec.js`

We can add following line to the test file to tell VS Code that this is a Cypress test, so that it enables some Cypress specific editor features
`/// reference types="cypress" />`

### Test Blocks
Tests need to have a ==block==.
It is defined either using `context` or `describe`
It takes in 2 args:
1. Description of the test
2. Function to configure tests (which is basically all our test code)

```js
context("test description", () => {
	// ...
})
```

For each test we can tell Cypress to go to a url using hook

```js
describe('', () => {
	beforeAll(() => {
		cy.visit('url')
	})
})
```

A test can be defined using `it()`

```js
describe('', () => {
	beforeAll(() => {
		cy.visit('url')
	})

	it('uses get', () => {
		cy.get('tag_name')
	})
})
```

`.get()` basically uses CSS selectors. So, tag names, class names, etc. can be used to get the elements

To get elements match multiple classes:
	*`[]` is attribute selector*

```js
it('uses get', () => {
	cy.get("[class=class1 class2]")
})
```

Getting using other attributes

```js
it('uses get', () => {
	// get element that has type attribute
	cy.get("[type='submit']")
})
```

### Combining selectors

```js
it('uses get', () => {
	// button is tag, className is the class
	cy.get("button.className")
})
```

### Custom Commands
We can also create custom commands and use them like `get()` to test.
They are created inside the `support` folder.

Syntax:

```js
Cypress.Commands.add('getByTestId', (id) => { 
	cy.get(`[data-cy=${id}]`)
})
```

### Firing Events
Simply append the event after getting the element.
After firing the event, we can get elements that we expect to appear on screen.
We can also use ==Assertions==

```js
it('uses get', () => {
	cy
	.get("button.className")
	.click() // click event
	
	// modal is only visible after click
	// which is what we are checking
	cy.get(".modal").should("be.visible")
})
```

Similarly, we can use `type` event to simulate typing

```js
it('uses get', () => {
	cy
	.get("input[placeholder='Habit']")
	.type("Drink a cup of water")
})
```

# Intercepting HTTP Requests
We use `cy.intercept()` to intercept HTTP requests
It takes 3 things (*need to check documentation for other signatures*)
1. Request method
2. Url 
3. A third arg that specifies the response
	1. This can either be a file (e.g. in a GET request) or a handler function from which we can manully return response

### Returning a file:
```js
describe("Accomplishment dashboard", () => {

    beforeEach(() => {
        cy.visit("/rewards")
    })

    it("should display a list of rewards with mock", () => {
	    // The page makes a request to `/rewards` url
	    // when a user lands on it.
	    // That is the request that we are intercepting
        cy
		.intercept(
			"GET", 
			"http://localhost:4000/rewards", 
			// returning a file
			{ fixture: "rewards.json" }
		)

		// After landing, we should see the
		// data returned from the intercepted request
        cy
        .get("ul")
        .should(
	        "contain", 
	        "500 points for drinking 8 cups of water for 7 straight days"
		)
        .and(
	        "contain", 
	        "850 points for fasting for 5 days straight"
		)
    })
})
```

### Using handler:
In the below example, we 
- first ==define the interceptor==
- then, ==fill in the required values==
- then, hit the ==submit== button

```js
it(
    "should display error when text includes giraffe (with mock)",
    () => {
        cy
        .intercept(
            'POST',
            'http://localhost:4000',
            (req) => {
                req.reply((res) => {
                    res.send({
                        msg: "Your content is not appropriate"
                    })
                })
            })

        cy.get("[placeholder='Title']")
	        .type("This is my accomplishment")
	        
        cy.get("[placeholder='My accomplishment...']")
	        .type("I pet a giraffe")
	        
        cy.get("[type='checkbox']").click();
        
        cy.get("button").click();
        
        cy
		.contains(
			"Your content is not appropriate"
		)
		.should("be.visible")
    }
)
```