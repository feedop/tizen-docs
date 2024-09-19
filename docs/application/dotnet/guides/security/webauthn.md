# Web Authentication

The Web Authentication module provides a C# API enabling the creation and use of
strong, attested, scoped, public key-based credentials by web applications, for the
purpose of strongly authenticating users.

It provides an Authenticator class containing methods for creating public key-based credentials
(`Authenticator.MakeCredential()`) and using them (`Authenticator.GetAssertion()`). Both these operations
are performed asynchronously. Callbacks passed as arguments are used to notify about the progress
or when user's interaction is necessary. Due to significant amount of time required to complete both
requests, cancelation is also possible `Authenticator.Cancel()`. The module also provides a variety of
data types based on W3C Web Authentication API ([https://www.w3.org/TR/webauthn-3/]) used to control
the credential creation and assertion process.

Please note that no other call (except for a cancel call) can be made as long as the authenticator
is busy with your request.

## Prerequisites

To use the methods and properties of the `Tizen.Security.WebAuthn` namespace, include it in your application:

```csharp
using Tizen.Security.WebAuthn;
```
The Web Authentication API is exposed via the `Authenticator` static class.

## Make credential
One of the two key methods exposed by WebAuthn is `Authenticator.MakeCredential()`, allowing the Relying Party
to request the creation of a new user credential.

Example usage:
```csharp
var clientData = new ClientData(
    new byte[] /* Client data JSON as a byte array. */,
    HashAlgorithm.Sha256);
var pubkeyCredCreationOptions = new PubkeyCredCreationOptions(
    new RpEntity(/* Relying Party info */);
    new UserEntity(/* User Entity info */),
    new List<PubkeyCredParam>() /* Key types and signature algorithms the Relying Party supports */,
    0 /* Timeout */,
    new List<PubkeyCredDescriptor>() /* Existing credentials for this user */,
    new AuthenticationSelCri(/* Capabilities and settings that the authenticator MUST or SHOULD satisfy */),
    new List<PubkeyCredHint>() ,
    AttestationPref.Indirect,
    new List<byte[]>() /* Attestation format info */,
    new List<AuthenticationExt>() /* Extensions */,
    new HybridLinkedData(
        new byte[] /* CBOR:"1" */,
        new byte[] /* CBOR:"2" */,
        new byte[] /* CBOR:"3" */,
        new byte[] /* CBOR:"4" */,
        new byte[] /* CBOR:"5" */,
        new byte[] /* CBOR:"6" */,
        new byte[] /* Domain String of tunnel server */,
        new byte[] /* Identity Key created during QR initiated transaction */
    ));

var callbacks = new McCallbacks(
    (string str, object obj) =>
        {
            /* A callback responsible for displaying the qr code. */
        },
    (PubkeyCredentialAttestation att, WauthnError err, object obj) =>
        {
            /* Callback function for getting the final response.
            invoked when the response for the MakeCredential request needs to be returned */
        },
    (HybridLinkedData att, WauthnError err, object obj) =>
    {
        /* Callback function for getting the updated linked device data. */
    },
    "some data" /* Any user defined object that will be used by the callbacks */);

try
{
    Authenticator.MakeCredential(clientData, pubkeyCredCreationOptions, callbacks);
}
catch (Exception e)
{
    // Error handling
}
```

Refer to the following W3C specification for more information:
[https://www.w3.org/TR/webauthn-3/#sctn-createCredential].

## Get assertion
`Authenticator.GetAssertion()` can be used by the Relying Party to discover an use an existing credential.
You can optionally specify what credential types you deem acceptable. The user is then presented with
credentials that match these criteria and may choose one or abort the operation. If one is chosen, it will be
used to sign a Relying Party-provided challenge and other collected data into an authentication assertion,
which is used as a credential.

Example usage:
```csharp
var clientData = new ClientData(
    new byte[] /* Client data JSON as a byte array. */,
    HashAlgorithm.Sha256);
var pubkeyCredRequestOptions = new PubkeyCredCreationOptions(
    0 /* Timeout */,
    "Relying Party id",
    new List<PubkeyCredDescriptor>() /* Existing credentials for this user */,
    new AuthenticationSelCri(/* Capabilities and settings that the authenticator MUST or SHOULD satisfy */),
    UserVerificationRequirement.Preferred,
    new List<PubkeyCredHint>(),
    AttestationPref.Indirect,
    new List<byte[]>() /* Attestation format info */,
    new List<AuthenticationExt>() /* Extensions */,
    new HybridLinkedData(
        new byte[] /* CBOR:"1" */,
        new byte[] /* CBOR:"2" */,
        new byte[] /* CBOR:"3" */,
        new byte[] /* CBOR:"4" */,
        new byte[] /* CBOR:"5" */,
        new byte[] /* CBOR:"6" */,
        new byte[] /* Domain String of tunnel server */,
        new byte[] /* Identity Key created during QR initiated transaction */
    ));

var callbacks = new GaCallbacks(
    (string str, object obj) =>
        {
            /* A callback responsible for displaying the qr code. */
        },
    (PubkeyCredentialAssertion att, WauthnError err, object obj) =>
        {
            /* Callback function for getting the final response. */
        },
    (HybridLinkedData att, WauthnError err, object obj) =>
    {
        /* Callback function for getting the updated linked device data. */
    },
    "some data" /* Any user defined object that will be used by the callbacks */);

try
{
    Authenticator.GetAssertion(clientData, pubkeyCredCreationOptions, callbacks);
}
catch (Exception e)
{
    // Error handling
}
```

Refer to the following W3C specification for more information:
[https://www.w3.org/TR/webauthn-3/#sctn-getAssertion].

## Related information
* Dependencies
  - Tizen 9.0 and Higher