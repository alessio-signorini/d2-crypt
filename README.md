# D2-CRYPT

This tool has been developed as proof-of-concept to transfer files securely between two parties. It uses static key encryption (AES-256-GCM) and compression (ZLib). The key is encrypted and signed using public-key encryption (RSA). Files, key and signature are then packaged in a Zip file.

It mostly uses native Ruby libraries with the exception of ``zip`` which is used to package all the encrypted files into a single archive.

## Archive Specifications
Each archive contains the encrypted AES key (``key``), its signature (``sign``) and a list of files (``file1`` to ``fileN``).

The 32 bytes key is encrypted using AES-256-GCM. The public key of the recipient is used to encrypt the AES key and the result is saved in ``key``. The private key of the sender is used to sign the result of the encryption and the 256 bytes signature is saved in ``sign``.

Each encrypted file has metadata (``.info``) and encrypted data (``.data``). There is no specific format for the metadata, but it is suggested to formatted in JSON and to contain at least:

* original_filename
* original_extension
* creation_date

The files are saved in the archive in this exact order:

* ``key``
* ``sign``
* ``file1.info``
* ``file1.data``
* ...
* ``fileN.info``
* ``fileN.data``

For each file compressed using AES-256-GCM, a random 12 bytes Initialization Vector (IV) is added at the beginning of the file and the 16 bytes Auth Tag is added to the end.

````
   12bytes IV | data | 16bytes Auth Tag
````

## Usage Examples

If the tool is launched without parameters it displays an online help. The tool expects to find the following files in its directory:
* ``local-private-key.pem`` - sender private key
* ``local-public-key.pem`` - sender public key
* ``remote-public-key.pem`` - recipient public key

Those can be symlinks but they need to be located in the same directory of the script. The tool can then be used from anywhere in the filesystem. Here are a few examples:

To encrypt files using ``d2-encrypt.rb`` one can type
````
d2-encrypt video.data video.info
d2-encrypt video.data video.info audio.data audio.info
````

To decrypt files using ``d2-decrypt.rb`` one can type
````
d2-decrypt b3811a03-cc9b-4e67-ad8e-e97d22b0668f.enc
d2-decrypt b3811a03-cc9b-4e67-ad8e-e97d22b0668f.enc /tmp
````

To test locally, just symlink ``local-public-key.pem`` into ``remove-public-key.pem`` so you will be able to decrypt what you have encrypted.

## How to generate a Public Key Pair
You can use ``openssl`` to generate the private key
````
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048
````
and then extracted the public key with this command
````
openssl rsa -pubout -in private_key.pem -out public_key.pem
````
