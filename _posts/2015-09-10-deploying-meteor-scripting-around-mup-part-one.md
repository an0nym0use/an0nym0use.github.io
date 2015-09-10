---
layout: post
title: "Deploying Meteor: Scripting around mup - part one"
permalink: deploying-meteor-scripting-around-mup-part-one
---

Hopefully you've come across [Meteor Up][mup] and one of the [bajillion][sacha-youtube] [awesome][meteortips] [resources][gentlenode] about getting started with the tool. It's much needed and makes life pretty easy in deployment-land. In the ideal situation deployment would happen through a CI system, but we'll work up to that.

Here are a few possible issues that can crop up when deploying with standard mup:

* Not including private packages that aren't available from the `packages` directory
* Settings and `mup.json` files may be encrypted in your repo, and not available unencrypted
* Deploying from a different ref than the appropriate one
* Deploying dependent packages from a different ref than the appropriate one
* Accidentally including uncommitted changes in the deployment (guilty)

Unfortunately, mup doesn't check for any of these potential issues during operation. Arguably these are all problems with process, and also arguably it's not within mup's scope. No process is perfect however, and in small, highly agile teams sometimes you need to jump between different concerns in the same codebase.

Let's start scripting around mup to correct our flawed processes (and process management), and make a better effort to not mess up production.

## The simplest thing - solving our private package problem

Some people symlink private, separate packages into the `packages/` directory, however I'm not a fan of this method. Instead, I prefer to symlink them to a single directory, and use the `PACKAGE_DIRS` environment variable to give meteor access to the contents. Here is how it looks:

```bash
  ➜  meteor-local-packages  l
  total 24
  drwxr-xr-x   5 andy  staff   170B 27 Aug 13:33 .
  drwxr-xr-x+ 84 andy  staff   2.8K 10 Sep 19:17 ..
  lrwxr-xr-x   1 andy  staff    43B 20 Aug 09:59 astronomy-status-behavior -> /Users/andy/repos/astronomy-status-behavior
  lrwxr-xr-x   1 andy  staff    27B 20 Aug 11:36 schema -> /Users/andy/repos/schema
```

And we can use it like so:

```bash
PACKAGE_DIRS=~/meteor-local-packages mup deploy
```

How about we chuck that into a script.

```bash
#!/bin/bash

PACKAGE_DIRS=~/meteor-local-packages mup deploy
```

Woot! Job done.

## Adding settings files

Next stage - let's dynamically add our `settings.json` and `mup.json` files.

For me, these files are [GPG][gpg] encrypted with the keys for all the members of the development team, and committed to the repo. Let's surround our deployment script with some decryption and cleanup code.

```bash

gpg -d settings.json.gpg > settings.json
gpg -d mup.json.gpg > mup.json
PACKAGE_DIRS=~/meteor-local-packages mup deploy
rm settings.json
rm mup.json

```

Great! Now we can keep the things we need secured secure, and let the security happen invisibly.

Hang on, but what if while we're developing we're running a local meteor server (and hence requiring the settings.json file)?

Let's move it out of the way.

```bash

function decrypt_settings {
  files=(mup.json settings.json)
  for file in ${files[@]}
  do
    if [ -e "$file" ]
    then
      mv $file "$file".bak
    fi
    gpg -d "$file".gpg > $file
  done
}

function cleanup {
  files=(mup.json settings.json)
  for file in ${files[@]}
  do
    rm $file
    if [ -e "${file}.bak" ]
    then
      mv "$file".bak $file
    fi
  done
}

decrypt_settings
PACKAGE_DIRS=~/meteor-local-packages mup deploy
cleanup

```

Excellent, now my local meteor server config won't be affected when I deploy. We should be putting our settings files into different environment directories, but this will do for now.

In the next part we'll dive into the status of each repo to make sure we're bundling the correct version/s of the code.

Happy deploying!

[mup]: https://github.com/arunoda/meteor-up
[sacha-youtube]: https://www.youtube.com/watch?v=WLGdXtZMmiI
[meteortips]: http://meteortips.com/deployment-tutorial/digitalocean-part-1/
[gentlenode]: https://gentlenode.com/journal/meteor-19-deploying-your-applications-in-a-snap-with-meteor-up-mup/41
[gpg]: https://www.gnupg.org/
