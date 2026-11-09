# Security policy

CredX is an experimental credit-risk monitoring project. It is not a production banking system and must not be used to make automated lending decisions.

## Supported versions

Security fixes are considered for the current `main` branch. No separate release or long-term-support branches are currently published.

## Reporting a vulnerability

Please do not publish exploitable vulnerability details, credentials, personal information, or customer financial data in a public issue or pull request.

Contact the repository maintainer through the private contact options on the maintainer's GitHub profile: https://github.com/Tirumala2824

Include, where safe:

- A concise description of the affected component and impact.
- The commit or revision and environment where the issue was observed.
- Minimal reproduction steps that use synthetic data.
- A suggested mitigation, if known.

Do not include live tokens, secrets, production records, or unredacted logs. Allow the maintainer a reasonable opportunity to investigate and coordinate a fix before public disclosure.

## Security boundaries

The current implementation has documented limitations, including SQLite persistence constraints, incomplete production authorization/security controls, and browser token storage that is not appropriate for a sensitive production banking application. Review the root README and backend deployment documentation before exposing any deployment to users.

Use synthetic or properly authorized test data only.
