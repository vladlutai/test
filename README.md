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

## NetworkCore: Centralized Service Management

In the ZeMind Unity Network Layer, the `NetworkCore` class is designed to manage and provide access to network services such as the `AuthorizationService`. Here’s how it works and how you can use it:

### Centralized Service Management

The `NetworkCore` class holds references to the various network services. This approach centralizes the initialization and management of these services, making it easier to handle them consistently throughout your application.

### Accessing Services

You can access network services via the `NetworkCore` class. For instance, to perform authorization, you can call:

```csharp
NetworkCore.AuthorizationService.Authorize(AuthenticationType.Credentials, new AuthorizationInfo()
{
    Type = AuthenticationType.Signin,
    Parameters = _credentialsData,
    OnDone = OnAuthorizationDone,
    OnProgressChanged = OnProgressChanged
});
```


### Service Initialization

All network services, including `AuthorizationService`, will be initialized within the `NetworkCore` class. This setup ensures that you have a single point of reference for managing these services, which simplifies your code and improves maintainability.

### Static Methods and Common Manager

Since the services do not need to retain references beyond their immediate use and only rely on callbacks, the methods can be static. The `NetworkCore` class acts as a common manager, holding references to the service instances and providing access to them in a centralized manner.

This design ensures that service instances are properly managed and accessed without requiring individual classes to handle their own service references.


# ZeMind Unity Framework

## Overview

The ZeMind Unity Framework provides a structured approach to managing game services and core functionalities within Unity applications. This README explains the roles of the `Main` and `Core` classes, and how to use them for initialization and event handling.

## Main Class

The `Main` class is derived from `MonoBehaviour` and serves as a central manager for handling key Unity lifecycle events and ensuring the proper initialization of the framework.

### Responsibilities

- **Singleton Instance**: Ensures that only one instance of the `Main` class exists and is accessible globally through the static `Instance` property.
- **Event Handling**: Subscribes to Unity’s application lifecycle events and sends corresponding events using `EventManager`.

### Key Methods

- **Awake()**: Checks if the instance is valid. If not, the object is destroyed. Initializes the instance and ensures it persists across scene loads with `DontDestroyOnLoad`.
- **IsInstanceValid()**: Validates whether the current instance is the only one.
- **Init()**: Sets the static `Instance` property and marks the object to persist across scenes.
- **OnApplicationFocus(bool hasFocus)**: Sends an event when the application gains or loses focus.
- **OnApplicationPause(bool pauseStatus)**: Sends an event when the application is paused or resumed.
- **OnApplicationQuit()**: Sends an event when the application is about to quit.

### Example Usage

The `Main` class should be added to a GameObject in your scene to handle initialization and lifecycle events. It ensures that core services and events are properly managed throughout the application's lifecycle.

## Core Class

The `Core` class is intended to handle the initialization of game services and core functionalities. It serves as a centralized location for setting up and managing essential components of the game.

### Responsibilities

- **Service Initialization**: All services and game core functionalities should be initialized in the `Core` class. This ensures that services are properly set up before they are used and provides a single point of reference for managing these services.

### Usage

- **Initialization**: Add initialization code for your services and core components within the `Core` class. Ensure that this class is instantiated early in the game’s lifecycle to set up the necessary components.

```csharp
namespace ZeMind.UnityFramework.Core.Main
{
    public class Core
    {
        // Initialize services and core functionalities here
    }
}
```


# ZeMind Unity Framework - Asset System

## Overview

The ZeMind Unity Framework includes a robust asset management system for handling addressable assets in Unity. This section explains how to use the `AssetManager` class, asset categories, and keys to manage your assets efficiently.

## AssetManager Class

The `AssetManager` class is responsible for loading, unloading, and managing assets within your Unity application. It utilizes Unity's Addressable Assets system to streamline asset management.

### Key Features

- **Asset Caching**: Caches loaded assets to prevent redundant loading.
- **Progress Reporting**: Provides progress updates during asset loading.
- **Asset Unloading**: Supports unloading of individual assets and entire categories.
- **Check Loaded Assets**: Verifies if an asset is currently loaded.

### Key Methods

- **LoadAsset<T>**:
  - **Parameters**: 
    - `string key`: The key used to load the asset.
    - `Action<T> onLoaded`: Callback invoked when the asset is loaded.
    - `Action<float> onProgressChanged`: Callback for progress updates.
    - `CategoryEnum categoryEnum`: The category of the asset (default is `Default`).
  - **Description**: Loads an asset asynchronously and invokes the appropriate callbacks.

- **LoadAssetList<T>**:
  - **Parameters**:
    - `List<string> keyList`: A list of keys to load assets from.
    - `Action<T> onLoaded`: Callback for when assets are loaded.
    - `Action<float> onProgressChanged`: Callback for progress updates.
    - `CategoryEnum categoryEnum`: The category of the assets (default is `Default`).
  - **Description**: (To be implemented) Loads multiple assets asynchronously.

- **UnloadAsset**:
  - **Parameters**:
    - `string key`: The key of the asset to unload.
    - `CategoryEnum categoryEnum`: The category of the asset (default is `Default`).
  - **Description**: Unloads a specific asset from memory.

- **UnloadAssetCategory**:
  - **Parameters**:
    - `CategoryEnum categoryEnum`: The category of assets to unload.
  - **Description**: Unloads all assets within the specified category.

- **IsAssetLoaded**:
  - **Parameters**:
    - `string key`: The key of the asset to check.
    - `CategoryEnum categoryEnum`: The category of the asset (default is `Default`).
  - **Description**: Checks if a specific asset is currently loaded.

- **Dispose**:
  - **Description**: Releases all cached assets and cleans up resources.

### Example Usage

```csharp
public class ExampleUsage : MonoBehaviour
{
    private AssetManager _assetManager;

    private void Start()
    {
        _assetManager = new AssetManager();

        _assetManager.LoadAsset<MyAsset>("MyAssetKey", OnAssetLoaded, OnProgressChanged);
    }

    private void OnAssetLoaded(MyAsset asset)
    {
        // Handle the loaded asset
    }

    private void OnProgressChanged(float progress)
    {
        // Handle progress updates
    }
}
```
## CategoryEnum

`CategoryEnum` defines various categories for assets, allowing you to group and manage them effectively.

### Enum Values

- **Default**: The default category.
- **Store**: Assets related to in-game store functionalities.
- **Inventory**: Assets used within the inventory system.
- **Loading**: Assets used during the loading process.

## KeyEnum

`KeyEnum` defines predefined keys for common assets. You can also add custom keys as needed.

### Enum Values

- **PlayerPrefab**: Key for player prefab.
- **EnemyPrefab**: Key for enemy prefab.
- **EnvironmentAsset**: Key for environment assets.

## KeyManager Class

The `KeyManager` class provides a mapping from `KeyEnum` values to string keys used by the Addressable Asset system.

### Key Methods

- **Get(KeyEnum keyEnum)**:
  - **Parameters**:
    - `KeyEnum keyEnum`: The key enum value to retrieve the string key.
  - **Description**: Returns the string key associated with the provided `KeyEnum`.

### Example Usage

```csharp
string playerPrefabKey = KeyManager.Get(KeyEnum.PlayerPrefab);
```



## SaveSystem<TSaveType> Class

The `SaveSystem<TSaveType>` class provides a generic system for managing save data in Unity. It supports saving and loading data to and from files, with integration for Unity's event system to trigger data saving on application quit.

### Key Features

- **Generic Type**: `TSaveType` allows for flexible data types to be saved and loaded.
- **Event Handling**: Integrates with Unity's event system to trigger data saving.
- **File Management**: Handles file paths and extensions for saving and loading data.

### Constructor

#### `SaveSystem(string savesName)`

- **Parameters**:
  - `savesName`: The name of the save file.
- **Description**: Initializes the save system with the specified save file name and sets up event listening for application quit events.

### Properties

- **`SomeSaves`**: Holds the loaded save data of type `TSaveType`.

### Methods

#### `void SaveData()`

- **Description**: Saves the current data to the file system and logs a confirmation message.

#### `void LoadAndInitData()`

- **Description**: Loads the save data from the file system and initializes the `SomeSaves` property.

### Private Methods

#### `void Save(string savesPath)`

- **Description**: Serializes the data to a file using the provided path.
- **Parameters**:
  - `savesPath`: The path where the data should be saved.

#### `TSaveType Load(string savesPath)`

- **Description**: Deserializes data from a file using the provided path.
- **Parameters**:
  - `savesPath`: The path where the data should be loaded from.

#### `(string directory, string filePath) GetSaveFilePath(string fileName)`

- **Description**: Constructs the file path for saving and loading data, taking into account the environment (Editor or build).
- **Parameters**:
  - `fileName`: The base name of the save file.
- **Returns**: A tuple containing the directory and file path.

### Constants

- **`SaveFileExtension`**: The file extension used for save files (".dat").
- **`FolderName`**: The folder name where save files are stored ("LocalSaves").

### Example Usage

```csharp
// Initialize save system with a specific save file name
var saveSystem = new SaveSystem<MySaveData>("PlayerData");

// Save data
saveSystem.SaveData();

// Load and initialize data
saveSystem.LoadAndInitData();
```

## Event Handling

- **Application Quit**: The `SaveData` method is triggered when the application quits. This is managed through event subscriptions established in the constructor and unsubscriptions in the destructor of the `SaveSystem<TSaveType>` class.



## SerializeUtility

The `SerializeUtility` class provides methods for serializing and deserializing data using encryption.

### Key Features

- **Encryption**: Data is encrypted using Rijndael (AES) encryption with a predefined key and initialization vector (IV).
- **Serialization**: Data is serialized into a binary format and encrypted before being written to a file.
- **Deserialization**: Data is read from a file, decrypted, and then deserialized from binary format.

### Methods

#### `Serialize<T>(T saves, (string directory, string filePath) savesPath)`

- **Parameters**:
  - `T saves`: The data to be serialized and encrypted.
  - `(string directory, string filePath) savesPath`: The directory and file path where the encrypted data will be saved.
- **Description**: Serializes and encrypts the provided data, then writes it to the specified file path. If the directory does not exist, it will be created.

#### `Deserialize<T>(string savesPath)`

- **Parameters**:
  - `string savesPath`: The path to the file containing the encrypted data.
- **Returns**: Returns the deserialized and decrypted data of type `T`. If the file does not exist or an error occurs, it returns the default value for type `T`.
- **Description**: Reads the encrypted data from the specified file path, decrypts it, and deserializes it into the specified type.

### Encryption Details

- **Key**: A static byte array used for encryption and decryption.
- **Initialization Vector (IV)**: A static byte array used along with the key to initialize the encryption algorithm.

```csharp
// Example usage
SerializeUtility.Serialize(myData, ("path/to/directory", "path/to/file.dat"));
var loadedData = SerializeUtility.Deserialize<MyDataType>("path/to/file.dat");
```


## Logger

The `Logger` class is responsible for capturing and writing log messages to a file. It is designed to work within Unity and can be used to record logs, including exceptions, to a specific directory.

### Key Features

- **Log Directory Setup**: Creates a `Logs` directory in the application's persistent data path (or data path in the editor) if it does not already exist.
- **Log Message Handling**: Captures log messages and exceptions, and writes them to a log file.
- **Editor Integration**: Provides functionality to reveal the log directory in the Unity Editor.

### Methods

#### `Awake()`

- **Description**: Initializes the log directory path, creates the directory if it doesn't exist, and sets up a listener for log messages.

#### `OnLogMessageReceived(string condition, string stacktrace, LogType type)`

- **Parameters**:
  - `string condition`: The log message or condition.
  - `string stacktrace`: The stack trace for exceptions.
  - `LogType type`: The type of log message (e.g., exception, error, warning, etc.).
- **Description**: Handles incoming log messages. Writes exception messages and stack traces to the log file.

#### `Update()`

- **Description**: (Editor Only) Allows you to reveal the log directory in the Finder (macOS) or Explorer (Windows) when the `L` key is pressed.

#### `OnDestroy()`

- **Description**: Disposes of the `FileWriter` when the `Logger` object is destroyed, ensuring that resources are properly released.

### Usage

To use the `Logger` class, simply attach it to a GameObject in your scene. It will automatically handle log message capturing and writing.

```csharp
// Example usage
// Attach this script to a GameObject in your scene
```

