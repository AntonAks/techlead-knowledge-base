# Authentication & OAuth

## Overview

**Authentication** is the process of verifying the identity of a user, device, or system. **OAuth** (Open Authorization) is an open standard authorization protocol that enables applications to obtain limited access to user accounts on an HTTP service without exposing user credentials.

While authentication answers "Who are you?", authorization answers "What are you allowed to do?". OAuth primarily handles authorization, though it's often used in conjunction with authentication mechanisms.

## Basic Authentication Concepts

### **Authentication vs Authorization**

**Authentication (AuthN)**
- Verifies identity ("Who are you?")
- Confirms user credentials
- Examples: Username/password, biometrics, certificates

**Authorization (AuthZ)**  
- Determines permissions ("What can you do?")
- Controls access to resources
- Examples: Role-based access, permissions, scopes

### **Common Authentication Methods**

#### **Basic Authentication**
Simplest form where credentials are sent with each request.

```http
GET /api/protected HTTP/1.1
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

#### **Bearer Token Authentication**
Uses tokens instead of credentials for each request.

```http
GET /api/protected HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### **API Key Authentication**
Uses pre-shared keys for service-to-service communication.

```http
GET /api/data HTTP/1.1
X-API-Key: your-api-key-here
```

## OAuth 2.0 Fundamentals

### **What is OAuth 2.0?**

OAuth 2.0 is an authorization framework that enables applications to obtain limited access to user accounts. Instead of using the user's credentials directly, OAuth introduces an authorization layer that separates the client from the resource owner.

### **OAuth 2.0 Roles**

#### **Resource Owner (User)**
The entity that can grant access to a protected resource. Usually the end-user.

#### **Client (Application)**  
The application requesting access to the protected resource on behalf of the resource owner.

#### **Resource Server (API)**
The server hosting the protected resources, capable of accepting and responding to protected resource requests using access tokens.

#### **Authorization Server**
The server that authenticates the resource owner and issues access tokens to the client after getting proper authorization.

### **OAuth 2.0 Flow Overview**

```
┌──────────────┐                                ┌──────────────┐
│    Client    │                                │ Authorization│
│ Application  │                                │    Server    │
└──────────────┘                                └──────────────┘
       │                                               │
       │ 1. Authorization Request                      │
       │──────────────────────────────────────────────→│
       │                                               │
       │ 2. Authorization Grant                        │
       │←──────────────────────────────────────────────│
       │                                               │
       │ 3. Authorization Grant                        │
       │──────────────────────────────────────────────→│
       │                                               │
       │ 4. Access Token                               │
       │←──────────────────────────────────────────────│
       │                                               │
       │                    ┌──────────────┐           │
       │ 5. Access Token    │   Resource   │           │
       │───────────────────→│    Server    │           │
       │                    │     (API)    │           │
       │ 6. Protected       │              │           │
       │    Resource        │              │           │
       │←───────────────────│              │           │
       │                    └──────────────┘           │
```

## OAuth 2.0 Grant Types

### **Authorization Code Grant**
Most secure flow for web applications with server-side backend.

#### **Step-by-Step Process:**

1. **User Authorization Request**
```
https://auth.example.com/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=https://app.example.com/callback&
  scope=read_profile&
  state=xyz123
```

2. **User Grants Permission**
User logs in and approves the requested permissions.

3. **Authorization Code Response**
```
https://app.example.com/callback?
  code=AUTHORIZATION_CODE&
  state=xyz123
```

4. **Exchange Code for Token**
```http
POST /oauth/token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTHORIZATION_CODE&
redirect_uri=https://app.example.com/callback&
client_id=CLIENT_ID&
client_secret=CLIENT_SECRET
```

5. **Access Token Response**
```json
{
  "access_token": "2YotnFZFEjr1zCsicMWpAA",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "tGzv3JOkF0XG5Qx2TlKWIA",
  "scope": "read_profile"
}
```

#### **Simple Implementation Example:**

```javascript
// Step 1: Redirect user to authorization server
app.get('/login', (req, res) => {
  const authUrl = 'https://auth.example.com/authorize?' +
    'response_type=code&' +
    'client_id=your_client_id&' +
    'redirect_uri=https://yourapp.com/callback&' +
    'scope=read_profile&' +
    'state=' + generateRandomState();
  
  res.redirect(authUrl);
});

// Step 2: Handle callback with authorization code
app.get('/callback', async (req, res) => {
  const { code, state } = req.query;
  
  // Verify state parameter
  if (!verifyState(state)) {
    return res.status(400).send('Invalid state');
  }
  
  // Exchange code for tokens
  const tokenResponse = await fetch('https://auth.example.com/oauth/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code: code,
      redirect_uri: 'https://yourapp.com/callback',
      client_id: 'your_client_id',
      client_secret: 'your_client_secret'
    })
  });
  
  const tokens = await tokenResponse.json();
  
  // Store tokens securely
  req.session.access_token = tokens.access_token;
  req.session.refresh_token = tokens.refresh_token;
  
  res.redirect('/dashboard');
});

// Step 3: Use access token to make API calls
app.get('/profile', async (req, res) => {
  const accessToken = req.session.access_token;
  
  const profileResponse = await fetch('https://api.example.com/user/profile', {
    headers: {
      'Authorization': `Bearer ${accessToken}`
    }
  });
  
  const profile = await profileResponse.json();
  res.json(profile);
});
```

### **Client Credentials Grant**
For server-to-server authentication without user involvement.

#### **Flow:**
```http
POST /oauth/token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&
client_id=CLIENT_ID&
client_secret=CLIENT_SECRET&
scope=api_access
```

#### **Response:**
```json
{
  "access_token": "2YotnFZFEjr1zCsicMWpAA",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "api_access"
}
```

#### **Simple Example:**
```javascript
async function getClientCredentialsToken() {
  const response = await fetch('https://auth.example.com/oauth/token', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      'Authorization': 'Basic ' + btoa('client_id:client_secret')
    },
    body: new URLSearchParams({
      grant_type: 'client_credentials',
      scope: 'api_access'
    })
  });
  
  return await response.json();
}

// Use the token
const tokenData = await getClientCredentialsToken();
const apiResponse = await fetch('https://api.example.com/data', {
  headers: {
    'Authorization': `Bearer ${tokenData.access_token}`
  }
});
```

### **Implicit Grant (Deprecated)**
Previously used for single-page applications, now replaced by Authorization Code with PKCE.

### **Resource Owner Password Credentials (Deprecated)**
Direct use of username/password, only for highly trusted applications.

## PKCE (Proof Key for Code Exchange)

PKCE enhances the Authorization Code flow security, especially for public clients like mobile apps and SPAs.

### **PKCE Flow:**

1. **Generate Code Verifier and Challenge**
```javascript
// Generate code verifier (random string)
const codeVerifier = generateRandomString(128);

// Create code challenge (SHA256 hash)
const codeChallenge = base64URLEncode(sha256(codeVerifier));
```

2. **Authorization Request with PKCE**
```
https://auth.example.com/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=https://app.example.com/callback&
  scope=read_profile&
  code_challenge=CODE_CHALLENGE&
  code_challenge_method=S256&
  state=xyz123
```

3. **Token Exchange with Code Verifier**
```http
POST /oauth/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTHORIZATION_CODE&
redirect_uri=https://app.example.com/callback&
client_id=CLIENT_ID&
code_verifier=CODE_VERIFIER
```

### **PKCE Implementation Example:**
```javascript
// Generate PKCE parameters
function generatePKCE() {
  const codeVerifier = base64URLEncode(crypto.getRandomValues(new Uint8Array(32)));
  const encoder = new TextEncoder();
  const data = encoder.encode(codeVerifier);
  
  return crypto.subtle.digest('SHA-256', data).then(hash => {
    const codeChallenge = base64URLEncode(new Uint8Array(hash));
    return { codeVerifier, codeChallenge };
  });
}

// Start OAuth flow with PKCE
async function startOAuthFlow() {
  const { codeVerifier, codeChallenge } = await generatePKCE();
  
  // Store code verifier for later use
  sessionStorage.setItem('code_verifier', codeVerifier);
  
  const authUrl = 'https://auth.example.com/authorize?' +
    'response_type=code&' +
    'client_id=your_client_id&' +
    'redirect_uri=https://yourapp.com/callback&' +
    'scope=read_profile&' +
    'code_challenge=' + codeChallenge + '&' +
    'code_challenge_method=S256&' +
    'state=' + generateRandomState();
  
  window.location.href = authUrl;
}

// Handle callback
async function handleCallback() {
  const urlParams = new URLSearchParams(window.location.search);
  const code = urlParams.get('code');
  const codeVerifier = sessionStorage.getItem('code_verifier');
  
  const tokenResponse = await fetch('https://auth.example.com/oauth/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code: code,
      redirect_uri: 'https://yourapp.com/callback',
      client_id: 'your_client_id',
      code_verifier: codeVerifier
    })
  });
  
  const tokens = await tokenResponse.json();
  // Store tokens securely
}
```

## Token Management

### **Access Tokens**
Short-lived tokens used to access protected resources.

### **Refresh Tokens**
Long-lived tokens used to obtain new access tokens without re-authentication.

#### **Token Refresh Example:**
```javascript
async function refreshAccessToken(refreshToken) {
  const response = await fetch('https://auth.example.com/oauth/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'refresh_token',
      refresh_token: refreshToken,
      client_id: 'your_client_id',
      client_secret: 'your_client_secret'
    })
  });
  
  return await response.json();
}

// Automatic token refresh middleware
async function ensureValidToken(req, res, next) {
  const accessToken = req.session.access_token;
  const refreshToken = req.session.refresh_token;
  
  try {
    // Try to use current access token
    const testResponse = await fetch('https://api.example.com/test', {
      headers: { 'Authorization': `Bearer ${accessToken}` }
    });
    
    if (testResponse.status === 401) {
      // Access token expired, refresh it
      const newTokens = await refreshAccessToken(refreshToken);
      req.session.access_token = newTokens.access_token;
      req.session.refresh_token = newTokens.refresh_token || refreshToken;
    }
    
    next();
  } catch (error) {
    res.status(401).json({ error: 'Authentication required' });
  }
}
```

## OAuth 2.0 Scopes

Scopes define the specific permissions an application is requesting.

### **Common Scope Examples:**
- `read`: Read access to user data
- `write`: Write access to user data
- `profile`: Access to user profile information
- `email`: Access to user email address
- `openid`: OpenID Connect authentication

### **Using Scopes:**
```javascript
// Request specific scopes
const authUrl = 'https://auth.example.com/authorize?' +
  'response_type=code&' +
  'client_id=your_client_id&' +
  'scope=read_profile email&' +
  'redirect_uri=https://yourapp.com/callback';

// Check token scopes
function hasRequiredScope(token, requiredScope) {
  const scopes = token.scope ? token.scope.split(' ') : [];
  return scopes.includes(requiredScope);
}

// Middleware to check scopes
function requireScope(requiredScope) {
  return (req, res, next) => {
    const token = req.session.token_info;
    
    if (!hasRequiredScope(token, requiredScope)) {
      return res.status(403).json({
        error: 'insufficient_scope',
        message: `Required scope: ${requiredScope}`
      });
    }
    
    next();
  };
}
```

## OpenID Connect (OIDC)

OpenID Connect is an identity layer built on top of OAuth 2.0 that adds authentication capabilities.

### **OIDC vs OAuth 2.0:**
- OAuth 2.0: Authorization framework
- OIDC: Authentication protocol built on OAuth 2.0

### **OIDC Tokens:**

#### **ID Token**
Contains user authentication information and claims about the user.

```json
{
  "iss": "https://auth.example.com",
  "sub": "user123",
  "aud": "your_client_id",
  "exp": 1642680000,
  "iat": 1642676400,
  "name": "John Doe",
  "email": "john@example.com",
  "email_verified": true
}
```

#### **OIDC Flow Example:**
```javascript
// OIDC Authorization request
const oidcAuthUrl = 'https://auth.example.com/authorize?' +
  'response_type=code&' +
  'client_id=your_client_id&' +
  'scope=openid profile email&' +
  'redirect_uri=https://yourapp.com/callback&' +
  'state=xyz123';

// Token response includes ID token
const tokenResponse = {
  "access_token": "2YotnFZFEjr1zCsicMWpAA",
  "token_type": "Bearer",
  "expires_in": 3600,
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "scope": "openid profile email"
};

// Verify and decode ID token
function verifyIdToken(idToken) {
  try {
    const decoded = jwt.verify(idToken, publicKey, {
      issuer: 'https://auth.example.com',
      audience: 'your_client_id'
    });
    return decoded;
  } catch (error) {
    throw new Error('Invalid ID token');
  }
}
```

## Common OAuth Providers

### **Google OAuth**
```javascript
// Google OAuth configuration
const googleConfig = {
  authUrl: 'https://accounts.google.com/oauth2/v2/auth',
  tokenUrl: 'https://oauth2.googleapis.com/token',
  clientId: 'your_google_client_id',
  clientSecret: 'your_google_client_secret',
  scope: 'openid profile email'
};

// Google OAuth flow
function initiateGoogleLogin() {
  const authUrl = `${googleConfig.authUrl}?` +
    `response_type=code&` +
    `client_id=${googleConfig.clientId}&` +
    `scope=${encodeURIComponent(googleConfig.scope)}&` +
    `redirect_uri=${encodeURIComponent('https://yourapp.com/auth/google/callback')}`;
  
  window.location.href = authUrl;
}
```

### **GitHub OAuth**
```javascript
// GitHub OAuth configuration
const githubConfig = {
  authUrl: 'https://github.com/login/oauth/authorize',
  tokenUrl: 'https://github.com/login/oauth/access_token',
  apiUrl: 'https://api.github.com',
  clientId: 'your_github_client_id',
  clientSecret: 'your_github_client_secret'
};

// GitHub user info
async function getGitHubUser(accessToken) {
  const response = await fetch('https://api.github.com/user', {
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Accept': 'application/vnd.github.v3+json'
    }
  });
  
  return await response.json();
}
```

## Security Best Practices

### **State Parameter**
Always use the state parameter to prevent CSRF attacks.

```javascript
// Generate and store state
function generateState() {
  const state = crypto.randomUUID();
  sessionStorage.setItem('oauth_state', state);
  return state;
}

// Verify state in callback
function verifyState(receivedState) {
  const storedState = sessionStorage.getItem('oauth_state');
  sessionStorage.removeItem('oauth_state');
  return storedState === receivedState;
}
```

### **Secure Token Storage**
```javascript
// For web applications - use httpOnly cookies
res.cookie('access_token', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 3600000 // 1 hour
});

// For SPAs - use memory storage with automatic refresh
class TokenManager {
  constructor() {
    this.accessToken = null;
    this.refreshToken = null;
    this.expiresAt = null;
  }
  
  setTokens(accessToken, refreshToken, expiresIn) {
    this.accessToken = accessToken;
    this.refreshToken = refreshToken;
    this.expiresAt = Date.now() + (expiresIn * 1000);
  }
  
  isTokenValid() {
    return this.accessToken && Date.now() < this.expiresAt;
  }
  
  async getValidToken() {
    if (this.isTokenValid()) {
      return this.accessToken;
    }
    
    if (this.refreshToken) {
      await this.refreshAccessToken();
      return this.accessToken;
    }
    
    throw new Error('No valid token available');
  }
}
```

### **HTTPS Only**
Always use HTTPS for OAuth flows in production.

### **Validate Redirect URIs**
Strictly validate redirect URIs to prevent authorization code interception.

## Error Handling

### **Common OAuth Errors:**

```javascript
// Handle OAuth errors
function handleOAuthError(error, errorDescription) {
  switch (error) {
    case 'invalid_request':
      console.error('Invalid OAuth request:', errorDescription);
      break;
    case 'unauthorized_client':
      console.error('Client not authorized:', errorDescription);
      break;
    case 'access_denied':
      console.log('User denied access');
      // Redirect to login page
      break;
    case 'unsupported_response_type':
      console.error('Unsupported response type:', errorDescription);
      break;
    case 'invalid_scope':
      console.error('Invalid scope requested:', errorDescription);
      break;
    case 'server_error':
      console.error('Authorization server error:', errorDescription);
      break;
    case 'temporarily_unavailable':
      console.error('Service temporarily unavailable:', errorDescription);
      // Implement retry logic
      break;
    default:
      console.error('Unknown OAuth error:', error, errorDescription);
  }
}

// Error handling in callback
app.get('/callback', (req, res) => {
  const { code, error, error_description, state } = req.query;
  
  if (error) {
    handleOAuthError(error, error_description);
    return res.redirect('/login?error=' + error);
  }
  
  if (!verifyState(state)) {
    return res.status(400).send('Invalid state parameter');
  }
  
  // Continue with normal flow...
});
```

## Testing OAuth Implementations

### **Unit Testing:**
```javascript
// Mock OAuth provider responses
const mockAuthProvider = {
  token: {
    access_token: 'mock_access_token',
    token_type: 'Bearer',
    expires_in: 3600,
    refresh_token: 'mock_refresh_token'
  },
  
  userInfo: {
    id: '123',
    name: 'Test User',
    email: 'test@example.com'
  }
};

// Test token exchange
test('should exchange authorization code for tokens', async () => {
  // Mock the token endpoint
  nock('https://auth.example.com')
    .post('/oauth/token')
    .reply(200, mockAuthProvider.token);
  
  const tokens = await exchangeCodeForTokens('test_code');
  
  expect(tokens.access_token).toBe('mock_access_token');
  expect(tokens.token_type).toBe('Bearer');
});
```

### **Integration Testing:**
```javascript
// Test complete OAuth flow
test('complete OAuth flow', async () => {
  // Start authorization
  const authUrl = await startOAuthFlow();
  expect(authUrl).toContain('response_type=code');
  
  // Simulate callback
  const tokens = await handleOAuthCallback('test_code', 'test_state');
  expect(tokens).toHaveProperty('access_token');
  
  // Test API call with token
  const userInfo = await getUserInfo(tokens.access_token);
  expect(userInfo).toHaveProperty('email');
});
```

## Summary

OAuth 2.0 provides a robust framework for secure authorization in modern applications. Key takeaways:

**Benefits:**
- Secure delegation of access without sharing passwords
- Standardized protocol with wide industry adoption  
- Flexible grant types for different application types
- Scope-based access control

**Best Practices:**
- Always use HTTPS in production
- Implement PKCE for public clients
- Use state parameter to prevent CSRF
- Store tokens securely
- Implement proper error handling
- Validate all redirect URIs

**Common Use Cases:**
- Social login (Google, Facebook, GitHub)
- API access delegation
- Single Sign-On (SSO) systems
- Microservices authentication
- Mobile app authentication

OAuth 2.0, combined with OpenID Connect, provides a complete solution for modern authentication and authorization needs while maintaining security and user experience.