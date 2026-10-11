# TechSparks Headless Doctor — Alpha Releases

TechSparks Headless Doctor is an early-alpha diagnostic WordPress plugin for troubleshooting headless WordPress stacks.

> **Warning:** This is early testing software intended for staging and development environments. It is not yet recommended for production use. Review diagnostic output before sharing it publicly.
>
> Never provide passwords, Application Passwords, API keys, cookies, tokens, private keys, or other credentials in bug reports.

## Current requirements

- WordPress
- PHP 8.2+
- Administrator access to configure Headless Doctor

## Basic installation

1. Download the latest plugin ZIP from [GitHub Releases](https://github.com/jbphinney/techsparks-headless-doctor-releases/releases).
2. In WordPress Admin, go to **Plugins → Add New → Upload Plugin**.
3. Upload the ZIP.
4. Activate **TechSparks Headless Doctor**.
5. Open **Settings → Headless Doctor**.
6. Configure the frontend URL.

## Included in this alpha

- WordPress REST diagnostics
- REST-index validation
- WPGraphQL availability detection
- Pretty-permalink diagnostics
- Application Password capability detection
- Sanitized diagnostic exports
- Browser-side frontend REST probing
- Backend/frontend correlation
- REST observability across WordPress and browser vantages
- Copy diagnostic JSON
- Downloadable frontend integration bundle
- Page Workload with a controlled page load
- Navigation Timing and Resource Timing evidence
- Copy workload JSON
- Conservative REST activity classification
- GraphQL endpoint activity classification when endpoint discovery is available
- Request Inspector: one anonymous GET to a REST route, with browser timing next to WordPress's numbers
- Headless Lab: REST requests from the frontend's origin, with saved runs you can compare

How to use it and how to read the result cards: [USER-GUIDE.md](USER-GUIDE.md).

## Frontend integration and Page Workload

The frontend integration bundle is available from **Settings → Headless Doctor → Download Frontend Integration**. It includes the static files required by the current tested integration and the generated site-specific manifest. Deploy the bundle to the configured frontend origin using your normal deployment process.

Page Workload performs one controlled page load and observes a bounded browser timing window. It reports browser-visible resource timing entries such as scripts, stylesheets, images, fonts, and network resources. REST and GraphQL activity are conservative classifications of that resource subset; GraphQL classification is available only when the endpoint can be established safely.

Page Workload does not inspect request bodies or GraphQL documents or replay requests. Missing browser visibility is reported as unavailable or limited rather than treated as a site defect.

Do not assume compatibility with a particular frontend framework, hosting platform, or deployment model without testing it in your environment.

## What feedback is useful?

- WordPress version
- PHP version
- Frontend framework and version
- Hosting platform
- Whether REST or GraphQL is used
- Exact Headless Doctor result or message code
- Sanitized safe report
- Reproduction steps

Remove secrets and other sensitive information before posting anything.

## Reporting Issues

Please use the [Issues tab](https://github.com/jbphinney/techsparks-headless-doctor-releases/issues) of this public releases repository for alpha feedback.
