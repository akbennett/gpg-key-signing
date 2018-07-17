# GPG Github Setup

The purpose of the document is to create a secure and healthy PGP set of keys to manage GitHub identities and have the `Verified` status on there.
We want to end up with a set of 3 keys:

1. __Master Key__ used to certify other keys and to create and revoke subkeys.
2. __Signing SubKey__ this subkey will be used to sign our commits.
3. __Encryption SubKey__ not really used in this context, but this is created by default and it could come handy.



We will keep the private part of the Master Key offline to protect our identity against theft or loss. Read more about subkeys [here](http://www.connexer.com/articles/openpgp-subkeys).

__This walkthrough refers to `gpg (GnuPG) 2.1.18`, please make sure to have the latest stable GnuPG2 installed for your distribution.__

## 1. Create your PGP Key

Even if you alredy have a GPG key-pair, it would be wise to have a IOTA-specific pair. If you still want to use your existing keypair, please make sure to understand how PGP subkeys work and skip to section 2.

Let's just generate a RSA 4096 key that never expires, so we are able to sign and encrypt. While the master key will never expire, the keys we will actually use on a daily basis will expire.

```bash
 $ gpg2 --full-generate-key
```

Select options like in here:

```
gpg (GnuPG) 2.1.18; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Tyler Baker
Email address: tyler@foundries.io
Comment: Tyler Baker Foundries Identity
You selected this USER-ID:
    "Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
```

After this the key will be generated after the OS obtains some entropy.

We can check the newly created key with:

```bash
$ gpg2 -K
```

Example output:

```bash
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
/home/youruser/.gnupg/pubring.kbx
---------------------------------
sec   rsa4096 2018-06-14 [SC]
      B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
uid           [ultimate] Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>
ssb   rsa4096 2018-06-14 [E]
```

This means we have a secret master key of type RSA 4096 associated to the `tyler@foundries.io` identity. The master key can be used to sign and certify `[SC]`.
There is also a secret subkey `ssb`, that can be used to encrypt data `[E]`.
The ID of the key is `B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD`.

### 1.1 Send your key to a KeyServer

For others to be able to have your identity handy, upload the newly created public key to a keyserver:

```
$ gpg2 --keyserver hkp://keys.gnupg.net --send-keys B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
```

## 2. Configure everyday-use subkeys

We do not want to use our master key for everyday use, we want to have subkeys that expire to sign and encrypt instead.

```
$ gpg2 --edit-key B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
```

```
gpg (GnuPG) 2.1.18; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  rsa4096/2E1920CC6D2B72DD
     created: 2018-06-14  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/2FBE06F3FC085E04
     created: 2018-06-14  expires: never       usage: E   
[ultimate] (1). Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>

gpg> 
```

Currently we have a master key and an encryption subkey, no expiration date is set for neither of them.

At continuation we set an expiration date for the first subkey and we create a signing subkey.

```
gpg> key 1

sec  rsa4096/2E1920CC6D2B72DD
     created: 2018-06-14  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb* rsa4096/2FBE06F3FC085E04
     created: 2018-06-14  expires: never       usage: E   
[ultimate] (1). Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>

gpg> expire
Changing expiration time for a subkey.
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Fri 14 Jun 2019 11:37:16 AM UTC
Is this correct? (y/N) y

sec  rsa4096/2E1920CC6D2B72DD
     created: 2018-06-14  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb* rsa4096/2FBE06F3FC085E04
     created: 2018-06-14  expires: 2019-06-14  usage: E   
[ultimate] (1). Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>

gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Fri 14 Jun 2019 11:37:31 AM UTC
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
```

You should end up in a similar configuration:

```
gpg> list

sec  rsa4096/2E1920CC6D2B72DD
     created: 2018-06-14  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb* rsa4096/2FBE06F3FC085E04
     created: 2018-06-14  expires: 2019-06-14  usage: E   
ssb  rsa4096/97A4AC4092576B55
     created: 2018-06-14  expires: 2019-06-14  usage: S   
[ultimate] (1). Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>

gpg> save
```

Don't forget to save the changes by issuing a `gpg> save` to the prompt.

## 3. Remove secret Master Key from device

We will proceed by exporting Master Key's revocation certificate and secret portion. We will then proceed to delete the secret Master Key from our keyring. You should store this Master Key offline, separatly from your main device, a location which is both easy to remember and accessible. The Master Key's secret is essentially the certificate for your identity, and you do not want to compromise your entire identity if your device is stolen or lost.
The Master Key's secret will be needed to:

1. Generate new subkeys.
2. Revoke subkeys.
3. Revoke Master Key.
4. Sign other people's Master Keys.

__You will need the Master Key's backup to participate in the SumSum Key Signing party!__

### 3.1 Generate Revocation Certificate
Let's generate a revocation certificate for the Master Key. This will be the only way to retract your Master Key in case you loose it.

```
$ gpg2 --output tyler.baker.foundries.gpg-revocation-certificate --gen-revoke tyler@foundries.io
```

Select these options:

```
sec  rsa4096/2E1920CC6D2B72DD 2018-06-14 Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>

Create a revocation certificate for this key? (y/N) y
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
(Probably you want to select 1 here)
Your decision? 0
Enter an optional description; end it with an empty line:
> 
Reason for revocation: No reason specified
(No description given)
Is this okay? (y/N) y
ASCII armored output forced.
File 'tyler.baker.foundries.gpg-revocation-certificate' exists. Overwrite? (y/N) y
Revocation certificate created.

Please move it to a medium which you can hide away; if Mallory gets
access to this certificate he can use it to make your key unusable.
It is smart to print this certificate and store it away, just in case
your media become unreadable.  But have some caution:  The print system of
your machine might store the data and make it available to others!
```

__Store the file offline and delete it from the device.__

### 3.2 Backup Master Key's secret

Let's export the secret portion of our Master Key:

```
$ gpg2 --export-secret-keys --output tyler.baker.secret.gpg --armor B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
```

The exported key file is encrypted with the passphrase you chose during the key's creation.

__Store the file in a safe, offline and easy-to-remember place and delete it from the device. Please note that the backup should be easily accessible from you at any time.__

### 3.3 Remove Master Key's secret from keyring

```
$ gpg2 --delete-secret-key B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
```

`gpg` will interactively prompt what secret keys you want to delete.
__REMOVE THE MASTER KEY'S SECRET ONLY, LEAVING SUBKEYS ALONE__.

Example output:

```
gpg (GnuPG) 2.1.18; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


sec  rsa4096/2E1920CC6D2B72DD 2018-06-14 Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>

Delete this key from the keyring? (y/N) y
This is a secret key! - really delete? (y/N) y
gpg: deleting secret subkey failed: Operation cancelled
gpg: B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD: delete key failed: Operation cancelled
```

To check that everything went according to plan issue:

```
$ gpg2 -K
```

You should have an output like this:

```
/home/youruser/.gnupg/pubring.kbx
---------------------------------
sec#  rsa4096 2018-06-14 [SC]
      B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
uid           [ultimate] Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>
ssb   rsa4096 2018-06-14 [E] [expires: 2019-06-14]
ssb   rsa4096 2018-06-14 [S] [expires: 2019-06-14]
```

The `#` next to the Master Key's means that the secret part of that key is not present in the keyring anymore, therefore it is not usable. This is what we wanted: generating two complete keypairs for everyday's use, while retaining only the public portion of the the Master Key on our device.

### 3.4 Test the key backup

After the deleting the private part, the Master Key is not usable anymore to  modifying or adding subkeys.

To verify your backup, retrieve it from your secure offline storage and issue:

```
$ gpg2 --import tyler.baker.secret.gpg
```

Example output:

```
gpg: key 2E1920CC6D2B72DD: "Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>" 1 new signature
gpg: key 2E1920CC6D2B72DD: "Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>" 1 new subkey
gpg: key 2E1920CC6D2B72DD: secret key imported
gpg: Total number processed: 1
gpg:            new subkeys: 1
gpg:         new signatures: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
gpg:  secret keys unchanged: 1
```

By listing the private keyring once again we can see that the `#` sign has disappeared, meaning the Master Key is once again usable.

```
$ gpg2 -K
/home/youruser/.gnupg/pubring.kbx
---------------------------------
sec   rsa4096 2018-06-14 [SC]
      B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
uid           [ultimate] Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>
ssb   rsa4096 2018-06-14 [S] [expires: 2019-06-14]
ssb   rsa4096 2018-06-14 [E] [expires: 2019-06-14]
```

### 3.5 Move Key's secret to Yubikey 4

Make sure the `pcscd` (On Linux) / `CryptoTokenKit` or `pcsc-lite` on Mac OSX SmartCard middleware is running on your host to interact with the smartcard.

If you don't have your YubiKey, one will be provided as part of the Welcome goodies at the SumSum 2018, and you can perform the following steps then.

Plug your Yubikey in!

#### 3.5.1 Change OpenPGP PIN and Admin PIN

Yubikey cards implement several modules and protocols for you to interact with the card. We will be interacting with the OpenPGP module of the card through the `gpg` cli.

```
$ gpg2 --edit-card
[BLURRED]
gpg/card> 
```

Information of the card will be listed.

We will now proceed to change the PIN and the Admin PIN from the default values. The default Admin PIN on a brand new Yubikey 4 defaults to `12345678`.

```
gpg/card> admin
Admin commands are allowed
gpg/card> passwd
gpg: OpenPGP card no. [BLURRED] detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 3
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 2
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? q
```

Please beware that issuing the Admin PIN incorrectly for several times will lock the contents of the card, and factory reset will be necessary.

#### 3.5.2 Change card's identity information

Let's make the key personal by adding your personal info:

```
gpg/card> name
Cardholder's surname: Baker
Cardholder's given name: Tyler

gpg/card> lang
Language preferences: en

gpg/card> sex
Sex ((M)ale, (F)emale or space): m

gpg/card> login
Login data (account name): andrea

gpg/card> quit
```

#### 3.5.3 Storing the secret on YubiKey 4

You can choose either of the following options:

1. Yubikey with Sub Key inside (_reccomended_): Master Key's secret is on an generic backup storage. Subkey is carried around with the Yukibey, regardless of the device. Loosing the backup will lead to entire identity compromise.
1. Yubikey with Master Key inside: well-identified object and often carried around. Subkeys on device, you will need to create / copy subkey per device you use. If you loose the Yubikey your entire identity is compromised.

##### 3.5.3.1 Moving Signing Sub Key's secret (reccomended)

Please, take a moment to understand the implications of this step: every time you will need to sign other's people identities, alter, revoke or generate subkeys for your identity you will have to recur to the offline Master Key backup you performed in section `3.2`. __Loosing your Yubikey won't lead to an identity compromise.__

```
gpg2 --edit-key B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
gpg (GnuPG) 2.1.18; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

pub  rsa4096/2E1920CC6D2B72DD
     created: 2018-06-14  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/97A4AC4092576B55
     created: 2018-06-14  expires: 2019-06-14  usage: S   
ssb  rsa4096/2FBE06F3FC085E04
     created: 2018-06-14  expires: 2019-06-14  usage: E   
[ultimate] (1). Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>

gpg> key 2

pub  rsa4096/2E1920CC6D2B72DD
     created: 2018-06-14  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/97A4AC4092576B55
     created: 2018-06-14  expires: 2019-06-14  usage: S   
ssb* rsa4096/2FBE06F3FC085E04
     created: 2018-06-14  expires: 2019-06-14  usage: E   
[ultimate] (1). Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>

gpg> keytocard 
Please select where to store the key:
   (1) Signature key
   (3) Authentication key
Your selection? 1

gpg> save
```

After this step you should end up in the followin situation:

```
$ gpg2 -K
/home/youruser/.gnupg/pubring.kbx
---------------------------------
sec#  rsa4096 2018-06-14 [SC]
      B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
uid           [ultimate] Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>
ssb   rsa4096 2018-06-14 [S] [expires: 2019-06-14]
ssb>  rsa4096 2018-06-14 [E] [expires: 2019-06-14]
```

The `>` symbol next to your Sub Key means that the secret is safely stored on the SmartCard. Notice that, by now, you should also have backed-up and removed the Master Key secret from the device as explained in section `3.3`, hence the `#` symbol next to the Master Key.

##### 3.5.3.2 Moving Master Key's secret

Please, take a moment to understand the implications of this step: __if your Yubikey is lost or stolen, you will have to assume the compromise of your entire identity__.

```
gpg2 --edit-key B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
gpg (GnuPG) 2.1.18; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  rsa4096/2E1920CC6D2B72DD
     created: 2018-06-14  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/97A4AC4092576B55
     created: 2018-06-14  expires: 2019-06-14  usage: S   
ssb  rsa4096/2FBE06F3FC085E04
     created: 2018-06-14  expires: 2019-06-14  usage: E   
[ultimate] (1). Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>

gpg> keytocard 
Really move the primary key? (y/N) y
Please select where to store the key:
   (1) Signature key
   (3) Authentication key
Your selection? 1

gpg> save
```

After this step you should end up in the followin situation:

```
$ gpg2 -K
/home/youruser/.gnupg/pubring.kbx
---------------------------------
sec>  rsa4096 2018-06-14 [SC]
      B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
uid           [ultimate] Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>
ssb   rsa4096 2018-06-14 [S] [expires: 2019-06-14]
ssb   rsa4096 2018-06-14 [E] [expires: 2019-06-14]
```

The `>` symbol next to your Master Key means that the secret is safely stored on the SmartCard.

## 4. Use GPG for coding

### 4.1 Upload your key to GitHub

Export your Master Key's public part:

```
$ gpg2 --export --armor B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
```

Copy and paste the output as a GPG key under your GitHub profile.


### 4.2 Configure git for signing

We now want to configure `git` to make use of the signing GPG subkey, corresponding to the Master Key we uploaded to GitHub. Since we deleted the private portion of the Master Key, a valid signing subkey will be used instead. 

```
$ gpg2 -K
```

Output:

```
/home/youruser/.gnupg/pubring.kbx
---------------------------------
sec>  rsa4096 2018-06-14 [SC]
      B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
uid           [ultimate] Tyler Baker (Tyler Baker Foundries Identity) <tyler@foundries.io>
ssb   rsa4096 2018-06-14 [E] [expires: 2019-06-14]
ssb   rsa4096 2018-06-14 [S] [expires: 2019-06-14]
```

Copy the ID corresponding to the key, `B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD` in this case.

Get into your `` repo folder and let's alter the local `git` configuration.

```
$ git config user.email tyler@foundries.io
$ git config user.name 'Tyler Baker'
$ git config user.signkey B3DFA611285A0FF08D4151AE2E1920CC6D2B72DD
$ git config commit.gpgSign true
```

From now on, every commit you issue from this device will be signed with the configured PGP key.
Since you uploaded the public part to GitHub, they will be marked as _Verified_.

## 5. Expiration?

If you followed all the above, your signing subkey will expire 1 year after creation. When the expiration date approaches you can either extend the current key or create a new subkey starting from the same Master Key. Please note that you will need to use the Master Key's private you safely stored offline to perform these actions.
Since the renewed or new key originates from the same Master Key, the world will recognise the new signatures as belonging to the same Master Key's public part, therefore same identity.

## 6. GitHub Releases

Realeses on GitHub are performed manually from the web interface. As long as a release points to a _Verified_ commit, the release will also be consiered Kosher.

### 6.1 Signatures and Checksums

If binary files are attached as part of the release, it is good practice to generate a detached armored-ASCII signature corresponding to the user issuing the release.

```
$ gpg2 --detach-sign --armor lmp-gateway-image-raspberrypi3-64.img.gz tyler@foundries.io
```

This will generate a `lmp-gateway-image-raspberrypi3-64.img.gz.asc` signature file for my `tyler@foundries.io` identity to be included in the release.

As part of the text of the release, you should also provide MD5 and SHA256 checksums of the supplied binary files.

```
$ md5sum lmp-gateway-image-raspberrypi3-64.img.gz
$ sha256sum lmp-gateway-image-raspberrypi3-64.img.gz
```
