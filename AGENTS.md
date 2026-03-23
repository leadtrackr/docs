# Documentation project instructions

## About this project

- This is the documentation site for [LeadTrackr](https://leadtrackr.io), built on [Mintlify](https://mintlify.com)
- Pages are MDX files with YAML frontmatter
- Configuration lives in `docs.json`
- Run `mint dev` to preview locally
- Run `mint broken-links` to check links
- The main application codebase lives in a sibling repo (`leadtrackr`). API route handlers are in `src/app/api/leads/`

## Terminology

- Use "project" (not "workspace" or "account") when referring to a LeadTrackr project
- Use "lead" (not "contact" or "submission") for tracked form submissions
- Use "conversion label" (not "status label" or "conversion action") for the user-defined pipeline stages
- Use "API key" (not "token" or "secret") when referring to the `X-API-Key` header value

## Style preferences

- Use active voice and second person ("you")
- Keep sentences concise — one idea per sentence
- Use sentence case for headings
- Bold for UI elements: Click **Settings**
- Code formatting for file names, commands, paths, and code references

## Content boundaries

### Excluded body parameters (DO NOT document these)

The following fields exist in the API route code but are **internal-only**. They must never appear in API documentation, request examples, or parameter lists:

- `platform` — internal field, automatically determined by the endpoint
- `createdBy` — internal field, set automatically to the system default user
- `status` — on create endpoints only; leads always start as `"open"`

### Excluded response fields (DO NOT include in response examples)

These fields exist on the database lead record but are **not returned** by the public API (filtered by `formatLeadForApi`):

- `platform`
- `createdBy`
- `active`
- `finished`
- `processingWarnings`
- `deviceData` (accepted as input but not returned in responses)
- `updatedAt`
- `uniqueEventId` (internal dedup field, not exposed in responses)

### formData documentation

- Only document `formName` and `uniqueEventId` as named sub-properties of `formData`
- Do not document `formCustomField1` through `formCustomField5` as named fields
- Instead, note that additional custom fields in `formData` are passed through as-is

### Field aliases

- Document `channelFlow` as the primary field name. Mention `lt_channelflow` as an accepted alias
- Document `uniqueIdentifier` as the primary field name. Mention `unique_identifier` as an accepted alias

### Request example data

- Use realistic but clearly fake data in all examples
- Standard example values:
  - Project ID: `"your-project-id"`
  - Email: `john@example.com`, `jane@example.com`
  - Phone: `+31612345678`, `+31687654321`
  - Names: John Doe, Jane Smith
  - Company: Acme Inc.
  - uniqueIdentifier: `crm-lead-456`, `crm-lead-789`
  - gclid: `abc123`, `xyz789`
  - cid: `1234567890.1234567890`
  - conversionId (UUID): `a1b2c3d4-e5f6-7890-abcd-ef1234567890`
  - uniqueEventId: `evt_abc123`
- Never use real project IDs, actual user data, or API keys in examples

## Authentication patterns

Each endpoint has a specific auth pattern. Follow this when documenting:

| Endpoint | Auth method | Notes |
|---|---|---|
| `createLead` | None | Public endpoint for client-side use. Project identified by `projectId` in body |
| `createServerSideLead` | `X-API-Key` header | Server-side only. Also requires `projectId` in body |
| `createCTLead` | `X-API-Key` header | Call tracking endpoint. Also requires `projectId` in body |
| `getLead` | `X-API-Key` header | Project resolved from the API key directly |
| `getLeads` | `X-API-Key` header | Project resolved from the API key directly |
| `updateLeadStatus` | `X-API-Key` header | Project resolved from the API key directly |

## Lead lookup identifiers

When an endpoint supports looking up a lead by identifier, the priority order is always:

1. `leadId` (numeric, internal database ID)
2. `uniqueIdentifier` (user-provided external ID)
3. `email`
4. `phone`

Only the highest-priority match is used. For email and phone lookups, the newest lead (most recent `createdAt`) is returned when multiple matches exist.

## API endpoint structure

- All public lead APIs live under `/api/leads/`
- `getLead` and `getLeads` use `GET` with query parameters for all filters and identifiers (no path parameters)
- `createLead`, `createServerSideLead`, `createCTLead`, and `updateLeadStatus` use `POST` with JSON body
- All responses include CORS headers
- All error responses have a `message` field
