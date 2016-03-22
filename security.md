
### Password Storage

Discourse uses the PBKDF2 algorithm to encrypt salted passwords. This algorithm is blessed by NIST. Security experts on the web [tend to agree that PBKDF2 is a secure choice](http://security.stackexchange.com/questions/4781/do-any-security-experts-recommend-bcrypt-for-password-storage).

**options you can customise in your production.rb file**

- `pbkdf2_algorithm`: the hashing algorithm used (default "sha256")
- `pbkdf2_iterations`: the number of iterations to run (default 64000)

### XSS

The main vector for [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting) attacks is via the post composer, as it allow users to enter Markdown, HTML (a safe subset thereof), and BBCode to format posts.

There are 2 main scenarios that are protected against:

1. **Markdown preview invokes an XSS.** This is possibly severe in one specific case: when a forum staff member edits a user's post, seeing the raw markup, where a malicious user may have inserted code to run JavaScript. This code would only show up in the preview, but it would run in the context of a forum staff member, which is *very* bad.

2. **Markdown displayed on the page invokes an XSS.** To protect against client side preview XSS, Discourse uses [Google Caja](https://developers.google.com/caja/) in the preview window.

On the server side they run a whitelist based sanitizer, implemented using the [Sanitize gem](https://github.com/rgrove/sanitize). See the [relevant Discourse code](https://github.com/discourse/discourse/blob/master/lib/pretty_text.rb).

In addition, titles and all other places where non-admins can enter code are protected either using the Handlebars library or standard Rails XSS protection.

### CSRF

[CSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery) allows malicious sites to perform HTTP requests in the context of a forum user without their knowledge -- mostly by getting users who already hold a valid forum login cookie to click a specific link in their web browser.

Discourse extends the built-in Rails CSRF protection in the following ways:

1. By default any non GET requests ALWAYS require a valid CSRF token. If a CSRF token is missing Discourse will raise an exception.

2. API calls using the secret API bypass CSRF checks.

3. Certain pages are "cachable", we do not render the CSRF token (`<meta name='csrf-token' ...`) on any cachable pages. Instead when users are about to perform the first non GET request they retrieve the token just in time via `GET session/csrf`

### DDOS

If you install via our recommended Docker image in our [install guide][ig], nginx is the front end web server. For additional DDOS protection we recommend placing [HAProxy](http://haproxy.1wt.eu/) in front.

### Deployment concerns

They strongly recommend that the various Discourse processes (web server, sidekiq) run under a non-elevated account. This is handled automatically if you install via their recommended Docker image -- see [the install guide][ig] for details.

[ig]: https://github.com/discourse/discourse/blob/master/docs/INSTALL.md


