# Privacy

IE Legacy Helper is designed to operate locally in the browser.

## Data handling principles

The project should not:

- Collect or transmit browsing history, page contents, form data, credentials, or personal data.
- Send telemetry, analytics, or extension settings to external servers.
- Log sensitive page content or user information.
- Load executable code from remote sources.

Compatibility shims may execute in a target page context when required to address a browser compatibility issue. This is why scope and permissions must remain narrow and clearly explained.

## Future changes

Any feature that changes these principles must undergo security and privacy review, be clearly documented, and be reflected in the extension's disclosures before release.

## User configuration

Users should configure only sites they are authorized to access. Do not publish private site URLs, credentials, screenshots, or page data in public configuration, issues, tests, or documentation.
