# Akeyless API Postman Collection

A complete [Postman](https://www.postman.com/) collection for the
[Akeyless](https://www.akeyless.io/) REST API. Every one of the 622 v2 API
operations is present as a ready-to-run request, grouped into domain folders,
with the session token wired in automatically and each command documented down
to the field level. The file holds 627 requests in total: the 622 generated
operations plus 5 hand-authored auth examples that capture a session token.

The audience is engineers integrating with Akeyless over the API directly:
platform teams automating onboarding, application developers wiring secret
retrieval into a service, and anyone evaluating the API surface before writing
code against the SDK.

## What problem this solves

Teams adopting the Akeyless API usually start from the per-command reference
pages and assemble requests by hand: copy a URL, work out the auth model,
re-read each parameter, and rebuild the token plumbing in every request. That
is slow, it is easy to get the SaaS-versus-gateway boundary wrong, and there is
no single artifact a developer can import and run.

This collection moves that work into one importable file.

| Concern | Where it lives now |
|---|---|
| Knowing which endpoints exist | All 622 v2 operations, grouped into 14 domain folders. |
| Authenticating | Ready-to-run auth requests for api-key, universal identity, JWT/OIDC, cert, AWS IAM, and Kubernetes each capture the session token into `{{token}}`. SAML and interactive OIDC stay out-of-band. Every other request reuses `{{token}}` with no manual copy-paste. |
| Understanding a command | Each request carries its command description, a link to the official reference page, and every field documented with type, enum, default, and meaning. |
| Knowing what mutates | Every state-changing request is tagged `[W]` in its name. |
| SaaS versus gateway | `base_url` is a single variable. Folder descriptions state which families need a gateway. |

## Quick start

Every request authenticates with a session token held in the `{{token}}`
variable. How you obtain that token depends on your auth method.

1. Import `akeyless-api.postman_collection.json` into Postman: **Import** then
   select the file.
2. Open the collection's **Variables** tab and set `base_url`:
   `https://api.akeyless.io` for SaaS, or
   `https://<your-gateway-host>:8000/api/v2` for a gateway.
3. Set `{{token}}`. Pick the path that matches your auth method:

   **Ready-to-run auth requests** (recommended): the Auth & Auth Methods folder
   has runnable requests for api-key, universal identity, JWT/OIDC, cert, AWS
   IAM, and Kubernetes. Fill the collection variables that request references,
   run it, and its test script writes the returned token into `{{token}}` for
   you. For API key, set `access_id` and `access_key`, then run
   **Auth & Auth Methods > auth (api-key)**.

   **SAML, interactive OIDC, or any method you prefer to handle yourself**: get
   a token out of band and paste it into `{{token}}`. From the CLI, `akeyless
   auth` prints a token (`t-...`); for SAML or OIDC it opens a browser to
   complete sign-in. You can also copy a token from the Akeyless web console.
   This path works for every auth method.
4. Run any other request. Each one passes `{{token}}` in its body
   automatically. Tokens expire after roughly 60 minutes; paste a fresh one or
   re-run `auth` to refresh.

Why a token and not always access_id/access_key: only the `api_key` auth method
uses a static access key. SAML, OIDC, LDAP, cert, and Kubernetes auth produce a
token through a sign-in or platform handshake, so the portable input across all
methods is the token itself.

## What is in the collection

627 requests across 14 folders. The `[W]` column counts mutating requests:
those that create, update, delete, set, or rotate state.

| Folder | Requests | `[W]` mutating |
|---|---|---|
| Auth & Auth Methods | 67 | 56 |
| Secrets & Items | 17 | 8 |
| Roles & RBAC | 10 | 7 |
| Targets | 133 | 124 |
| Dynamic Secrets | 65 | 57 |
| Rotated Secrets | 50 | 49 |
| Gateway | 117 | 94 |
| Kubernetes | 2 | 0 |
| Secure Access, PKI & Certificates | 19 | 7 |
| Encryption & Classification | 24 | 1 |
| Universal Secrets Connector | 5 | 0 |
| Event Center & Forwarders | 19 | 14 |
| Account & Settings | 14 | 2 |
| Other | 85 | 27 |
| **Total** | **627** | **446** |

## How each request is documented

Every request's description is built from three layers, so a developer who has
never seen the API can run it without leaving Postman.

| Layer | Source | Coverage |
|---|---|---|
| Command description | The one-line summary from the Akeyless CLI for that command. | 562 of 622 (90%) |
| Reference link | A direct URL to the command's page on `docs.akeyless.io`. | 617 of 622 (99%) |
| Field documentation | Every field with required/optional, type, enum values, default, and meaning, from the API spec. | All 622 |

These coverage figures describe the 622 generated operations. The five
hand-authored auth example requests are additions layered on top, documented by
hand.

A request description reads like this:

```
## Creates a new static secret item
_Command description from the Akeyless CLI._

`POST /create-secret`  -  MUTATING (changes state)
Reference: https://docs.akeyless.io/reference/createsecret

Required: name, value
Fields:
- `name` (**required**, string): Secret name
- `value` (**required**, string): The secret value (relevant only for type 'generic')
- `format` (optional, string, default: text): Secret format [text/json/key-value]
- `type` (optional, string, default: generic): The secret sub type [generic/password]
...
```

## Authentication model

Akeyless authenticates per request by carrying the session token in the JSON
body field `token`, not in an `Authorization` header. Every request in this
collection references `{{token}}`, so the only thing you manage is that one
variable.

The collection provides runnable auth requests for the common methods: api-key,
universal identity, JWT/OIDC, cert, AWS IAM, and Kubernetes. Each one wires its
credentials to collection variables and carries a test script that writes the
returned token into `{{token}}`. The AWS IAM request additionally runs a
pre-request script that signs an STS GetCallerIdentity call with SigV4 and
builds the `cloud_id` the method expects.

How the token is produced depends on the auth method. The `api_key` method
exchanges a static `access_id` and `access_key` for a token. SAML and
interactive OIDC complete a browser sign-in and cannot be a plain POST, so for
those you obtain a token out of band and paste it into `{{token}}`. All methods
yield a token, which is why `{{token}}` is the portable input the collection is
built around.

## Public SaaS versus Gateway

`base_url` selects the endpoint. Most control-plane operations work against
public SaaS. Operations that produce or fetch runtime values, and gateway
configuration itself, require a running gateway and return an error against
SaaS. For a gateway, `base_url` must include the `/api/v2` suffix; the
collection states this requirement.

| Surface | `base_url` | What works |
|---|---|---|
| Public SaaS | `https://api.akeyless.io` | Auth, static secrets, roles, auth methods, target and producer definitions. |
| Gateway | `https://<your-gateway-host>:8000/api/v2` | Everything SaaS does, plus runtime fetches (`get-dynamic-secret-value`, `get-rotated-secret-value`) and `gateway-*` configuration. |

Each folder description notes its SaaS-versus-gateway behavior. Creating a
dynamic-secret or rotated-secret object succeeds on SaaS; producing its value
requires the gateway.

## Conventions

- **`[W]` in a request name** marks a mutating operation. Read-only requests
  carry no marker.
- **Request bodies contain required fields only**, expressed as `<placeholder>`
  values, so a request is runnable after you fill the placeholders. Optional
  fields are not in the body but are fully documented in the description. This
  holds for the generated operations; the six auth requests instead wire their
  credentials to collection variables (`{{...}}`).
- **Collection variables**, not hard-coded values, hold the base URL and
  credentials. The collection ships with no real credentials.
- **Batch endpoints** (`encrypt-batch`, `decrypt-batch`, and similar) take a
  JSON array body; their example body is an array of one line object.

## Repository layout

| Path | What it is |
|---|---|
| `akeyless-api.postman_collection.json` | The collection. Postman Collection format v2.1. Import this. |
| `README.md` | This document. |
| `LICENSE` | MIT. |

## Provenance

The collection is generated, not hand-maintained, so it stays faithful to the
API. The six runnable auth requests in the Auth & Auth Methods folder are
hand-authored, layered on top of the generated collection.

- **Endpoints, request schemas, field documentation, enums, and defaults** come
  from the official Akeyless OpenAPI specification (the v5 API SDK definition,
  OpenAPI 3.0.0, 622 operations).
- **Command-level descriptions** come from the Akeyless CLI's own help text,
  mapped to API commands.
- **Reference links** resolve to `docs.akeyless.io` and were each verified to
  return HTTP 200 against the live public documentation site.

The five commands without a reference link (`folder-sync`, `folder-sync-all`,
`folder-delete-sync`, `rotated-secret-create-hashi-vault`, and
`rotated-secret-update-hashi-vault`) are omitted from that layer because the
public documentation site has no reference page for them. They keep their
command description and full field documentation.

## Compatibility

| Component | Version |
|---|---|
| Akeyless API | v2 endpoints, v5 SDK definition |
| Postman collection format | v2.1 |
| Endpoint | SaaS (`api.akeyless.io`) or any Akeyless Gateway |

## Additional resources

- [Akeyless documentation](https://docs.akeyless.io)
- [Akeyless API reference](https://docs.akeyless.io/reference)
- [Authentication methods](https://docs.akeyless.io/docs/access-and-authentication-methods)
- [Akeyless CLI](https://docs.akeyless.io/docs/cli)

## License

MIT. See [LICENSE](LICENSE).
