# GPG: Renew key for multiple YubiKeys

In short, transferring a key to a smartcard is a destructive operation in GnuPG. Therefore we have to follow a special procedure of

1. delete old secret key stubs
2. restore full key from backup
3. renew full key
4. backup the full key
5. run key-to-card for the first YubiKey
6. delete new secret key stubs
7. restore full key from backup
8. run key-to-card for the second YubiKey

Repeat step 6-8 for any further YubiKey

## Before you begin

Unplug your YubiKeys.

## 1. Delete old secret key stubs

Check using `gpg --list-secret-keys` (or `gpg -K`) whether the key is just a
stub.  In that case the output starts with `ssb>`.  Proceed to delete both
secret and public keys.

```console
$ gpg --list-secret-keys "${KEYID}"
sec   rsa4096/...
...
ssb>  rsa4096/...
ssb>  rsa4096/...
ssb>  rsa4096/...
$ gpg --delete-secret-and-public-keys "${KEYID}"
```

## 2. Restore full key from backup

Restore the secret key from backup.  The public key is not so important because
this can always be regenerated from the secret key if necessary.  Check that the
restored key is actually the full key by using `gpg --list-secret-keys` (or `gpg
-K`).  The output should start with `sec` (without trailing `#`) and with `ssb`
(without trailing `>`).

```console
$ gpg --import secret-key-backup.asc
<enter passphrase>
$ gpg --list-secret-keys "${KEYID}"
sec   rsa4096/...
...
ssb   rsa4096/...
ssb   rsa4096/...
ssb   rsa4096/...
```

## 3. Renew full key

Before we renew the key we trust it (because this is our own secret key)

```console
$ gpg --edit-key "${KEYID}"
gpg> trust
...
Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y
gpg> save
```

Now start the next editing session to renew the key.  First the primary key is
renewed.  If there are subkeys, select all of them and also renew their
expiration date.

```console
$ gpg --edit-key "${KEYID}"
gpg> expire
...
Key is valid for? (0) 1y
Key expires at ...
Is this correct? (y/N) y
gpg> key 1
gpg> key 2
gpg> key 3
gpg> expire
Are you sure you want to change the expiration time for multiple subkeys? (y/N) y
...
Key is valid for? (0) 1y
Key expires at ...
Is this correct? (y/N) y
gpg> save
```

## 4. Backup the full key

Now we export the public and private keys to disk.

```console
$ gpg --armor --export "${KEYID}" > public-key-renewed.asc 
$ gpg --armor --export-secret-keys "${KEYID}" > secret-key-renewed.asc
```

You can inspect the backup to see whether all the desired information is there

```console
$ gpg --list-packets secret-key-renewed.asc
```

## 5. Run key-to-card for the first YubiKey

Now it's time to plug the YubiKey in.  See whether it can be detected using

```console
$ gpg --card-status
```

Transfer all the keys to the smartcard using `keytocard`.  The keys have to be
transferred one at a time, so you have to make a dance with
select-transfer-unselect.  Enter the passphrase for the key and the admin PIN
for the smartcard as needed.  When being prompted in which slot to store the key
a selection has to be made, even if there is only a single option.  Just
pressing enter at this prompt will dismiss it and result in nothing being
transferred.

```console
$ gpg --edit-key "${KEYID}"
gpg> key 1
gpg> keytocard
Please select where to store the key:
   (2) Encryption key
Your selection? 2
gpg> key 1
gpg> key 2
gpg> keytocard
Please select where to store the key:
   (1) Signature key
   (3) Authentication key
Your selection? 1
gpg> key 2
gpg> key 3
gpg> keytocard
Please select where to store the key:
   (3) Authentication key
Your selection? 3
gpg> key 3
gpg> save
```

After that is done `gpg --list-secret-keys` should show

```console
$ gpg --list-secret-keys "${KEYID}"
sec   rsa4096/...
...
ssb>  rsa4096/...
ssb>  rsa4096/...
ssb>  rsa4096/...
```

Unplug the YubiKey.

## 6. Delete new secret key stubs

```console
$ gpg --delete-secret-and-public-keys "${KEYID}"
```

## 7. Restore full key from backup

```console
$ gpg --import secret-key-renewed.asc
```

## 8. Run key-to-card for the second YubiKey

Plug in the second YubiKey and repeat step 5 for it.

## Optional final steps

I don't want to have the primary secret key present in my keyring, so I will
again delete everything and only import the public key.

```console
$ gpg --delete-secret-and-public-keys "${KEYID}"
$ gpg --import public-key-renewed.asc
$ gpg --edit-key "${KEYID}"
gpg> trust
Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y
gpg> save
```

Now I have only the stubs left which is sufficient for everything I do.

```console
$ gpg --list-secret-keys "${KEYID}"
sec#  rsa4096/...
...
ssb>  rsa4096/...
ssb>  rsa4096/...
ssb>  rsa4096/...
```
