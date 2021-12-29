---
title: "Renewing a SSL Certificate on OSX Server"
date: "2011-12-19T17:00:00.000Z"
description: "OpenSSL magic runes"
warning: ancient
---

This article relies on having a soon-to-expire SSL certificate on an Apple OSX Server.  Ours are running Snow Leopard, and I’m yet to try the whole thing on Lion.

I’ve got to admit, I went through a bit of a rigmarole to do this.

To generate a new certificate, you need a key, and a CSR.

To get the key, you need to export a PKCS12 file from KeychainAccess as ROOT.  Yes, Root. Yes, OSX = Toy operating system. No, another admin user won’t cut it. Yes it’s a pain in the arse. 

For an imaginary wibblesplat.com:

1. Open Terminal.
1. Run `sudo /Applications/Utilities/Keychain\ Access/Contents/MacOS/Keychain\ Access`
1. Unlock the System keychain.
1. Locate certificate (Category -> Certificates) , Control + click => Export... 
1. Export to `/tmp/wibblesplat.p12`
1. Feed it a password for the p12 archive.  Do **NOT** forget this. You’ll need it.
1. Go back to the Server Admin panel, grab the expiring certificate, and hit the Gearwheel, then select Generate Certificate Signing Request.
1. Save that to a file. (`/tmp/wibblesplat.csr`)

Next, we need to split the PKCS12 archive, to get the old private key out.

```
tom.oconnor@cloud-white:~$ cd /tmp
tom.oconnor@cloud-white:/tmp$ openssl pkcs12 -in wibblesplat.p12 -nocerts -out wibblesplat.key
> Enter Import Password: *****
> MAC verified OK
> Enter PEM pass phrase: *****
> Verifying - Enter PEM pass phrase: *****
```

Strip the passphrase from the key (otherwise you have to enter it lots when you restart services.)

```
tom.oconnor@cloud-white:/tmp$ openssl rsa -in wibblesplat.key -out wibblesplat.unprotected.key
> Enter pass phrase for wibblesplat.key: *****
> writing RSA key
```
Export the old Certificate from the p12.  You might as well.

`tom.oconnor@cloud-white:/tmp$ openssl pkcs12 -in wibblesplat.p12 -clcerts -nokeys -out wibblesplat.old.crt`
Generate the Certificate from the CSR from earlier, and the freshly exported key.

```
tom.oconnor@cloud-white:/tmp$ openssl x509 -req -days 7300 -in wibblesplat.csr -signkey wibblesplat.unprotected.key -out wibblesplat.new.crt
> Signature ok
> subject=/CN=wibblesplat.com/C=GB
> Getting Private key
```

Now, you go back to Server Admin.  Re-select the expired certificate, and hit Gearwheel -> Replace with new signed certificate.

Find  the file "`wibblesplat.new.crt`" in Finder, and drag it into the Server Admin "Replace Screen"

You don’t need to replace the Key, because of the above steps, we used the old key.

Head back over to Keychain Access, 

Find the newly updated certificate, and you should find that the new expiry time is somewhere about 20 years from now (7300 days, which is the longest you can set a certificate Valid To date)

Then double click the new certificate, and under the Trust dropdown/treeview thingy, set 

`"When using this certificate, to Always Trust"`

Congrats.  You've just replaced an expired certificate with one that won't expire for 20 years (well, near enough.)