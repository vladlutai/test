# ZeMind Network Layer

## Overview

The ZeMind Unity Network Layer is designed for managing network interactions, authorization, and data storage within Unity applications. This framework provides robust tools for handling authorization flows, sending network requests, and managing user and session data.

## Features

- **Flexible Authorization**: Support for multiple authorization types including credentials, sequence wallets, and UDIDs.
- **Network Communication**: Utilities for sending and managing network requests with support for progress reporting, error handling, and token-based authentication.
- **Token Management**: Mechanisms for handling access and refresh tokens, including automatic token refresh and storage.
- **Persistent Data Storage**: Management of account data with local saving and loading capabilities.

## Components

### Authorization

#### `AuthorizationBase`

- **Description**: Abstract base class for all authorization providers.
- **Key Methods**:
  - `Authorize(IAuthorizationInfo info)`: Handles the authorization process by determining the appropriate endpoint and sending a network request.
  - `UpdateData(JSONNode data)`: Updates authorization data with the response from the network request.

#### `CredentialsAuthorization`

- **Description**: Handles authorization using credentials (username and password).
- **Inheritance**: Inherits from `AuthorizationBase`.

#### `SequenceWalletAuthorization`

- **Description**: Handles authorization using a sequence wallet.
- **Inheritance**: Inherits from `AuthorizationBase`.

#### `UdidAuthorization`

- **Description**: Handles authorization using a UDID (Unique Device Identifier).
- **Inheritance**: Inherits from `AuthorizationBase`.

### `AuthorizationData`

- **Description**: Holds information about access and refresh tokens.
- **Key Properties**:
  - `AccessToken`: Contains the access token and its expiry information.
  - `RefreshToken`: Contains the refresh token and its expiry information.
- **Key Methods**:
  - `UpdateData(JSONNode json)`: Updates the token data from a JSON response.
  - `IsValid()`: Checks if both the access and refresh tokens are valid.

### `AuthorizationInfo`

- **Description**: Provides the necessary information for an authorization request.
- **Key Properties**:
  - `Type`: The type of authentication (e.g., Signin, Signup).
  - `Parameters`: The parameters for the authorization request.
  - `OnProgressChanged`: Callback for progress updates.
  - `OnDone`: Callback for completion of the request.

### Network Requests

#### `NetworkBase`

- **Description**: Base class for sending network requests.
- **Key Methods**:
  - `SendRequest(IRequestInfo requestInfo)`: Sends a network request based on the provided `RequestInfo`.
  - `MonitorProgress(UnityWebRequestAsyncOperation operation, Action<float> onProgressChanged)`: Monitors the progress of the request.
  - `CreateRequest(IRequestInfo requestInfo)`: Creates a `UnityWebRequest` based on the request info.
  - `RequestAuthorizationTokenIfNeeded()`: Requests a new authorization token if the current one is invalid.
  - `ParseResponse<T>(string json)`: Parses the JSON response into a specified type.
  - `HandleError(UnityWebRequest request)`: Handles errors encountered during the request.

#### `RequestInfo`

- **Description**: Contains parameters for making a network request.
- **Key Properties**:
  - `RequestMethod`: The HTTP method to use (GET, POST, etc.).
  - `OnProgressChanged`: Callback for progress updates.
  - `OnDone`: Callback for request completion.
  - `URL`: The URL for the request.
  - `Parameters`: The request parameters.
  - `UseAuth`: Indicates if authentication should be used.
  - `CanRefreshTokenIfNeeded`: Indicates if the token can be refreshed if necessary.

### Data Management

#### `AccountData`

- **Description**: Manages account-related data, including device ID and authorization tokens.
- **Key Properties**:
  - `BaseUrl`: The base URL for API requests based on the game stage.
  - `RefreshTokenUrl`: The URL used for refreshing tokens.
  - `AuthorizationData`: Contains the current authorization data.
  - `DeviceID`: The unique identifier for the device.
- **Key Methods**:
  - `Init()`: Initializes account data by loading saved data or creating new defaults.

#### `AccountSaveData`

- **Description**: Stores the device ID and authorization data for saving and loading.
- **Key Properties**:
  - `DeviceID`: The unique identifier for the device.
  - `AuthorizationData`: The stored authorization data.

### Token Management

#### `Token`

- **Description**: Represents an authorization token, including access and refresh tokens.
- **Key Properties**:
  - `Value`: The token string.
  - `BearerToken`: The token formatted as a Bearer token for HTTP requests.
  - `ExpiryDate`: The expiration date of the token.
  - `UpdateDate`: The date when the token was last updated.
- **Key Methods**:
  - `IsValid()`: Checks if the token is valid based on its expiry date.

## Token Refresh Process

1. **Request Authorization Token**: When sending a request, `NetworkBase` checks if the access token is valid using `RequestAuthorizationTokenIfNeeded()`.
2. **Refresh Token**: If the access token is expired but the refresh token is valid, a new access token is requested using the refresh token.
3. **Update Token**: The new tokens are parsed from the response and updated in `AuthorizationData`.
4. **Save Token**: Updated tokens are saved to persistent storage using `AccountData`.

## Usage

### Authorization Example

To perform authorization, create an instance of `AuthorizationInfo` and pass it to `AuthorizationService`.

```csharp
var authInfo = new AuthorizationInfo
{
    Type = AuthenticationType.Credentials,
    Parameters = new CredentialsData("email@example.com", "password"),
    OnProgressChanged = progress => Debug.Log($"Progress: {progress}"),
    OnDone = success => Debug.Log($"Authorization {(success ? "succeeded" : "failed")}")
};

var authService = new AuthorizationService();
authService.Authorize(AuthenticationType.Credentials, authInfo);
```

## Sending Network Requests

To send network requests using the ZeMind Unity Framework, follow these steps:

1. **Create a `RequestInfo` Object**: Configure the request parameters.

2. **Call `SendRequest`**: Use the `SendRequest` method to execute the request.

### Example

```csharp
var requestInfo = new RequestInfo
{
    RequestMethod = HttpMethod.Post,
    URL = "https://example.com/api/resource",
    Parameters = new { key = "value" },
    UseAuth = true,
    OnProgressChanged = progress => Debug.Log($"Progress: {progress}"),
    OnDone = success => Debug.Log($"Request {(success ? "succeeded" : "failed")}")
};

var networkBase = new YourNetworkBaseImplementation();
await networkBase.SendRequest(requestInfo);
```

## Developer Notes

### Entry Point

- **Authorization Flow**: 
  To start the authorization flow, interact with the `AuthorizationService`. Create an instance of `AuthorizationInfo` and pass it to the `Authorize` method.

### Checking Login Status

- **Token Validity**: 
  To check if the session has expired, use the `AuthorizationData` class. The `IsValid` method will help you determine if the access and refresh tokens are still valid.

### Handling Tokens

- **Token Usage**: 
  The framework manages token storage and application to requests automatically. Tokens are saved persistently and used for authenticated requests. For endpoints where you don’t want to use the token, set `UseAuth` to `false` in `RequestInfo`.

### Token Expiration

- **Automatic Refresh**: 
  If a request encounters an expired token error, `NetworkBase` will attempt to refresh the token and retry the request. Manual handling of token refreshing is not required.

