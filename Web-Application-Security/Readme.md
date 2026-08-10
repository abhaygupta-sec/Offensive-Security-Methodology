# Web Application Security Methodology

Web application penetration testing focuses on understanding how an application processes requests, handles authentication and authorization, accepts user-controlled input, manages sessions, and interacts with backend systems.

The methodology documented here emphasizes **manual request analysis, attack-surface mapping, controlled input testing, and manual validation of potential vulnerabilities**.

---

## Methodology

My general web application testing workflow follows:

```text
Application Discovery
        ↓
Technology Identification
        ↓
Endpoint / Parameter Discovery
        ↓
Request & Response Analysis
        ↓
Authentication / Authorization Testing
        ↓
Input Validation Testing
        ↓
Vulnerability Identification
        ↓
Manual Validation
        ↓
Impact Assessment
        ↓
Professional Reporting

The core principle is:

Understand the application before attacking it.

1. Application Reconnaissance

Start by understanding how the application works.

Review:

Main pages
Login functionality
Registration
Password reset
Search functionality
File uploads
Administrative functionality
API endpoints
HTTP methods
Cookies
Session tokens
Redirects
Error messages
JavaScript
Hidden functionality

The objective is to build an application map.

Application
    ↓
Pages
    ↓
Endpoints
    ↓
Parameters
    ↓
Authentication
    ↓
Authorization
    ↓
Backend Functionality
2. Burp Suite / HTTP Proxy

Burp Suite is used to intercept, modify, replay, and analyze HTTP requests.

The basic workflow is:

Browser
   ↓
Burp Proxy
   ↓
HTTP Request
   ↓
Modify Request
   ↓
Send to Application
   ↓
Analyze Response

Important request components include:

HTTP method
URL
Parameters
Headers
Cookies
Authorization tokens
Request body
Content type

Important response components include:

Status code
Response body
Headers
Cookies
Redirects
Error messages
Response length
Timing
3. HTTP Method Testing

Identify which HTTP methods the application accepts.

Common methods include:

GET
POST
PUT
DELETE
PATCH
OPTIONS
HEAD

The objective is to determine whether functionality changes depending on the HTTP method.

Example:

GET /resource
POST /resource
PUT /resource
DELETE /resource

Do not assume that because an endpoint rejects one method, the underlying functionality is inaccessible through another.

4. Endpoint Discovery

Identify hidden and undocumented application functionality.

Potential sources include:

Robots.txt
Sitemap
JavaScript files
HTTP history
API documentation
Error messages
Backup files
Source code
Directory enumeration
URL parameters

FFUF can be used for authorized content discovery:

ffuf -u http://<TARGET>/FUZZ -w <WORDLIST>

Common interesting paths include:

/admin
/login
/api
/uploads
/backup
/config
/.git

Every discovered endpoint should be manually reviewed.

5. Parameter Discovery

Parameters are often more important than endpoints.

Look for:

?id=
?user=
?file=
?page=
?url=
?redirect=
?search=
?cmd=
?path=
?template=

Also inspect parameters in:

POST bodies
JSON
Cookies
Headers
Multipart requests
API requests

For every parameter ask:

What does it control?
        ↓
Is it reflected?
        ↓
Is it processed server-side?
        ↓
Is it used in a file path?
        ↓
Is it used in a command?
        ↓
Is it used in a database query?
        ↓
Is it used for authorization?
6. Authentication Testing

Authentication testing focuses on how the application identifies and verifies users.

Test areas include:

Login
Registration
Password reset
Account recovery
Session handling
Remember-me functionality
Authentication tokens
Username enumeration
Rate limiting
Authentication bypass conditions

Compare responses carefully.

For example:

Valid Username + Wrong Password
        vs.
Invalid Username + Wrong Password

Differences in:

Status code
Response length
Error message
Redirect
Response timing

may reveal useful information.

7. Brute-Force / Credential Testing

Where authorized, authentication endpoints can be tested for weak credential controls.

The methodology is:

Identify Login Endpoint
        ↓
Understand Request Format
        ↓
Identify Failure Condition
        ↓
Determine Rate Limiting
        ↓
Test Controlled Credentials
        ↓
Validate Authentication

The important part is identifying the application's actual success/failure condition before automating requests.

8. Authorization Testing

Authentication answers:

Who are you?

Authorization answers:

What are you allowed to do?

Test whether one user's requests can access another user's resources.

Look for:

User A
  ↓
Object ID
  ↓
Modify ID
  ↓
User B's Object
  ↓
Unauthorized Access?

Potentially interesting parameters include:

?id=
?user_id=
?account=
?document=
?order=
?profile=

Authorization testing should be performed using controlled accounts and objects in an authorized environment.

9. IDOR / Broken Object-Level Authorization

When an application references objects using predictable identifiers, test whether authorization is enforced server-side.

Example:

GET /api/user/1001

Test with another authorized test account:

GET /api/user/1002

The important question is not whether the ID can be changed.

The important question is:

Does the server verify that the authenticated user is authorized to access the requested object?

10. SQL Injection

SQL injection occurs when user-controlled input influences a database query without sufficient separation between data and SQL syntax.

Potential parameters include:

?id=
?user=
?search=
?sort=
?filter=

The testing workflow is:

Identify Parameter
        ↓
Observe Normal Response
        ↓
Test Input Behavior
        ↓
Identify SQL-Related Errors / Behavioral Changes
        ↓
Validate Manually
        ↓
Determine Database Interaction

SQLMap can be used in authorized environments after the parameter has been identified:

sqlmap -u "http://<TARGET>/page?id=1" --batch

Do not treat a scanner result as proof without understanding the underlying request and impact.

11. Cross-Site Scripting (XSS)

XSS testing determines whether attacker-controlled input can be interpreted as executable client-side content.

Common locations include:

Search fields
Profile fields
Comments
URL parameters
HTTP headers
Error pages

Testing workflow:

Input
 ↓
Reflection / Storage
 ↓
Context Identification
 ↓
Output Encoding Analysis
 ↓
Controlled Payload
 ↓
Execution Validation

Distinguish between:

Reflected XSS
Stored XSS
DOM-based XSS

The context matters because HTML, attribute, JavaScript, and URL contexts require different validation approaches.

12. Command Injection

Command injection occurs when user-controlled input reaches an operating-system command interpreter.

Potentially interesting parameters include:

?host=
?ip=
?file=
?cmd=
?path=
?name=

Testing workflow:

Identify Input
      ↓
Determine Server-Side Processing
      ↓
Test Command Context
      ↓
Validate Execution
      ↓
Determine Privilege / Impact

The objective is to prove controlled command execution rather than simply observe an application error.

13. Server-Side Template Injection

SSTI occurs when attacker-controlled input is interpreted by a server-side template engine.

The workflow is:

Identify Reflected Input
        ↓
Determine Whether Template Syntax Is Evaluated
        ↓
Identify Template Engine
        ↓
Understand Execution Context
        ↓
Validate Impact

The important distinction is:

Input reflected
        ≠
Template execution

Manual validation is required.

14. File Inclusion

Applications that accept file paths or resource identifiers should be reviewed for unintended file access.

Potential parameters:

?file=
?page=
?path=
?template=
?include=

The testing process is:

Identify File Parameter
        ↓
Understand Expected Input
        ↓
Test Path Handling
        ↓
Determine Traversal / Inclusion Behavior
        ↓
Validate Accessible Resources

Potential impact depends on the application and server configuration.

15. File Upload Testing

File upload functionality should be assessed beyond simply checking whether a file can be uploaded.

Review:

Extension validation
MIME validation
Filename handling
Storage location
Permissions
Renaming
Server-side processing
Image/document processing
Access control
Download behavior

Workflow:

Upload Functionality
        ↓
Identify Validation
        ↓
Test Filename / Type Handling
        ↓
Determine Storage Location
        ↓
Determine Execution / Processing Behavior
        ↓
Validate Impact
16. SSRF

Server-Side Request Forgery occurs when the application makes outbound requests using attacker-controlled input.

Potential parameters include:

?url=
?uri=
?path=
?redirect=
?endpoint=

Testing workflow:

Identify Server-Side Request
        ↓
Control Destination
        ↓
Determine Reachability
        ↓
Identify Internal / External Access
        ↓
Validate Security Impact

The critical question is:

What can the server reach that the external attacker cannot?

17. JWT Testing

When JSON Web Tokens are used, inspect:

Header
Payload
Algorithm
Claims
Expiration
Issuer
Audience
Signature validation

Example token structure:

HEADER.PAYLOAD.SIGNATURE

Testing workflow:

Capture Token
      ↓
Decode Header / Payload
      ↓
Understand Claims
      ↓
Identify Trust Boundaries
      ↓
Test Validation Controls
      ↓
Validate Authorization Impact

Never assume that decoding a JWT means it can be modified successfully. The server's signature and claim validation must be tested.

18. CSRF

Cross-Site Request Forgery testing focuses on whether sensitive actions can be triggered without sufficient request-origin validation.

Review:

CSRF tokens
SameSite cookie settings
Origin validation
Referer validation
Authentication requirements

Potentially sensitive actions include:

Password changes
Email changes
Account settings
Administrative actions

The key question is:

Can an authenticated user's browser be induced to perform a state-changing action without sufficient CSRF protection?

19. WebSockets

WebSocket applications should be tested by inspecting messages flowing between client and server.

Review:

Connection establishment
Authentication
Message format
Client-controlled parameters
Server-side authorization
Origin validation
Input validation

Burp Suite can be used to intercept and modify WebSocket traffic in an authorized environment.

20. WordPress Enumeration

When WordPress is identified, investigate:

Version
Themes
Plugins
Users
Login functionality
Exposed files
Backup files
XML-RPC
Administrative endpoints

The goal is to identify whether outdated components, exposed information, weak authentication, or configuration issues create an attack path.

21. API Testing

APIs should be tested separately from traditional web pages.

Review:

Endpoints
Methods
Parameters
Authentication
Authorization
Object IDs
Tokens
HTTP status codes
Error handling
Rate limiting

Example:

GET /api/user/1
POST /api/user
PUT /api/user/1
DELETE /api/user/1

Important questions:

Can unauthorized users access endpoints?
Can one user access another user's objects?
Are hidden parameters accepted?
Are administrative endpoints exposed?
Are HTTP methods restricted?
Are server-side authorization checks enforced?
22. Manual Request Analysis

A major part of web testing is understanding the complete request.

Example:

POST /login HTTP/1.1
Host: <TARGET>
Content-Type: application/x-www-form-urlencoded

username=test&password=test

Instead of immediately automating the request, determine:

Which parameter controls authentication?
        ↓
What indicates failure?
        ↓
What indicates success?
        ↓
Are there hidden parameters?
        ↓
Does changing the method matter?
        ↓
Does the response change?

This makes later automated testing much more reliable.

23. Scanner Validation

Automated tools can accelerate discovery.

Examples include:

Burp Suite
FFUF
Nmap NSE
Nikto
SQLMap
Nessus

The methodology is:

Automated Finding
        ↓
Inspect Request
        ↓
Understand Finding
        ↓
Reproduce Manually
        ↓
Determine Impact
        ↓
Document Evidence

A scanner finding should not automatically become a report finding.

24. Web Testing Decision Tree
Application
     |
     v
Map Endpoints
     |
     v
Identify Parameters
     |
     +-----------------------------+
     |             |               |
 Authentication  Authorization   Input
     |             |               |
     v             v               v
 Login / JWT     IDOR / ACL      SQLi / XSS
 Reset / Session                 SSTI / CMDi
                                  File Inclusion
                                  Upload
                                  SSRF
     |
     v
Manual Validation
     |
     v
Impact Assessment
     |
     v
Report
25. Common Mistakes

Avoid:

Running scanners without understanding the application
Testing only the homepage
Ignoring JavaScript
Ignoring API endpoints
Ignoring HTTP history
Testing parameters without understanding their purpose
Treating reflected input as automatic XSS
Treating an error as automatic SQL injection
Assuming a JWT can be forged just because it can be decoded
Testing authorization with only one account
Ignoring cookies and session state
Ignoring alternate HTTP methods
Reporting scanner findings without manual validation
Failing to document failed tests and reasoning
26. Web Application Testing Mindset

The objective is not:

Run Scanner
    ↓
Find Vulnerability
    ↓
Done

The objective is:

Understand Application
        ↓
Map Attack Surface
        ↓
Identify Trust Boundaries
        ↓
Manipulate Inputs
        ↓
Observe Behavior
        ↓
Form Hypothesis
        ↓
Validate
        ↓
Determine Impact

The strongest web assessments come from understanding how the application processes attacker-controlled input.

27. Reporting Web Findings

For every confirmed vulnerability, document:

WHAT
What functionality is vulnerable?

WHY
Why does the behavior matter?

REQUEST
What request was tested?

RESULT
What did the server return?

IMPACT
What can an attacker actually achieve?

REMEDIATION
How should the vulnerability be fixed?

Example structure:

Finding
    ↓
Affected Endpoint
    ↓
Affected Parameter
    ↓
Technical Explanation
    ↓
Validation
    ↓
Impact
    ↓
Remediation
Related Projects
HTB Writeups

Practical machine assessments demonstrating web enumeration and exploitation.

HTB-Writeups

Offensive Security Methodology

See the repository root for the broader penetration-testing methodology.

Responsible Use

All techniques documented here are intended for:

Authorized penetration tests
Web-security laboratories
Hack The Box and other CTF environments
Security research
Applications owned or explicitly authorized for testing

Never test applications or endpoints without authorization.
