# IE Legacy Helper

> A lightweight, privacy-conscious Chrome extension that helps selected legacy web applications work in modern Chrome, where practical.

Legacy web applications may rely on outdated browser APIs, event models, DOM behaviors, CSS quirks, or browser identification. **IE Legacy Helper** provides a small, configurable compatibility layer to help address specific compatibility gaps without attempting to recreate Internet Explorer or another browser engine.

**IE Legacy Helper is not an IE emulator.** Chrome continues to use Chromium. Compatibility features are targeted shims and workarounds for specific, verified requirements.

## Goals

- Help selected legacy websites work in modern Chrome where practical.
- Keep the extension lightweight, dependency-free, and easy to inspect and maintain.
- Apply compatibility behavior only to explicitly configured websites or through a clearly user-initiated workflow.
- Use small, isolated, documented shims rather than a broad compatibility framework.
- Minimize permissions and avoid collecting or transmitting browsing data.
- Make compatibility fixes modular so they can be enabled, disabled, tested, and removed independently.
- Be useful to individuals, teams, and organizations without requiring any particular company or network environment.

## Non-goals

- Reimplement Internet Explorer 11's rendering engine or browser internals.
- Provide ActiveX, VBScript, Silverlight, or other unsupported plugin runtimes.
- Guarantee compatibility with every IE-specific API or behavior.
- Modify arbitrary websites by default or inject scripts broadly without a user/configuration gate.
- Bypass authentication, browser protections, TLS restrictions, or website security controls.
- Replace proper modernization of legacy applications where source changes are possible.

## Design principles

1. **Lightweight:** no framework, bundler, external runtime, or unnecessary dependency by default.
2. **Least privilege:** request only permissions needed for implemented features.
3. **Explicit scope:** restrict compatibility to configured origins or an appropriate user-initiated scope.
4. **Modular:** one shim per API or behavior, with clear ownership and tests.
5. **Fail safely:** do not silently alter unrelated pages; avoid installing a shim if prerequisites are not met.
6. **Maintainable:** readable code, documentation, tests, and predictable structure.
7. **Privacy-first:** no telemetry, analytics, remote code, or extension-initiated external network calls by default.
8. **Reversible:** users can disable compatibility without permanent modification of the target site.

## Platform and technology

- Browser: Chrome (supported stable versions to be documented for each release)
- Extension platform: Manifest V3
- Language: plain JavaScript
- UI: minimal HTML and CSS; popup and options page only as needed
- Runtime dependencies: none by default
- Build tooling: none initially; keep source directly loadable in Chrome

Avoid TypeScript, React, Vue, Angular, large polyfill packages, and bundlers unless a documented technical need justifies introducing them.

## Compatibility approach

Compatibility is implemented as focused shims. Each shim should:

- Have one clearly defined purpose.
- Run only within its configured scope.
- Avoid replacing native implementations unless necessary and documented.
- Detect unsupported or incompatible situations and fail safely.
- Include tests or reproducible manual test steps.
- Document known differences from the original browser behavior.

### Candidate modules

This list is a backlog, not a commitment to implement every feature.

| Module | Purpose | Priority | Notes |
|---|---|---|---|
| Event compatibility | Selected legacy event patterns such as `attachEvent` | Based on verified need | Carefully map event names and listener behavior |
| Event object compatibility | Selected properties such as `srcElement`, `returnValue`, `cancelBubble` | Based on verified need | Native behavior differs; test edge cases |
| AJAX compatibility | Help replace specific legacy request patterns where feasible | Case-by-case | Consider CORS and page-context limitations |
| `XDomainRequest` shim | Assist specific legacy cross-origin request patterns | Low / case-by-case | Cannot reproduce IE security and network semantics fully |
| DOM compatibility | Selected legacy DOM APIs or behavior | Case-by-case | Avoid broad DOM proxying |
| CSS workarounds | Specific documented layout/rendering issues | Case-by-case | Prefer scoped styles, not global styles |
| User-Agent workaround | Address legacy UA checks where required | Case-by-case | UA spoofing alone does not provide IE APIs |

### Limitations

The extension cannot reproduce the full IE11 environment. ActiveX, VBScript, IE's rendering engine, certain security models, proprietary plugins, and deep browser internals are outside the practical scope. A shim may also be unable to intercept an API when the page executes before the shim or runs in a restricted context.

## Architecture

Initial components:

- **Manifest:** extension metadata, permissions, popup/options, and injection configuration.
- **Service worker:** handles extension events and tab-level actions; do not assume it remains continuously running.
- **Popup:** minimal controls and status for the active tab.
- **Compatibility modules:** injected scripts or styles, only when required.
- **Configuration:** origin rules and per-site module selection.
- **Tests:** unit tests where practical and manual regression tests against representative pages.

### Execution contexts

Chrome content scripts normally execute in an isolated world and do not share the page's JavaScript global scope. Shims that need to alter page-visible APIs may require injection into the page's MAIN world using supported Chrome mechanisms. Document the execution context and timing requirements for each shim. Do not assume that assigning a property in a content script modifies the page's own global object.

## Suggested repository structure

```text
ie-legacy-helper/
├── README.md
├── LICENSE
├── SECURITY.md
├── CONTRIBUTING.md
├── .gitignore
├── manifest.json
├── src/
│   ├── background/
│   │   └── service-worker.js
│   ├── popup/
│   │   ├── popup.html
│   │   ├── popup.css
│   │   └── popup.js
│   ├── options/
│   │   ├── options.html
│   │   ├── options.css
│   │   └── options.js
│   ├── content/
│   │   └── content-script.js
│   ├── compatibility/
│   │   ├── event/
│   │   ├── ajax/
│   │   ├── dom/
│   │   └── css/
│   ├── config/
│   └── shared/
├── tests/
│   ├── unit/
│   └── manual/
│       └── test-plan.md
└── docs/
    ├── ARCHITECTURE.md
    ├── PRIVACY.md
    └── compatibility-modules.md
```

Keep the structure flexible. Do not create empty directories or abstractions until they are needed. This is a guide, not a requirement to scaffold every file immediately.

## Website scope and configuration

Compatibility should not run indiscriminately on every website.

Recommended initial model:

- Maintain an origin allowlist or user-managed site configuration.
- Associate each configured origin with only the modules it needs.
- Where possible, use user-initiated access rather than broad persistent host access.
- Avoid wildcard host permissions unless a feature genuinely requires them and the need is documented.
- Do not enable compatibility on public sites by default.

Example configuration (illustrative only):

```js
const siteRules = [
  {
    id: "example-site",
    origins: ["https://example.com"],
    modules: ["attach-event"],
    enabledByDefault: false
  }
];
```

`example.com` is a placeholder. Users should configure their own target sites. Do not commit credentials, tokens, private user data, or sensitive URL paths to configuration or documentation.

## Permissions and security

Security matters because this extension may be used with sites containing sensitive information.

### Permission policy

- Request only permissions required by implemented features.
- Prefer narrow host permissions or user-initiated access over broad access to all websites.
- Avoid `tabs`, `webRequest`, `cookies`, `history`, `downloads`, and other sensitive permissions unless a concrete requirement is documented and reviewed.
- Use `activeTab` where a user-initiated, temporary access model is sufficient.
- Review permission changes before merging.

### Security requirements

- Do not load or execute code from remote URLs.
- Do not use remote scripts, CDN libraries, dynamic code evaluation, `eval`, or `new Function`.
- Do not collect browsing history, page contents, form data, credentials, or personal data.
- Do not transmit page data or extension settings to external servers.
- Do not log sensitive page content or user information.
- Restrict injection to intended origins and contexts.
- Treat all page content as untrusted; never use page data as executable code.
- Keep the service worker and injected scripts small and reviewable.
- Use a restrictive Content Security Policy compatible with Manifest V3.
- Do not attempt to bypass website or organizational security policies.

### Security checklist

- [ ] Permissions are minimal and justified.
- [ ] Site scope is restricted and understandable.
- [ ] No remote executable code or unnecessary dependencies.
- [ ] No telemetry or unexpected extension network requests.
- [ ] Sensitive page data is not logged or stored.
- [ ] Injection is limited to intended origins, tabs, and contexts.
- [ ] Each shim's behavior and limitations are documented.

## Privacy

IE Legacy Helper is designed to operate locally. The project should not collect or transmit browsing data, page content, form data, credentials, or telemetry. If a future feature changes this behavior, it must be reviewed, documented, and reflected in the privacy documentation before release. See [docs/PRIVACY.md](docs/PRIVACY.md).

## Coding standards

- Use plain, readable JavaScript.
- Prefer small functions and explicit control flow.
- Avoid clever metaprogramming and broad monkey-patching.
- Avoid global namespace pollution; use a module or well-defined wrapper where practical.
- When patching a global API, preserve native behavior where possible and check whether the API already exists.
- Explain why a compatibility patch is needed in comments, not merely what it does.
- Avoid unnecessary abstractions and configuration complexity.
- Keep UI accessible, simple, and usable without a framework.
- Use English for source code, comments, identifiers, and documentation.
- Do not include secrets, private data, or proprietary code in the repository.

### Shim implementation pattern

1. Check whether the shim is needed.
2. Check whether the target API and prerequisites exist.
3. Install the shim only if safe and relevant.
4. Match only the documented subset of behavior required.
5. Avoid exposing unnecessary globals.
6. Provide a test or manual verification procedure.

Illustrative example only:

```js
(function () {
  if (typeof window.attachEvent === "function") {
    return;
  }

  // Implement only the documented subset needed by the target application.
  // This is not a complete IE emulator.
})();
```

## Development workflow

### Load locally in Chrome

1. Clone the repository.
2. Open `chrome://extensions`.
3. Enable **Developer mode**.
4. Click **Load unpacked**.
5. Select the project directory containing `manifest.json`.
6. Test on a page you are authorized to use and have configured for compatibility.
7. After changes, reload the extension and refresh the test page. Inspect the service worker from the Extensions page if needed.

### Development rules

- Work on one compatibility module at a time.
- Reproduce the issue before implementing a fix.
- Record the Chrome version and expected/actual behavior.
- Test with compatibility disabled and enabled.
- Test reloads, navigation, and new tabs where relevant.
- Prefer safe test environments and synthetic data.
- Never include proprietary site details in public issues, screenshots, test fixtures, or pull requests.

## Testing

### Unit tests

Test compatibility modules independently where practical. The initial project may use a simple test harness without introducing a heavy testing framework.

### Manual regression testing

For each configured test site, verify:

- Page loading and navigation.
- Authentication flows are not disrupted (use authorized test accounts).
- Forms and validation behave as expected.
- Buttons, events, popups, and dialogs work.
- AJAX/API operations work within the site's security constraints.
- Downloads, printing, or other workflows work if required.
- Disabling compatibility returns the page to normal Chrome behavior as far as the shim allows.
- No unrelated websites are affected.

| Scenario | Expected result |
|---|---|
| Configured site, compatibility disabled | Normal Chrome behavior |
| Configured site, compatibility enabled | Only selected shims are applied |
| Unconfigured site | No compatibility injection |
| Unsupported API/module | Safe failure; diagnostic status where practical |
| Extension reload | Configuration and behavior remain consistent |
| Page reload/navigation | Scope is correctly applied |

## Debugging

- Inspect the page with Chrome DevTools.
- Inspect isolated content scripts separately from MAIN world scripts.
- Inspect the service worker through `chrome://extensions`.
- Use prefixed debug messages only when debug logging is enabled.
- Never log page contents, credentials, tokens, or sensitive business data.
- Reproduce issues with minimal, synthetic test pages where possible.

## Roadmap

### Phase 0 — Requirements and investigation

- [ ] Identify representative legacy applications and use cases.
- [ ] Document the required browser APIs, behaviors, and errors using sanitized descriptions.
- [ ] Check whether requirements depend on ActiveX, VBScript, plugins, or other out-of-scope features.
- [ ] Establish a test environment and safe test data.

### Phase 1 — Minimal extension foundation

- [ ] Create Manifest V3 manifest.
- [ ] Create a minimal service worker.
- [ ] Create a basic popup with active-tab status.
- [ ] Define narrow site scope and injection strategy.
- [ ] Add one shim for a confirmed compatibility issue.
- [ ] Review permissions and security posture.

### Phase 2 — Compatibility modules

- [ ] Implement additional shims only for verified requirements.
- [ ] Add tests or reproducible manual steps per shim.
- [ ] Document limitations and supported behavior.
- [ ] Run regression testing.

### Phase 3 — Configuration and usability

- [ ] Add per-site configuration if multiple sites require different shims.
- [ ] Add options UI only when configuration cannot remain simple.
- [ ] Add status and troubleshooting information.
- [ ] Consider configuration import/export only if genuinely needed; review imported data as untrusted.

### Phase 4 — Release and maintenance

- [ ] Review permissions and code before each release.
- [ ] Test on supported Chrome versions.
- [ ] Maintain a compatibility list using sanitized test cases and descriptions.
- [ ] Remove obsolete shims when the underlying sites are updated.

## Verified compatibility matrix

Record verified APIs or behaviors—not private site names, URLs, screenshots, internal workflow details, or company information.

| API / behavior | Status | Chrome version tested | Test case / notes |
|---|---|---|---|
| `attachEvent` | Not tested | TBD | Sanitized test case |
| `window.event` subset | Not tested | TBD | Sanitized test case |
| `XDomainRequest` subset | Not tested | TBD | Sanitized test case |

Suggested status values: `Not tested`, `Investigating`, `In progress`, `Verified`, `Known limitation`, `Not supported`.

## Definition of done

- [ ] The shim addresses a verified and documented requirement.
- [ ] Its scope is limited to the intended origin or user-enabled tab.
- [ ] It is modular and documented.
- [ ] Tests or manual verification steps are provided.
- [ ] No unnecessary permission is introduced.
- [ ] Security considerations are reviewed.
- [ ] Regression tests are completed for affected workflows.
- [ ] Known limitations are documented.
- [ ] Examples, screenshots, and test artifacts are sanitized and contain no private data.

## References

- [Chrome Extensions Manifest V3 migration guide](https://developer.chrome.com/docs/extensions/develop/migrate)
- [Chrome Extensions documentation](https://developer.chrome.com/docs/extensions)
- [Chrome scripting API](https://developer.chrome.com/docs/extensions/reference/api/scripting)
- [Chrome permissions](https://developer.chrome.com/docs/extensions/develop/concepts/declare-permissions)

---

**Project status:** Planning / requirements gathering  
**Initial strategy:** Build a minimal, permission-conscious compatibility layer for verified legacy web requirements. Do not attempt to recreate IE11 as a browser.
