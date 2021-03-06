---
id: user-login-user-registration
title: User Login and User Registration
---

ORY Kratos supports two type of login and registration flows:

- Browser-based (easy): This flow works for all applications running on top of a
  browser. Websites, single-page apps, Cordova/Ionic, and so on.
- API-based (advanced): This flow works for native applications like iOS
  (Swift), Android (Java), Microsoft (.NET), React Native, Electron, and others.

## Self-Service User Login and User Registration for Browser Applications

ORY Kratos supports browser applications that run on server-side (e.g. Java,
NodeJS, PHP) as well as client-side (e.g. JQuery, ReactJS, AngularJS, ...).

Browser-based login and registration makes use of three core HTTP technologies:

- HTTP Redirects
- HTTP POST (`application/json`, `application/x-www-urlencoded`) and RESTful GET
  requests.
- HTTP Cookies to prevent CSRF and Session Hijaking attack vectors.

The browser flow is the easiest and most secure to set up and integrated with.
ORY Kratos takes care of all required session and CSRF cookies and ensures that
all security requirements are fulfilled.

Future versions of ORY Kratos will be able to deal with multi-domain
environments that require SSO. For example, one account would be used to sign
into both `mydomain.com` and `anotherdomain.org`. A common real-world example is
using your Google account to seamlessly be signed into YouTube and Google at the
same time.

This flow is not suitable for scenarios where you use purely programmatic
clients that do not work well with HTTP Cookies and HTTP Redirects.

### The Login and Registration User Interface

In Browser Applications, the User Interface is typically rendered as an HTML
Form:

```html
<!-- Login -->
<form action="..." method="POST">
  <input type="text" name="identifier" placeholder="Enter your username" />
  <input type="password" name="password" placeholder="Enter your password" />
  <input type="hidden" name="csrf_token" value="cdef..." />
  <input type="submit" />
</form>

<!-- Registration -->
<form action="..." method="POST">
  <input type="email" name="email" placeholder="Enter your E-Mail Address" />
  <input type="password" name="password" placeholder="Enter your password" />
  <input
    type="first_name"
    name="password"
    placeholder="Enter your First Name"
  />
  <input type="last_name" name="password" placeholder="Enter your Last Name" />
  <input type="hidden" name="csrf_token" value="cdef..." />
  <input type="submit" />
</form>
```

Depending on the type of login flows you want to support, this might also be a
"Sign up/in with GitHub" flow:

```html
<!-- Login and Registration -->
<form action="..." method="POST">
  <input type="hidden" name="csrf_token" value="cdef..." />

  <!-- Basically <a href="https://github.com/login/oauth/authorize?...">Sign up/in with GitHub</a> -->
  <input type="submit" name="provider" value="GitHub" />
</form>
```

In stark contrast to other Identity Systems, ORY Kratos does not render this
HTML. Instead, you need to implement the HTML code in your application (e.g.
NodeJS + ExpressJS, Java, PHP, ReactJS, ...), which gives you extreme
flexibility and customizability in your user interface flows and designs.

Each Login and Registration Strategy (e.g.
[Username and Password](../strategies/username-email-password.md),
[Social Sign In](../strategies/openid-connect-social-sign-in-oauth2.md),
Passwordless, ...) works a bit differently but they all boil down to the same
abstract sequence:

[![Abstract Login and Registration User Flow](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIHBhcnRpY2lwYW50IEIgYXMgQnJvd3NlclxuICBwYXJ0aWNpcGFudCBLIGFzIE9SWSBLcmF0b3NcbiAgcGFydGljaXBhbnQgQSBhcyBZb3VyIEFwcGxpY2F0aW9uXG5cblxuICBCLT4-SzogSW5pdGlhdGUgTG9naW5cbiAgSy0-PkI6IFJlZGlyZWN0cyB0byB5b3VyIEFwcGxpY2F0aW9uJ3MgL2xvZ2luIGVuZHBvaW50XG4gIEItPj5BOiBDYWxscyAvbG9naW5cbiAgQS0tPj5LOiBGZXRjaGVzIGRhdGEgdG8gcmVuZGVyIGZvcm1zIGV0Y1xuICBCLS0-PkE6IEZpbGxzIG91dCBmb3JtcywgY2xpY2tzIGUuZy4gXCJTdWJtaXQgTG9naW5cIlxuICBCLT4-SzogUE9TVHMgZGF0YSB0b1xuICBLLS0-Pks6IFByb2Nlc3NlcyBMb2dpbiBJbmZvXG5cbiAgYWx0IExvZ2luIGRhdGEgdmFsaWRcbiAgICBLLS0-PkI6IFNldHMgc2Vzc2lvbiBjb29raWVcbiAgICBLLT4-QjogUmVkaXJlY3RzIHRvIGUuZy4gRGFzaGJvYXJkXG4gIGVsc2UgTG9naW4gZGF0YSBpbnZhbGlkXG4gICAgSy0tPj5COiBSZWRpcmVjdHMgdG8geW91ciBBcHBsaWNhaXRvbidzIC9sb2dpbiBlbmRwb2ludFxuICAgIEItPj5BOiBDYWxscyAvbG9naW5cbiAgICBBLS0-Pks6IEZldGNoZXMgZGF0YSB0byByZW5kZXIgZm9ybSBmaWVsZHMgYW5kIGVycm9yc1xuICAgIEItLT4-QTogRmlsbHMgb3V0IGZvcm1zIGFnYWluLCBjb3JyZWN0cyBlcnJvcnNcbiAgICBCLT4-SzogUE9TVHMgZGF0YSBhZ2FpbiAtIGFuZCBzbyBvbi4uLlxuICBlbmRcbiIsIm1lcm1haWQiOnsidGhlbWUiOiJuZXV0cmFsIiwic2VxdWVuY2VEaWFncmFtIjp7ImRpYWdyYW1NYXJnaW5YIjoxNSwiZGlhZ3JhbU1hcmdpblkiOjE1LCJib3hUZXh0TWFyZ2luIjowLCJub3RlTWFyZ2luIjoxNSwibWVzc2FnZU1hcmdpbiI6NDUsIm1pcnJvckFjdG9ycyI6dHJ1ZX19fQ)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIHBhcnRpY2lwYW50IEIgYXMgQnJvd3NlclxuICBwYXJ0aWNpcGFudCBLIGFzIE9SWSBLcmF0b3NcbiAgcGFydGljaXBhbnQgQSBhcyBZb3VyIEFwcGxpY2F0aW9uXG5cblxuICBCLT4-SzogSW5pdGlhdGUgTG9naW5cbiAgSy0-PkI6IFJlZGlyZWN0cyB0byB5b3VyIEFwcGxpY2F0aW9uJ3MgL2xvZ2luIGVuZHBvaW50XG4gIEItPj5BOiBDYWxscyAvbG9naW5cbiAgQS0tPj5LOiBGZXRjaGVzIGRhdGEgdG8gcmVuZGVyIGZvcm1zIGV0Y1xuICBCLS0-PkE6IEZpbGxzIG91dCBmb3JtcywgY2xpY2tzIGUuZy4gXCJTdWJtaXQgTG9naW5cIlxuICBCLT4-SzogUE9TVHMgZGF0YSB0b1xuICBLLS0-Pks6IFByb2Nlc3NlcyBMb2dpbiBJbmZvXG5cbiAgYWx0IExvZ2luIGRhdGEgdmFsaWRcbiAgICBLLS0-PkI6IFNldHMgc2Vzc2lvbiBjb29raWVcbiAgICBLLT4-QjogUmVkaXJlY3RzIHRvIGUuZy4gRGFzaGJvYXJkXG4gIGVsc2UgTG9naW4gZGF0YSBpbnZhbGlkXG4gICAgSy0tPj5COiBSZWRpcmVjdHMgdG8geW91ciBBcHBsaWNhaXRvbidzIC9sb2dpbiBlbmRwb2ludFxuICAgIEItPj5BOiBDYWxscyAvbG9naW5cbiAgICBBLS0-Pks6IEZldGNoZXMgZGF0YSB0byByZW5kZXIgZm9ybSBmaWVsZHMgYW5kIGVycm9yc1xuICAgIEItLT4-QTogRmlsbHMgb3V0IGZvcm1zIGFnYWluLCBjb3JyZWN0cyBlcnJvcnNcbiAgICBCLT4-SzogUE9TVHMgZGF0YSBhZ2FpbiAtIGFuZCBzbyBvbi4uLlxuICBlbmRcbiIsIm1lcm1haWQiOnsidGhlbWUiOiJuZXV0cmFsIiwic2VxdWVuY2VEaWFncmFtIjp7ImRpYWdyYW1NYXJnaW5YIjoxNSwiZGlhZ3JhbU1hcmdpblkiOjE1LCJib3hUZXh0TWFyZ2luIjowLCJub3RlTWFyZ2luIjoxNSwibWVzc2FnZU1hcmdpbiI6NDUsIm1pcnJvckFjdG9ycyI6dHJ1ZX19fQ)

The exact data being fetched and the step _"Processes Login / Registration
Info"_ depend, of course, on the actual Strategy being used. But it is important
to understand that **"Your Application"** is responsible for rendering the
actual Login and Registration HTML Forms. You can of course implement one app
for rendering all the Login, Registration, ... screens, and another app (think
"Service Oriented Architecture", "Micro-Services" or "Service Mesh") is
responsible for rendering your Dashboards, Management Screens, and so on.

::: Note It is RECOMMENDED to put all applications (or "services"), including
ORY Kratos, behind a common API Gateway or Reverse Proxy. This greatly reduces
the amount of work you have to do to get all the Cookies working properly. We
RECOMMEND using [ORY Oathkeeper](http://github.com/ory/oathkeeper) for this as
it integrates best with the ORY Ecosystem and because all of our examples use
ORY Oathkeeper. You MAY of course use any other reverse proxy (Envoy, AWS API
Gateway, Ambassador, Nginx, Kong, ...), but we do not have examples or guides
for those at this time. :::

### Code

Because Login and Registration are so similar, we can use one common piece of
code to cover both. A functioning example of the code and approach used here can
be found on
[github.com/ory/kratos-selfservice-ui-node](https://github.com/ory/kratos-selfservice-ui-node).

The code example used here is universal and does not use an SDK because we want
you to understand the fundamentals of how this flow works.

While this example assumes a Server-Side Application, a Client-Side (e.g.
ReactJS) Application would work the same, but use ORY Kratos' Public API
instead.

```js
const fetch = require('node-fetch');

const config = {
  kratos: {
    // The browser config key is used to redirect the user. It reflects where ORY Kratos' Public API
    // is accessible from. Here, we're assuming traffic going to `http://example.org/.ory/kratos/public/`
    // will be forwarded to ORY Kratos' Public API.
    browser: 'http://example.org/.ory/kratos/public/',

    // This is where ORY Kratos' Admin API is accessible.
    admin: 'https://ory-kratos-admin.example-org.vpc/',
  },
};

// The parameter "flow" can be "login" and "registration".
// You would register the two routes in express js like this:
//
//  app.get('/auth/registration', authHandler('registration'))
//  app.get('/auth/login', authHandler('login'))

export const authHandler = (flow) => (req, res, next) => {
  // The request ID is used to identify the login and registraion request and
  // return data like the csrf_token and so on.
  const request = req.query.request;
  if (!request) {
    console.log('No request found in URL, initializing ${flow} flow.');
    res.redirect(`${config.kratos.browser}/auth/browser/${flow}`);
    return;
  }

  // This is the ORY Kratos URL. If this app and ORY Kratos are running
  // on the same (e.g. Kubernetes) cluster, this should be ORY Kratos's internal hostname.
  const url = new URL(`${config.kratos.admin}/auth/browser/requests/${flow}`);
  url.searchParams.set('request', request);

  fetch(url.toString())
    .then((response) => {
      // A 404 code means that this code does not exist. We'll retry by re-initiating the flow.
      if (response.status == 404) {
        res.redirect(`${config.kratos.browser}/auth/browser/${flow}`);
        return;
      }

      return response.json();
    })
    .then((request) => {
      // Request contains all the request data for this Registration request.
      // You can process that data here, if you want.

      // Lastly, you probably want to render the data using a view (e.g. Jade Template):
      res.render(flow, request);
    });
};
```

For details on payloads and potential HTML snippets consult the individual
Self-Service Strategies for:

- [Username and Password Strategy](../strategies/username-email-password.md)
- [Social Sign In Strategy](../strategies/openid-connect-social-sign-in-oauth2.md)

### Server-Side Browser Applications

Let's take a look at the concrete network topologies, calls, and payloads. Here,
we're assuming that you're running a server-side browser application (written in
e.g. PHP, Java, NodeJS) to render the login and registration screen on the
server and make all API calls from that server code. The counterpart to this
would be a client-side browser application (written in e.g. Vanilla JavaScript,
JQuery, ReactJS, AngularJS, ...) that uses AJAX requests to fetch data. For
these type of applications, read this section first and go to section
[Client-Side Browser Applications](#client-side-browser-applications) next.

#### Network Architecture

We recommend checking out the
[Quickstart Network Architecture](../../quickstart.mdx#network-architecture) for
a high-level, exemplary, overview of the network. In summary:

1. The SecureApp (your application) is exposed at http://127.0.0.1:4455 and
   proxies requests matching path `./ory/kratos/public/*` to ORY Krato's Public
   API Port.
1. ORY Kratos exposes (for debugging only!!) the Public API at
   http://127.0.0.1:4433 and Admin API at http://127.0.0.1:4434.
1. Within the "intranet" or "private network", ORY Kratos is exposed at
   http://kratos:4433 and http://kratos:4434. These URLs are be used by the
   SecureApp to communicate with ORY Kratos.

Keep in mind that his architecture is just one of many possible network
architectures. It is however one of the simplest as well and it works locally.
For production deployments you would probably use an Reverse Proxy such as
Nginx, Kong, Envoy, ORY Oathkeeper, or others.

#### User Login and User Registration Process Sequence

The Login and Registration User Flow is composed of several high-level steps
summarized in this state diagram:

[![User Login and Registration State Machine](https://mermaid.ink/img/eyJjb2RlIjoic3RhdGVEaWFncmFtXG4gIHMxOiBVc2VyIGJyb3dzZXMgYXBwXG4gIHMyOiBFeGVjdXRlIFwiQmVmb3JlIExvZ2luL1JlZ2lzdHJhdGlvbiBIb29rKHMpXCJcbiAgczM6IFVzZXIgSW50ZXJmYWNlIEFwcGxpY2F0aW9uIHJlbmRlcnMgXCJMb2dpbi9SZWdpc3RyYXRpb24gUmVxdWVzdFwiXG4gIHM0OiBFeGVjdXRlIFwiQWZ0ZXIgTG9naW4vUmVnaXN0cmF0aW9uIEhvb2socylcIlxuICBzNTogVXBkYXRlIFwiTG9naW4vUmVnaXN0cmF0aW9uIFJlcXVlc3RcIiB3aXRoIEVycm9yIENvbnRleHQocylcbiAgczY6IExvZ2luL1JlZ2lzdHJhdGlvbiBzdWNjZXNzZnVsXG5cblxuXG5cdFsqXSAtLT4gczFcbiAgczEgLS0-IHMyIDogVXNlciBjbGlja3MgXCJMb2cgaW4gLyBTaWduIHVwXCJcbiAgczIgLS0-IEVycm9yIDogQSBob29rIGZhaWxzXG4gIHMyIC0tPiBzMyA6IFVzZXIgaXMgcmVkaXJlY3RlZCB0byBMb2dpbi9SZWdpc3RyYXRpb24gVUkgVVJMXG4gIHMzIC0tPiBzNCA6IFVzZXIgcHJvdmlkZXMgdmFsaWQgY3JlZGVudGlhbHMvcmVnaXN0cmF0aW9uIGRhdGFcbiAgczMgLS0-IHM1IDogVXNlciBwcm92aWRlcyBpbnZhbGlkIGNyZWRlbnRpYWxzL3JlZ2lzdHJhdGlvbiBkYXRhXG4gIHM1IC0tPiBzMyA6IFVzZXIgaXMgcmVkaXJlY3RlZCB0byBMb2dpbi9SZWdpc3RyYXRpb24gVUkgVVJMXG4gIHM0IC0tPiBFcnJvciA6IEEgSG9vayBmYWlsc1xuICBzNCAtLT4gczZcbiAgczYgLS0-IFsqXVxuXG4gIEVycm9yIC0tPiBbKl1cblxuXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoic3RhdGVEaWFncmFtXG4gIHMxOiBVc2VyIGJyb3dzZXMgYXBwXG4gIHMyOiBFeGVjdXRlIFwiQmVmb3JlIExvZ2luL1JlZ2lzdHJhdGlvbiBIb29rKHMpXCJcbiAgczM6IFVzZXIgSW50ZXJmYWNlIEFwcGxpY2F0aW9uIHJlbmRlcnMgXCJMb2dpbi9SZWdpc3RyYXRpb24gUmVxdWVzdFwiXG4gIHM0OiBFeGVjdXRlIFwiQWZ0ZXIgTG9naW4vUmVnaXN0cmF0aW9uIEhvb2socylcIlxuICBzNTogVXBkYXRlIFwiTG9naW4vUmVnaXN0cmF0aW9uIFJlcXVlc3RcIiB3aXRoIEVycm9yIENvbnRleHQocylcbiAgczY6IExvZ2luL1JlZ2lzdHJhdGlvbiBzdWNjZXNzZnVsXG5cblxuXG5cdFsqXSAtLT4gczFcbiAgczEgLS0-IHMyIDogVXNlciBjbGlja3MgXCJMb2cgaW4gLyBTaWduIHVwXCJcbiAgczIgLS0-IEVycm9yIDogQSBob29rIGZhaWxzXG4gIHMyIC0tPiBzMyA6IFVzZXIgaXMgcmVkaXJlY3RlZCB0byBMb2dpbi9SZWdpc3RyYXRpb24gVUkgVVJMXG4gIHMzIC0tPiBzNCA6IFVzZXIgcHJvdmlkZXMgdmFsaWQgY3JlZGVudGlhbHMvcmVnaXN0cmF0aW9uIGRhdGFcbiAgczMgLS0-IHM1IDogVXNlciBwcm92aWRlcyBpbnZhbGlkIGNyZWRlbnRpYWxzL3JlZ2lzdHJhdGlvbiBkYXRhXG4gIHM1IC0tPiBzMyA6IFVzZXIgaXMgcmVkaXJlY3RlZCB0byBMb2dpbi9SZWdpc3RyYXRpb24gVUkgVVJMXG4gIHM0IC0tPiBFcnJvciA6IEEgSG9vayBmYWlsc1xuICBzNCAtLT4gczZcbiAgczYgLS0-IFsqXVxuXG4gIEVycm9yIC0tPiBbKl1cblxuXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

1. The **Login/Registration User Flow** is initiated because a link was clicked
   or an action was performed that requires an active user session.
1. ORY Kratos executes Jobs defined in the **Before Login/Registration
   Workflow**. If a failure occurs, the whole flow is aborted.
1. The user's browser is redirected to
   `http://127.0.0.1:4455/.ory/kratos/public/auth/browser/(login|registration)`
   (the notation `(login|registration)` expresses the two possibilities of
   `../auth/browser/login` or `../auth/browser/registration`).
1. ORY Kratos does some internal processing (e.g. checks if a session cookie is
   set, generates payloads for form fields, sets CSRF token, ...) and redirects
   the user's browser to the Login UI URL which is defined using the
   `urls.login_ui` (or `urls.registration_ui`) config or `URLS_LOGIN_UI` (or
   `URLS_REGISTRATION_UI`) environment variable, which is set to the ui
   endpoints - for example `https://127.0.0.1:4455/auth/login` and
   `https://127.0.0.1:4455/auth/registration`). The user's browser is thus
   redirected to
   `https://127.0.0.1:4455/auth/(login|registration)?request=abcde`. The
   `request` query parameter includes a unique ID which will be used to fetch
   contextual data for this login request.
1. Your Server-Side Application makes a `GET` request to
   `http://kratos:4434/auth/browser/requests/(login|registration)?request=abcde`.
   ORY Kratos responds with a JSON Payload that contains data (form fields,
   error messages, ...) for all enabled User Login Strategies:
   `json5 { "id": "abcde", "expires_at": "2020-01-27T09:34:39.3249566Z", "issued_at": "2020-01-27T09:24:39.3249689Z", "request_url": "https://example.org/.ory/kratos/public/auth/browser/(login|registration)", "methods": { "oidc": { "method": "oidc", "config": { /* ... */ } }, "password": { "method": "password", "config": { /* ... */ } } // ... } }`
1. Your Server-Side applications renders the data however you see fit. The User
   interacts with it an completes the Login by clicking, for example, the
   "Login", the "Login with Google", ... button.
1. The User's browser makes a request to one of ORY Kratos' Strategy URLs (e.g.
   `http://127.0.0.1:4455/.ory/kratos/public/auth/browser/methods/password/(login|registration)`
   or
   `https://127.0.0.1:4455/.ory/kratos/public/auth/browser/methods/oidc/auth/abcde`).
   ORY Kratos validates the User's credentials (when logging in - e.g. Username
   and Password, by performing an OpenID Connect flow, ...) or the registration
   form data (when signing up - e.g. is the E-Mail address valid, is the person
   at least 21 years old, ...):
   - If the credentials / form data is invalid, the Login Request's JSON Payload
     is updated - for example with
     ```json5
     {
       id: 'abcde',
       expires_at: '2020-01-27T10:05:50.1678228Z',
       issued_at: '2020-01-27T09:55:50.1678348Z',
       request_url: 'http://127.0.0.1:4455/auth/browser/(login|registration)',
       methods: {
         oidc: {
           method: 'oidc',
           config: {
             /* ... */
           },
         },
         password: {
           method: 'password',
           config: {
             /* ... */
             errors: [
               {
                 message: 'The provided credentials are invalid. Check for spelling mistakes in your password or username, email address, or phone number.',
               },
             ],
           },
         },
       },
     }
     ```
     and the user's Browser is redirected back to the Login UI:
     `https://127.0.0.1:4455/auth/(login|registration)?request=abcde`.
   - If credentials / data is valid, ORY Kratos proceeds with the next step.
   - If the flow is a registration request and the registration data is valid,
     the identity is created.
1. ORY Kratos executes hooks defined in the **After Login/Registration
   Workflow**. If a failure occurs, the whole flow is aborted.
1. The client receives the expected response. For browsers, this is a HTTP
   Redirection, for API clients it will be a JSON response containing the
   session and/or identity. For more information on this topic check
   [Self-Service Flow Completion](../../concepts/selfservice-flow-completion.md).

[![User Login Sequence Diagram for Server-Side Applications](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIHBhcnRpY2lwYW50IEIgYXMgQnJvd3NlclxuICBwYXJ0aWNpcGFudCBBIGFzIFlvdXIgU2VydmVyLVNpZGUgQXBwbGljYXRpb25cbiAgcGFydGljaXBhbnQgS1AgYXMgT1JZIEtyYXRvcyBQdWJsaWMgQVBJXG4gIHBhcnRpY2lwYW50IEtBIGFzIE9SWSBLcmF0b3MgQWRtaW4gQVBJXG5cbiAgQi0-PitBOiBHRVQgLy5vcnkva3JhdG9zL3B1YmxpYy9zZWxmLXNlcnZpY2UvYnJvd3Nlci9mbG93cy8obG9naW58cmVnaXN0cmF0aW9uKVxuICBBLT4-K0tQOiBHRVQgL3NlbGYtc2VydmljZS9icm93c2VyL2Zsb3dzL2xvZ2luKGxvZ2lufHJlZ2lzdHJhdGlvbilcbiAgS1AtLT4-S1A6IEV4ZWN1dGUgSG9va3MgZGVmaW5lZCBpbiBcIkJlZm9yZSBMb2dpbi9SZWdpc3RyYXRpb25cIlxuICBLUC0tPj4tQTogSFRUUCAzMDIgRm91bmQgL2F1dGgvKGxvZ2lufHJlZ2lzdHJhdGlvbik_cmVxdWVzdD1hYmNkZVxuICBBLS0-Pi1COiBIVFRQIDMwMiBGb3VuZCAvYXV0aC8obG9naW58cmVnaXN0cmF0aW9uKT9yZXF1ZXN0PWFiY2RlXG5cbiAgQi0-PitBOiBHRVQgL2F1dGgvKGxvZ2lufHJlZ2lzdHJhdGlvbik_cmVxdWVzdD1hYmNkZVxuICBBLT4-K0tBOiBHRVQvc2VsZi1zZXJ2aWNlL2Jyb3dzZXIvZmxvd3MvcmVxdWVzdHMvKGxvZ2lufHJlZ2lzdHJhdGlvbik_cmVxdWVzdD1hYmNkZVxuICBLQS0-Pi1BOiBTZW5kcyBMb2dpbi9SZWdpc3RyYXRpb24gUmVxdWVzdCBKU09OIFBheWxvYWRcbiAgTm90ZSBvdmVyIEEsS0E6ICB7XCJtZXRob2RzXCI6e1wicGFzc3dvcmRcIjouLi4sXCJvaWRjXCI6Li59fVxuICBBLS0-PkE6IEdlbmVyYXRlIGFuZCByZW5kZXIgSFRNTFxuICBBLS0-Pi1COiBSZXR1cm4gSFRNTCAoRm9ybSwgLi4uKVxuXG4gIEItLT4-QjogRmlsbCBvdXQgSFRNTFxuXG4gIEItPj4rS1A6IFBPU1QgSFRNTCBGb3JtXG4gIEtQLS0-PktQOiBDaGVja3MgbG9naW4gLyByZWdpc3RyYXRpb24gZGF0YVxuXG5cbiAgYWx0IExvZ2luIGRhdGEgaXMgdmFsaWRcbiAgICBLUC0tPj4tS1A6IEV4ZWN1dGUgSm9icyBkZWZpbmVkIGluIFwiQWZ0ZXIgTG9naW4gV29ya2Zsb3cocylcIlxuICAgIEtQLS0-PkE6IEhUVFAgMzAyIEZvdW5kIC9kYXNoYm9hcmRcbiAgICBOb3RlIG92ZXIgS1AsQjogU2V0LUNvb2tpZTogYXV0aF9zZXNzaW9uPS4uLlxuICAgIEItPj4rQTogR0VUIC9kYXNoYm9hcmRcbiAgICBBLS0-S0E6IFZhbGlkYXRlcyBTZXNzaW9uIENvb2tpZVxuICAgIEEtPj4tQjogU2VuZCBEYXNoYm9hcmQgUmVzcG9uc2VcbiAgZWxzZSBMb2dpbiBkYXRhIGlzIGludmFsaWRcbiAgICBOb3RlIG92ZXIgS1AsQjogVXNlciByZXRyaWVzIGxvZ2luIC8gcmVnaXN0cmF0aW9uXG4gICAgS1AtLT4-QjogSFRUUCAzMDIgRm91bmQgL2F1dGgvKGxvZ2lufHJlZ2lzdHJhdGlvbik_cmVxdWVzdD1hYmNkZVxuICBlbmRcbiAgIiwibWVybWFpZCI6eyJ0aGVtZSI6Im5ldXRyYWwiLCJzZXF1ZW5jZURpYWdyYW0iOnsiZGlhZ3JhbU1hcmdpblgiOjE1LCJkaWFncmFtTWFyZ2luWSI6MTUsImJveFRleHRNYXJnaW4iOjEsIm5vdGVNYXJnaW4iOjEwLCJtZXNzYWdlTWFyZ2luIjo1NSwibWlycm9yQWN0b3JzIjp0cnVlfX0sInVwZGF0ZUVkaXRvciI6ZmFsc2V9)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIHBhcnRpY2lwYW50IEIgYXMgQnJvd3NlclxuICBwYXJ0aWNpcGFudCBBIGFzIFlvdXIgU2VydmVyLVNpZGUgQXBwbGljYXRpb25cbiAgcGFydGljaXBhbnQgS1AgYXMgT1JZIEtyYXRvcyBQdWJsaWMgQVBJXG4gIHBhcnRpY2lwYW50IEtBIGFzIE9SWSBLcmF0b3MgQWRtaW4gQVBJXG5cbiAgQi0-PitBOiBHRVQgLy5vcnkva3JhdG9zL3B1YmxpYy9zZWxmLXNlcnZpY2UvYnJvd3Nlci9mbG93cy8obG9naW58cmVnaXN0cmF0aW9uKVxuICBBLT4-K0tQOiBHRVQgL3NlbGYtc2VydmljZS9icm93c2VyL2Zsb3dzL2xvZ2luKGxvZ2lufHJlZ2lzdHJhdGlvbilcbiAgS1AtLT4-S1A6IEV4ZWN1dGUgSG9va3MgZGVmaW5lZCBpbiBcIkJlZm9yZSBMb2dpbi9SZWdpc3RyYXRpb25cIlxuICBLUC0tPj4tQTogSFRUUCAzMDIgRm91bmQgL2F1dGgvKGxvZ2lufHJlZ2lzdHJhdGlvbik_cmVxdWVzdD1hYmNkZVxuICBBLS0-Pi1COiBIVFRQIDMwMiBGb3VuZCAvYXV0aC8obG9naW58cmVnaXN0cmF0aW9uKT9yZXF1ZXN0PWFiY2RlXG5cbiAgQi0-PitBOiBHRVQgL2F1dGgvKGxvZ2lufHJlZ2lzdHJhdGlvbik_cmVxdWVzdD1hYmNkZVxuICBBLT4-K0tBOiBHRVQvc2VsZi1zZXJ2aWNlL2Jyb3dzZXIvZmxvd3MvcmVxdWVzdHMvKGxvZ2lufHJlZ2lzdHJhdGlvbik_cmVxdWVzdD1hYmNkZVxuICBLQS0-Pi1BOiBTZW5kcyBMb2dpbi9SZWdpc3RyYXRpb24gUmVxdWVzdCBKU09OIFBheWxvYWRcbiAgTm90ZSBvdmVyIEEsS0E6ICB7XCJtZXRob2RzXCI6e1wicGFzc3dvcmRcIjouLi4sXCJvaWRjXCI6Li59fVxuICBBLS0-PkE6IEdlbmVyYXRlIGFuZCByZW5kZXIgSFRNTFxuICBBLS0-Pi1COiBSZXR1cm4gSFRNTCAoRm9ybSwgLi4uKVxuXG4gIEItLT4-QjogRmlsbCBvdXQgSFRNTFxuXG4gIEItPj4rS1A6IFBPU1QgSFRNTCBGb3JtXG4gIEtQLS0-PktQOiBDaGVja3MgbG9naW4gLyByZWdpc3RyYXRpb24gZGF0YVxuXG5cbiAgYWx0IExvZ2luIGRhdGEgaXMgdmFsaWRcbiAgICBLUC0tPj4tS1A6IEV4ZWN1dGUgSm9icyBkZWZpbmVkIGluIFwiQWZ0ZXIgTG9naW4gV29ya2Zsb3cocylcIlxuICAgIEtQLS0-PkE6IEhUVFAgMzAyIEZvdW5kIC9kYXNoYm9hcmRcbiAgICBOb3RlIG92ZXIgS1AsQjogU2V0LUNvb2tpZTogYXV0aF9zZXNzaW9uPS4uLlxuICAgIEItPj4rQTogR0VUIC9kYXNoYm9hcmRcbiAgICBBLS0-S0E6IFZhbGlkYXRlcyBTZXNzaW9uIENvb2tpZVxuICAgIEEtPj4tQjogU2VuZCBEYXNoYm9hcmQgUmVzcG9uc2VcbiAgZWxzZSBMb2dpbiBkYXRhIGlzIGludmFsaWRcbiAgICBOb3RlIG92ZXIgS1AsQjogVXNlciByZXRyaWVzIGxvZ2luIC8gcmVnaXN0cmF0aW9uXG4gICAgS1AtLT4-QjogSFRUUCAzMDIgRm91bmQgL2F1dGgvKGxvZ2lufHJlZ2lzdHJhdGlvbik_cmVxdWVzdD1hYmNkZVxuICBlbmRcbiAgIiwibWVybWFpZCI6eyJ0aGVtZSI6Im5ldXRyYWwiLCJzZXF1ZW5jZURpYWdyYW0iOnsiZGlhZ3JhbU1hcmdpblgiOjE1LCJkaWFncmFtTWFyZ2luWSI6MTUsImJveFRleHRNYXJnaW4iOjEsIm5vdGVNYXJnaW4iOjEwLCJtZXNzYWdlTWFyZ2luIjo1NSwibWlycm9yQWN0b3JzIjp0cnVlfX0sInVwZGF0ZUVkaXRvciI6ZmFsc2V9)

### Client-Side Browser Applications

Because Client-Side Browser Applications do not have access to ORY Kratos' Admin
API, they must use the ORY Kratos Public API instead. The flow for a Client-Side
Browser Application is almost the exact same as the one for Server-Side
Applications, with the small difference that
`https://127.0.0.1:4455/.ory/kratos/public/auth/browser/requests/login?request=abcde`
would be called via AJAX instead of making a request to
`https://kratos:4434/auth/browser/requests/login?request=abcde`.

::: Note To prevent brute force, guessing, session injection, and other attacks,
it is required that cookies are working for this endpoint. The cookie set in the
initial HTTP request made to
`https://127.0.0.1:4455/.ory/kratos/public/auth/browser/login` MUST be set and
available when calling this endpoint! :::

## Self-Service User Login and User Registration for API Clients

Will be addressed in a future release.

## Hooks

ORY Kratos allows you to configure hooks that run before and after a Login or
Registration Request is generated. This may be helpful if you'd like to restrict
logins to IPs coming from your internal network or other logic.

For more information about hooks please read the
[Hook Documentation](../hooks/index.md).
