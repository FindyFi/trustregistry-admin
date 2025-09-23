# Trust Registry Admin

A JavaScript client library for administrating an OpenID Federation server. This library provides a comprehensive API client for managing trust registries, including accounts, keys, metadata, subordinates, and trust marks.

## Overview

The `index.js` file exports the `OpenIDFederationAPIAdmin` class (aliased as `Admin`) which serves as the main interface for interacting with an OpenID Federation server's administrative API.

## Installation

```bash
npm install @findyfi/trustregistry-admin
```

## Quick Start

```js
import { Admin } from '@findyfi/trustregistry-admin'

// Initialize the client
const adminUrl = 'https://admin.example.trustregistry.eu'
const authUrl = 'https://auth.example.com/oauth/token'
const oidf = new Admin(adminUrl)

// Authenticate
const clientId = 'YOUR_CLIENT_ID'
const clientSecret = 'YOUR_CLIENT_SECRET'
await oidf.authenticate(authUrl, clientId, clientSecret)

// Create an account
const account = await oidf.createAccount('Test Service', 'https://test.example.com')
console.log('Created account:', account)
```

## API Reference

### Constructor

```js
new Admin(baseUrl, options = {})
```

- `baseUrl` (string): Base URL of the OpenID Federation API
- `options` (object): Optional configuration
  - `apiKey` (string): Pre-existing API key (optional)
  - `username` (string): Default username for requests (optional)

### Authentication

#### `authenticate(authUrl, clientId, clientSecret, scope='email')`

Authenticates using OAuth2 client credentials flow and automatically handles token refresh.

```js
await oidf.authenticate(
  'https://auth.example.com/oauth/token',
  'client_id',
  'client_secret',
  'email'
)
```

#### `setUsername(username)`

Sets the default username for subsequent API calls.

#### `quit()`

Clears authentication and cancels token refresh.

### Account Management

#### `accounts()`

Retrieves all accounts.

```js
const accounts = await oidf.accounts()
```

#### `createAccount(username, identifier)`

Creates a new account.

```js
const account = await oidf.createAccount('myservice', 'https://myservice.example.com')
```

#### `deleteAccount(username)`

Deletes an account.

```js
await oidf.deleteAccount('myservice')
```

### Key Management

#### `createKey(kmsKeyRef, signatureAlgorithm='ES256', username=null)`

Creates a new cryptographic key.

```js
const key = await oidf.createKey('my-kms-key-ref', 'ES256')
```

#### `getKeys(username=null)`

Retrieves all keys for an account.

```js
const keys = await oidf.getKeys()
```

#### `deleteKey(keyId, reason=null, username=null)`

Deletes a key with optional reason.

```js
await oidf.deleteKey('key-123', 'Key compromised')
```

### Metadata Management

#### `createMetadata(json, username=null)`

Adds metadata to an account.

```js
await oidf.createMetadata({
  "federation_entity": {
    "contacts": ["admin@example.com"],
    "homepage_uri": "https://example.com"
  }
})
```

#### `getMetadata(username=null)`

Retrieves metadata for an account.

#### `deleteMetadataEntry(entryId, username=null)`

Deletes a specific metadata entry.

### Authority Hints

#### `addAuthorityHint(authorityId, username=null)`

Adds an authority hint.

```js
await oidf.addAuthorityHint('https://authority.example.com')
```

#### `getAuthorityHints(username=null)`

Retrieves all authority hints.

#### `deleteAuthorityHint(authorityId, username=null)`

Removes an authority hint.

### Entity Configuration

#### `getEntityConfiguration(username=null)`

Retrieves the entity configuration statement.

#### `publishEntityConfiguration(kmsKeyRef, kid, dryRun=false, username=null)`

Publishes an entity configuration statement.

```js
await oidf.publishEntityConfiguration('kms-key-ref', 'key-id', false)
```

### Subordinate Management

#### `addSubordinate(subordinateId, username=null)`

Adds a subordinate entity.

```js
await oidf.addSubordinate('https://subordinate.example.com')
```

#### `getSubordinates(username=null)`

Retrieves all subordinates.

#### `deleteSubordinate(subordinateId, username=null)`

Removes a subordinate.

#### Subordinate Metadata

- `addSubordinateMetadata(subordinateId, json, username=null)`
- `getSubordinateMetadata(subordinateId, username=null)`
- `deleteSubordinateMetadataEntry(subordinateId, entryId, username=null)`

#### Subordinate Keys

- `addSubordinateJWKS(subordinateId, json, username=null)`
- `getSubordinateJWKS(subordinateId, username=null)`
- `deleteSubordinateJWKS(subordinateId, jwkId, username=null)`

#### Subordinate Statements

- `getSubordinateStatement(subordinateId, username=null)`
- `publishSubordinateStatement(subordinateId, kmsKeyRef, kid, dryRun=false, username=null)`

### Trust Mark Management

#### Trust Mark Types

```js
// Get all trust mark types
const types = await oidf.getTrustMarkTypes()

// Create a trust mark type
await oidf.createTrustMarkType('https://example.com/trust-mark-type')

// Delete a trust mark type
await oidf.deleteTrustMarkType('https://example.com/trust-mark-type')
```

#### Trust Mark Issuers

```js
// Add issuer to trust mark type
await oidf.addTrustMarkIssuer('trust-mark-type-id', 'issuer-id')

// Remove issuer
await oidf.removeTrustMarkIssuer('trust-mark-type-id', 'issuer-id')
```

#### Trust Marks

```js
// Issue a trust mark
const trustMark = {
  sub: 'https://entity.example.com',
  id: 'https://trust-mark-type.example.com',
  // ... other trust mark claims
}
await oidf.issueTrustMark(trustMark)

// Get all issued trust marks
const trustMarks = await oidf.getTrustMarks()

// Delete a trust mark
await oidf.deleteTrustMark('trust-mark-id')
```

#### Received Trust Marks

```js
// Add a received trust mark
await oidf.addTrustMark('trust-mark-id', 'jwt-token')

// Get all received trust marks
const received = await oidf.getReceivedTrustMarks()

// Delete a received trust mark
await oidf.deleteReceivedTrustMark('trust-mark-id')
```

### System Operations

#### `status()`

Retrieves API status.

```js
const status = await oidf.status()
```

#### `logs(limit=100, filter={})`

Retrieves system logs with optional filtering.

```js
// Get last 50 logs
const logs = await oidf.logs(50)

// Filter by severity
const errorLogs = await oidf.logs(100, { severity: 'Error' })

// Filter by tag
const taggedLogs = await oidf.logs(100, { tag: 'authentication' })
```

## Error Handling

The client includes built-in error handling that logs errors to the console and re-throws them. All API methods are async and should be wrapped in try-catch blocks:

```js
try {
  const accounts = await oidf.accounts()
  console.log(accounts)
} catch (error) {
  console.error('Failed to fetch accounts:', error.message)
}
```

## Environment Variables

Common environment variables for configuration:

```bash
API_URL=https://admin.trustregistry.example.com
AUTH_URL=https://auth.example.com/oauth/token
CLIENT_ID=your_client_id
CLIENT_SECRET=your_client_secret
```

## Complete Example

```js
import { Admin } from '@findyfi/trustregistry-admin'

const adminUrl = process.env.API_URL || 'https://admin.findy.trustregistry.eu'
const authUrl = process.env.AUTH_URL || 'https://id.keycloak.findy.fi/realms/trustregistry-dev/protocol/openid-connect/token'

const oidf = new Admin(adminUrl)
const clientId = process.env.CLIENT_ID || 'MY_CLIENT_ID'
const clientSecret = process.env.CLIENT_SECRET || 'MY_CLIENT_SECRET'

try {
  // Authenticate
  await oidf.authenticate(authUrl, clientId, clientSecret)
  
  // Create an account
  const accId = 'Test Service'
  const actorUrl = 'https://test.example/'
  const acc = await oidf.createAccount(accId, actorUrl)
  console.log('New account:', acc)

  // List all accounts
  const accounts = await oidf.accounts()
  console.log('Accounts:', accounts)

  // Get subordinates
  const subordinates = await oidf.getSubordinates()
  console.log('Root subordinates:', subordinates)

  // Check system status
  const status = await oidf.status()
  console.log('API Status:', status)

} catch (error) {
  console.error('Error:', error.message)
} finally {
  // Clean up
  oidf.quit()
}
```

## License

Apache-2.0
