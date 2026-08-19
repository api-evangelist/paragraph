# Paragraph auth.md

You are an agent registering to use Paragraph on behalf of a user. Paragraph supports a user-claimed registration flow. The user must sign in, select a publication, and approve access before Paragraph issues a credential.

Do not try to approve the request for the user. Do not expose the returned credential in logs, messages, or source code.

## Discover

- Protected resource metadata: https://paragraph.com/.well-known/oauth-protected-resource
- Authorization server metadata: https://paragraph.com/.well-known/oauth-authorization-server
- API resource: https://public.api.paragraph.com/api
- Supported identity type: `anonymous`, followed by a user claim
- Credential type: `api_key`
- Credential transport: `Authorization: Bearer <api-key>`

Paragraph API keys grant read and write access to the publication the user selects during approval. The keys are not currently fine-grained.

## Register

Create a short-lived registration session:

```http
POST https://public.api.paragraph.com/api/v1/api/auth/sessions
Content-Type: application/json

{
  "deviceName": "A short name the user will recognize"
}
```

`deviceName` is optional. You may also include an optional `callbackUrl` if its HTTPS host is accepted by the service.

The response is:

```json
{
  "sessionId": "<session-id>",
  "verificationUrl": "https://paragraph.com/api/auth?session=<session-id>",
  "expiresAt": "<ISO-8601 timestamp>"
}
```

## Claim

Show `verificationUrl` to the user and ask them to open it. The user signs in or creates an account, selects a publication, and approves access. Always use the returned `verificationUrl`; do not construct the URL yourself.

Registration sessions expire after five minutes. If the session expires, start again with a new registration request.

## Poll for the credential

Poll no more often than every five seconds:

```http
GET https://public.api.paragraph.com/api/v1/api/auth/sessions/<session-id>
```

While the user has not approved the request, the response has `status: "pending"`. Stop polling if the response has `status: "expired"`.

After approval, Paragraph returns the API key once:

```json
{
  "status": "completed",
  "apiKey": "<api-key>"
}
```

Store the API key securely.

## Use the credential

Send the API key in the Authorization header when calling protected endpoints under `https://public.api.paragraph.com/api`:

```http
Authorization: Bearer <api-key>
```

Read the API documentation at https://docs.paragraph.com/developers before making requests.

## Revoke

Ask the user to revoke an API key in Paragraph under Settings, Publication, Developer. Revocation takes effect immediately.
