---
layout: post
title: Azure AD and Cypress - How tests can "login" without the UI
categories: testing
tags: [testing,azure,devops,cypress]
author: AnitaLipsky 
---

# Intro

This walks through the challenge, the desired type of solution and the more detailed solution that our team came up with together for this.

It would not have been possible for myself to solve this alone.  Because each application is unique I needed to be guided through how that particular application processed token(s) and then try out potential solutions together.


## The challenge
Run automated UI tests using [Cypress.io](https://www.cypress.io/) for an application which uses Azure AD.

It is not only bad practise to use the UI to click on login buttons for each test, there are [issues doing so with Cypress](https://github.com/cypress-io/cypress/issues/1342).


### Why tests should not build up state via the UI

It is considered bad practise for automated tests to login via the UI is because it is typically slow and can be unstable potentially causing the tests to fail while building up state.

For example, UI test steps look for the desired elements such as username and password fields, interact with them by typing in values, look for the Login button, then interact with it by clicking.  Any of these steps could fail for the wrong reason, and the total time to run them is typically slow compared to test steps without UI interactions.

Instead tests should fail due to undesired behaviour in the application. A test should build up state for what is being tested in the fastest and most stable way.  The exception is if the test is verifying the login page itself, in which case interacting with elements on the login page via the UI is desired test behaviour.


## The type of solution we were looking for
A way to generate the necessary authentication token(s) that give access to the application for that particular user and set of roles.

Using endpoints only - in other words, not via interacting with any slow and possibly unstable UI elements.

## The high level solution


What we knew about the application

1. The app uses Azure AD for SSO
1. The app expects an authentication id_token
1. We knew what name the app used to set the token to, and that it should be stored in session storage

The solution

1. Use [ROPC flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth-ropc#authorization-request) to generate the id_token. An access_token is also returned however we did not need to do anything with it.
2. Save the value of the id_token in session storage using the name the app expects

Note: Although Microsoft has a warning regarding using the ROPC flow, it is not applicable in this case because this solution will not work on the production environment - the test environment and users are setup in a different and separate way.

## The detailed solution

Create the request using ```cy.request``` and get the response, store it in a token with the appropriate name in session storage.

This was added to a custom command in Cypress, and this command was called at the start of individual tests.

### tests/e2e/support/commands.js


```bash
Cypress.Commands.add("authenticateUsingToken", window => {
  const clientId = "myClientId";
  const tenantId = "mytenantId";
  const client_secret = "myclient_secret";

  cy.request({
    // Given: I send auth request
    method: "POST",
    url: `https://login.microsoftonline.com/${tenantId}/oauth2/v2.0/token`,
    header: {
      "cache-control": "no-cache",
      "Content-Type": "application/x-www-form-urlencoded"
    },
    form: true,
    body: {
      client_id: clientId,
      username: "myuser@myapp.onmicrosoft.com",
      password: "mypassword",
      grant_type: "password",
      client_secret: my_client_secret,
      scope: "api://myappapi/Users.Read"
    }
  }).then(response => {
    // When: I get a token
    const token = response.body.access_token;

    // Then: I set a token
    window.sessionStorage.setItem("my-token-name", token);
  });
});
```


### Example of usage in a test file

It will now be possible to navigate directly to the url desired

```bash
describe("View Products", () => {
  beforeEach(() => {
    cy.window().then(win => {

      // ensure session storage is cleared
      win.sessionStorage.clear();

      // set the token
      cy.authenticateUsingToken(win);
    });
  });

  it("should visit the Product Page", () => {
    // Given: I visit the page
    cy.visit("/products");
    // etc..
   });
});
```

This custom command can be refactored to have the username and password as parameters, so tests can use the appropriate user.


### But what about Two factor authentication (2FA)?

Best practice today is to use 2FA even on non production environments.  In that case there needs to be some authenticator installed on that environment.  This blog post does not address this topic.


## Conclusion

This recipe will work for any OAuth sign on service like Okta, IDCS, Google, AWS...

Having good communication with team members with various expertise is always the key to solving challenges.  I am fortunate to have such knowledgable and patient teammates willing to work together to deliver software of value early and often.

In this case knowledge of how the application processes the token, how Azure AD is set up on Azure DevOps for the application and knowledge of Cypress were the key to solving this.

In short, team work!