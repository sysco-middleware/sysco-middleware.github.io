---
layout: post
title: Managing secrets using Google Cloud KMS
categories: google-cloud-platform, security, golang
tags: [security, golang, google-cloud-platform]
author: PrakharSrivastav
---


# Introduction

At [Sysco AS](https://sysco.no/), we are developing a serverless integration platform on GCP. Being the integration provider between third parties, a common requirement is to securely save user secrets (eg API-keys) so that we can call the third-party services on behalf of users. In order to securely save these credentials with us, we have a few requirements:

- Save customer secrets securely.(Encryption)
- Use the saved secrets and call third party services as required. (Decryption)
- Use managed services for PKI infrastructure. We did not want to manage the keys and the key-pairs ourselves.

In this post, I will explain, how Google Cloud [Key Management Service](https://cloud.google.com/kms/) enabled us to achieve this. All the code used in this post is available [here](https://github.com/sysco-middleware/post-gcp-kms).

# Architecture

The figure below provides an extremely simplified version of our application.


<img src="/images/2019-08-21-gcp-kms-encryption/KMS-secret.png" alt="kms secret" position="center" style="border-radius: 2px;margin-top:5px;" >

1. We have a write application, that saves user credentials in the datastore. It does below operations.
    1. Accepts user input.
    2. Encrypts the user input using Google Cloud KMS - Encryption.
    3. Stores the results in datastore.

2. At several other places in our application, we have read functionality that
    1. Reads the encrypted secrets from datastore.
    2. Decrypts them using Google Cloud KMS - Decryption.
    3. Uses the secrets as needed.

# Setup

In order to start with KMS, we should understand the object hierarchy used in the Google Cloud KMS ecosystem.

- **Projects**: All the objects in the hierarchy belong to a project.
- **Location**: It defines the geographical location (data center) where the keys will be stored and served from.
- **Key Ring**: Represents a set of keys. This is a logical way to group key (for example specific department, functional domain, user group, etc).
- **Key**: The cryptographic key used for encryption and decryption.
- **Key Version** : Numerical version linked to a key.

In order to set these objects, we need to follow below steps:

1. Enable Cloud KMS API.
2. Provision two service-accounts. First to create KeyRing and Keys, and another to allow encryption and decryption operations.
3. Create a "key-ring" in a specific "project" and "location".
4. We then create the required "key" under the "key-ring".

We additionally create one more service account for [Cloud Datastore](https://cloud.google.com/datastore/) to save user data. In the next few sections, we will get into the implementation details of these steps.

## Enable KMS API

Go to the Google Cloud Console. Chose a correct project and enable the [Cloud KMS API](https://console.developers.google.com/apis/api/cloudkms.googleapis.com/overview).

## Creating required service accounts for KMS and Datastore.

We start by creating three service accounts.

- kms-admin with role Cloud KMS Admin.
- kms-enc-dec with a role Cloud KMS CryptoKey Encrypter/Decrypter.
- datastore-user with a role Cloud Datastore User.

If the users are following the code in the [github](https://github.com/PrakharSrivastav/gcp-kms-encryption-decryption), they should create these service-accounts and save the three credential files under the project root.

#### KMS - Admin service account

Create a service account with role Cloud KMS Admin. Download the service account json file. Save it as `kms-admin.json` under the project root.

<img src="/images/2019-08-21-gcp-kms-encryption/kms-admin-2.png" alt="Cloud KMS Admin - add role" style="border-radius: 2px;margin-top:5px;" >

#### KMS - Encrpyt/Decrypt service account

Create a service account with role Cloud KMS CryptoKey Encrypter/Decrypter. Download the service account json file. Save it as `kms-enc-dec.json` under the project root.

<img src="/images/2019-08-21-gcp-kms-encryption/kms-encdec-2.png" alt="Cloud KMS Enc/Dec - add role"  style="border-radius: 2px;margin-top:5px;" >

#### Cloud Datastore User service account

Create a service account with role Cloud Datastore User. Download the service account json file. Save it as `datastore-user.json` under the project root.

<img src="/images/2019-08-21-gcp-kms-encryption/datastore-user.png" alt="Datastore USer - add role" style="border-radius: 2px;margin-top:5px;" >

# Creating key and key-ring

We first create a key-ring using KMS client and then add a key to it. Note that we provide the purpose of the key as `CryptoKey_ENCRYPT_DECRYPT` and use the encryption algorithm as `CryptoKeyVersion_GOOGLE_SYMMETRIC_ENCRYPTION`.

```go
// create KMS client
if client, err = cloudkms.NewKeyManagementClient(
   context.Background(),
   option.WithCredentialsFile("../kms-admin.json"));err != nil {
  log.Fatal(err)
}
defer client.Close()

// first create a key ring. Provide project-name and location
pKeyRing := fmt.Sprintf("projects/%s/locations/%s","gcp-project-name", "global")
if result, err = client.CreateKeyRing(context.Background(), &kmspb.CreateKeyRingRequest{
   Parent:pKeyRing,
   KeyRingId: "key-ring-name"});err != nil {
  log.Fatalf("error creating keyring %s", err)
}

// prepare a request for creating a symmetric key
pKey := fmt.Sprintf("projects/%s/locations/%s/keyRings/%s", "gcp-project-name", "global", "key-ring-name")
r := &kmspb.CreateCryptoKeyRequest{
  Parent:       pKey,
  CryptoKeyId: "key-name",
  CryptoKey: &kmspb.CryptoKey{
    Purpose:         kmspb.CryptoKey_ENCRYPT_DECRYPT, // Provide the purpose for the key
    VersionTemplate: &kmspb.CryptoKeyVersionTemplate{
      Algorithm: kmspb.CryptoKeyVersion_GOOGLE_SYMMETRIC_ENCRYPTION
    },
  },
}

// finally create the key
if res, err = client.CreateCryptoKey(context.Background(), r);err != nil {
  log.Fatal(err)
}
log.Printf("created crypto key %s", res)
```

# Encrypting password and storing it in the datastore

Before starting with the encryption and decryption, Let us define our entity that we want to store in the database. The structure for our user entity is.

```go
type User struct {
  Username string `json:"username"`
  Password []byte `json:"password"`
}
```

We want to store the password as sequence of bytes which can only be encrypted using the KMS keys. Therefore the type of the password field is []byte. This way we always store the password as bytes (after encryption) and decrypt them using KMS key in order to call third parties. To encrypt the credentials, we will use the service account kms-enc-dec, which has a limited scope to encrypt or decrypt the password. The following code snippet shows how to encrypt the password and store it as a datastore entity.

```go
// create kms client
ctx := context.Background()
if kms, err = cloudkms.NewKeyManagementClient(
  ctx,
  option.WithCredentialsFile("../kms-enc-dec.json"));err != nil {
  log.Fatal(err)
}
defer kms.Close()

// encrypt the password. Use the project, key-ring and key-name as setup previously
p := fmt.Sprintf("projects/%s/locations/%s/keyRings/%s/cryptoKeys/%s",
  "project-name",
  "location",
  "key-ring-name",
  "key-name")
req := &kmspb.EncryptRequest{Name:p,Plaintext: []byte("MySecretPassword")}
if resp, err = kms.Encrypt(ctx, req);err != nil {
  log.Fatal(err)
}

// Use encrypted password to prepare user entity and save it to datastore
u := User{Username: "test@test.com", Password: resp.GetCiphertext()}
if ds, err = datastore.NewClient(
  ctx,
  "gcp-project-name",
  option.WithCredentialsFile("../datastore-user.json")); err != nil {
  log.Fatal(err)
}
defer ds.Close()
key := datastore.IDKey("users", 1, nil)
if _, err = ds.Put(ctx, key, &u); err != nil {
  log.Fatalf("ds err %v", err)
}
```
You could see the saved password in datastore as



<img src="/images/2019-08-21-gcp-kms-encryption/datastore-enc.png" alt="Datastore User - add role" style="border-radius: 2px;margin-top:5px;" >

# Reading datastore and decrypting password before using

In the other parts of the application where we use the secrets on behalf of the user, we simply read the byte sequence from the datastore and decrypt them before using.

```go
ctx := context.Background()
if ds, err = datastore.NewClient(
  ctx,
  "gcp-project",
  option.WithCredentialsFile("../datastore-user.json")); err != nil {
   log.Fatal(err)
}
defer ds.Close()
key := datastore.IDKey("users", 1, nil)

u := User{}
if err = ds.Get(ctx, key, &u); err != nil {
  log.Fatalf("ds err %v", err)
}

if client, err := cloudkms.NewKeyManagementClient(
  ctx,
  option.WithCredentialsFile("../kms-enc-dec.json")); err != nil {
  log.Fatal(err)
}
defer client.Close()

p := fmt.Sprintf("projects/%s/locations/%s/keyRings/%s/cryptoKeys/%s",
  "project-name",
  "location",
  "key-ring-name",
  "key-name")
r := &kmspb.DecryptRequest{Name: p,Ciphertext: u.Password}
if resp, err := client.Decrypt(ctx, r); err != nil {
  log.Println(err)
}
log.Println(string(resp.Plaintext))
```
You should see the output as:
```bash
prakhar@tardis (master)✗ [2] % go run main.go    
2019/08/19 20:02:42 MySecretPassword
```

# Conclusion

With Cloud KMS, setting up and maintaining the PKI infrastructure is extremely. Cloud KMS, abstracts away all the underlying complexities and provides a set of simple APIs to easily manage cryptographic keys for cloud services as well as for your on-premise applications. You can generate, use, rotate, and destroy AES256, RSA 2048, RSA 3072, RSA 4096, EC P256, and EC P384 cryptographic keys. Cloud KMS is integrated with Cloud IAM and Cloud Audit Logging so that you can manage permissions on individual keys and monitor how these are used.

I encourage the readers of this post to give it a try.


# References

- [gcp: kms-object-hierarchy](https://cloud.google.com/kms/docs/object-hierarchy)
- [github repository](https://github.com/sysco-middleware/post-gcp-kms)
- [gcp: kms roles](https://cloud.google.com/kms/docs/reference/permissions-and-roles#custom_roles)

<small>Note: This content originally appeared [here](https://www.prakharsrivastav.com/posts/gcp-using-kms-to-manage-secrets/). It has been altered at places to suite guidelines of this forum.</small>