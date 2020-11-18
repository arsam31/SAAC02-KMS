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

kms generate dek using cmk and GenerateDataKey operation.This dek is used to encrypt data larger than 4 kb

The dek that is generated, is linked to specific cmk.So kms can tell which dek is attached with which cmk

kms does not stores the dek in any way <- (important)

kma provides this dek to you directly and then discards it.

the reason kms discards it, because kms do not actually perform the encryption or decryption of data using dek

kms does not use dek for anything but only geneates them, thats it.

## How dek works

when dek is generated, kms provide two versions of that dek 

1: plainText versions: something that can be used immediately

2: cipherText versions: encrypted version of the same dek

this dek is encrypted by that same customer master key, which generated it and also decrypted by the same cmk

##Architecture of dek

You would generate a dek immediately before you wanted to encrypt something

you take plainText version that dek and encrypt your plainText.That geneates cipherText for your data and then you discards the plainText version of dek(This whole process is done by you not by KMS as the size is larger than 4 kb)

The encrypted DEK(cipherText version) is stored next to the ciphertext generated earlier(on above line). like side by side

you could use same dek to encrypt millions of file or you could generate different dek to encrypt these millions files.

Decrypting that file is simple, you pass encrypted dek to kms and ask to decrypt it using the same cmk that was used to generate it.



