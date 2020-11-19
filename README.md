key management service

kms is used to create, manage cryptographic keys

these cryptographic keys convert plaintext to chippertext and vice versa

kms support both symmetric and asymmetric keys 

kms performs actual cryptographic operations(like encryption, decryption etc)

cryptographic keys never leaves kms(<- important)

kms create, manage, import and perform operations on these keys, but the keys NEVER leave kms

kms primary purpose is to contain and secure the keys with in it

kms provide a FIPS 140-2 level two, service -> This is a US security standard

some kms features are compliant with level three also, but mostly work with level two.

----------
Managed customer master keys (CMK)

kms mainly manages cmk or customer master keys

these cmks are used by kms within cryptographic operations

you,applications can use these cmk

cmk are logical 

cmk is a container for actual physical material keys-This material key is, what acually KMS stores

cmk contains id, creationdate, key policy(type of resource policy), description, state of the key(active or inactive)

every cmk holds a physical key material

this material which is contained inside a kms is used by kms to encrypt or decrypt data for upto **4kb in size**

cmk is only used to encrypt or decrypt data that is a maximum of 4 kb in size <- important

##the process##

(see the cmk diagram to understand the process visually)

##Scenario 1

Ali runs createKey operation and inside kms, cmk is created which contains physical key maerial 

after creation,cmk are never stored in persistant state,kms will take this cmk and encrypt it before it's stored persistanently on disk

## Scenario 2

Now ali want to encrypt some data. he will make encrypt call to kms with the data he wants to encrypt

kms will accept this data assuming ali has permission to use the created cmk

kms will decrypt the key(which is stored persistanently on disk in encryption form) and will use that decrypted cmk to encrypt the plianText that ali has supplied, and return it to ali

## Scenario 3

Now in future ali want to decrypt the data in the future maybe,he will make decrypt call to kms with the data he wants to decrypt

Ali dont need to send information that which cmk will be used to decrypt the data. because that information is always available in encrypted chippertext which ali wants to decrypt.

kms will decrypt the cmk(which is stored persistanently on disk in encryption form) and will use that decrypted cmk to decrypt the chippertext that ali has supplied, and return the plaintext to ali

-----------

These above scenarios are for different operations(create cmk operation, encrypt data operation, decrypt data operation). This is called Role Separation

---------------------

## Data Encryption Keys (DEK)

As you have noted the DEK is used when you need to encrypt data larger than 4kB (which is the maximum as CMK can do)

A DEK is generated and can be encrypted / decrypted using a CMK

Unlike CMKs, AWS does not manage DEKs for you, they are simply returned to you.(<- important)

DEK gives two versions:

- 1: PlainText version: used to encrypt data
- 2: CypherText version: encrypted dek is placed side by side with encrypted data

When you need to do this, you are expected to do something like:

-- Storage --

- Generate a DEK
- Use the DEK to encrypt the data
- Store an encrypted version of the DEK along with the encrypted data
-Delete (discard) the unencrypted version of the DEK

-- Retrieval --

- Access data + encrypted DEK
- Use the CMK to decrypt the DEK
- Use the (now decrypted) DEK to decrypt the data
- This pattern means that even if someone has access to the encrypted data and the encrypted DEK, they still cannot access the data without the CMK.
- The key here is that AWS know that you need to do 2 things.
- 1. Use the DEK to encrypt data - this requires a PLAINTEXT version
- 2. Store an encrypted version of the DEK with the data - this requires a CIPHERTEXT version.

Thus, the reason they are both returned is to make your life easier.

If only the CIPHERTEXT version was returned, it would need to be decrypted before it could be used.

If only the PLAINTEXT version was returned, it would need to be encrypted before it could be persisted anywhere.


