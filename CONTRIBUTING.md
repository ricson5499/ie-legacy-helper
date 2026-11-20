# Contributing

Thank you for considering a contribution to IE Legacy Helper.

## Before contributing

- Check existing issues to avoid duplicate work.
- For significant changes, open an issue first to discuss the approach.
- Do not submit proprietary code, confidential information, internal URLs, credentials, private screenshots, or data from systems you are not authorized to share.
- Use sanitized, synthetic test pages and data wherever possible.

## Pull requests

- Keep changes focused and small.
- Explain the problem, the proposed behavior, and limitations.
- Include tests or reproducible manual verification steps.
- Document any new permissions and justify why they are required.
- Avoid new dependencies unless there is a clear need and they have been reviewed.
- Do not include remote executable code or telemetry.
- Follow the project's plain JavaScript and modular shim principles.

## Compatibility shim contributions

A shim should be narrow, opt-in or scoped, documented, and tested. It should not attempt to recreate the full IE environment. Include known limitations and a sanitized example that demonstrates the intended behavior.

## Style

- Use English for code, comments, identifiers, and documentation.
- Prefer straightforward, maintainable code.
- Explain why a patch exists, especially when modifying native APIs.
