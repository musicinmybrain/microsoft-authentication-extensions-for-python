
# Microsoft Authentication Extensions for Python

The Microsoft Authentication Extensions for Python offers secure mechanisms for client applications to perform cross-platform token cache serialization and persistence. It gives additional support to the [Microsoft Authentication Library for Python (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-python).

MSAL Python supports an in-memory cache by default and provides the [SerializableTokenCache](https://msal-python.readthedocs.io/en/latest/#msal.SerializableTokenCache) to perform cache serialization. You can read more about this in the MSAL Python [documentation](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-python-token-cache-serialization). Developers are required to implement their own cache persistence across multiple platforms and Microsoft Authentication Extensions makes this simpler.

The supported platforms are Windows, Mac and Linux.
- Windows - [DPAPI](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection) is used for encryption.
- MAC - The MAC KeyChain is used.
- Linux - [LibSecret](https://wiki.gnome.org/Projects/Libsecret) is used for encryption.

> Note: It is recommended to use this library for cache persistance support for Public client applications such as Desktop apps only. In web applications, this may lead to scale and performance issues. Web applications are recommended to persist the cache in session. Take a look at this [webapp sample](https://github.com/Azure-Samples/ms-identity-python-webapp).

## Installation

You can find Microsoft Authentication Extensions for Python on [Pypi](https://pypi.org/project/msal-extensions/).
1. If you haven't already, [install and/or upgrade the pip](https://pip.pypa.io/en/stable/installing/)
   of your Python environment to a recent version. We tested with pip 18.1.
2. Run `pip install msal-extensions`.

## Versions

This library follows [Semantic Versioning](http://semver.org/).

You can find the changes for each version under
[Releases](https://github.com/AzureAD/microsoft-authentication-extensions-for-python/releases).

## Usage

### Creating an encrypted token cache file to be used by MSAL

The Microsoft Authentication Extensions library provides the `PersistedTokenCache` which accepts a platform-dependent persistence instance. This token cache can then be used to instantiate the `PublicClientApplication` in MSAL Python.

The token cache includes a file lock, and auto-reload behavior under the hood.



Here is an example of this pattern for multiple platforms (taken from the complete [sample here](https://github.com/AzureAD/microsoft-authentication-extensions-for-python/blob/dev/sample/token_cache_sample.py)):

```python
def build_persistence(location, fallback_to_plaintext=False):
    """Build a suitable persistence instance based your current OS"""
    try:
        return build_encrypted_persistence(location)
    except:
        if not fallback_to_plaintext:
            raise
        logging.warning("Encryption unavailable. Opting in to plain text.")
        return FilePersistence(location)

persistence = build_persistence("token_cache.bin")
print("Type of persistence: {}".format(persistence.__class__.__name__))
print("Is this persistence encrypted?", persistence.is_encrypted)

cache = PersistedTokenCache(persistence)
```
Now you can use it in an MSAL application like this:
```python
app = msal.PublicClientApplication("my_client_id", token_cache=cache)
```

### Creating an encrypted persistence file to store your own data

Here is an example of this pattern for multiple platforms (taken from the complete [sample here](https://github.com/AzureAD/microsoft-authentication-extensions-for-python/blob/dev/sample/persistence_sample.py)):

```python
def build_persistence(location, fallback_to_plaintext=False):
    """Build a suitable persistence instance based your current OS"""
    try:
        return build_encrypted_persistence(location)
    except:  # pylint: disable=bare-except
        if not fallback_to_plaintext:
            raise
        logging.warning("Encryption unavailable. Opting in to plain text.")
        return FilePersistence(location)

persistence = build_persistence("storage.bin", fallback_to_plaintext=False)
print("Type of persistence: {}".format(persistence.__class__.__name__))
print("Is this persistence encrypted?", persistence.is_encrypted)

data = {  # It can be anything, here we demonstrate an arbitrary json object
    "foo": "hello world",
    "bar": "",
    "service_principle_1": "blah blah...",
    }

persistence.save(json.dumps(data))
assert json.loads(persistence.load()) == data
```

## Python version support policy

Python versions which are 6 months older than their
[end-of-life cycle defined by Python Software Foundation (PSF)](https://devguide.python.org/versions/#versions)
will not receive new feature updates from this library.


## Community Help and Support

We leverage Stack Overflow to work with the community on supporting Azure Active Directory and its SDKs, including this one!
We highly recommend you ask your questions on Stack Overflow (we're all on there!).
Also browse existing issues to see if someone has had your question before.

We recommend you use the "msal" tag so we can see it!
Here is the latest Q&A on Stack Overflow for MSAL:
[http://stackoverflow.com/questions/tagged/msal](http://stackoverflow.com/questions/tagged/msal)


## Contributing

All code is licensed under the MIT license and we triage actively on GitHub.

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.


## We value and adhere to the Microsoft Open Source Code of Conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
