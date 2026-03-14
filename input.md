QA-204

Summary: Verify user login and logout flow on DemoQA Book Store

Description:
The Book Store application on DemoQA has a login page at
https://demoqa.com/login where registered users can sign in
using their username and password. After a successful login
the user should land on their profile page showing their
username and a collection of books. The logout button should
return the user to the login page. Invalid credentials should
show an error message without logging in.

Acceptance Criteria:
1. Navigate to https://demoqa.com/login — page loads with
   Username and Password fields and a Login button visible

2. Click Login without entering any credentials — form should
   show validation error or button should remain non-functional,
   page should not navigate away

3. Enter invalid username: fakeuser123 and invalid
   password: wrongpass — click Login — error message
   "Invalid username or password!" should appear, user
   should remain on login page

4. Clear fields and enter valid username: testuser
   and valid password: Test@1234 — fields should accept input
   without errors

5. Click Login with valid credentials — user should be
   redirected to /profile, username should be visible on page,
   Log Out button should be present

6. Click Log Out button — user should be redirected back
   to /login page, session should be cleared

Attachments: demoqa-login-design.png, error-state-reference.png