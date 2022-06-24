---
categories:
  - technology
  - gitlab
  - english
draft: false
image: imgs/quiet.jpg
pubdate: '2018-01-05'
recept:
  calculator:
    end: 0
    enkelvoud: ''
    meervoud: ''
    plusfactor: 0
    start: 0
  ingrediënten: []
  tijd:
    bereidingstijd: ''
    werktijd: ''
resources: []
subtitle: Store secrets encrypted in gitlab and decrypt them at deployment
title: GitLab CI & Blackbox
type: artikel

---

In this short tutorial I will show how to setup Blackbox for your GitLab project and create a gitlab-ci.yml recipe to decrypt secrets at deployment. You wil never have to worry about storing secret information in GitLab again.

You should have some experience with GitLab. I am a novice blackbox user myself, I recommend you read the Blackbox documentation first.

---

## Prepare your gitlab-project

First create a new project in Gitlab and clone it to you machine. I’ve called my project *blackbox_with_gitlab_demo*.

Now cd into your empty project and create to following files

./secret

```
secret
```

./test.rb

```ruby
content = File.read('secret')
if content.strip == 'secret'
  print "success\n"
else
  raise "Failure"
end
```

./.gitlab-ci.yml

```yaml
image: "ruby:2.4"simple_test:
  script:
  - ruby test.rb
```

Now commit and push …

```
git add .
git commit -m "initial import of project with gitlab-ci" -a
```

If everything works the output in project pipeline should be:

```
Running with gitlab-ci-multi-runner 9.5.0 (413da38)
  on xxx (e347b3ac)
Using Docker executor with image ruby:2.4 ...
Using docker image sha256:b967f6d2a7a1fbba2c3b476a9554635e95d45f207f1dd61a5bc7db34cecfbad7 for predefined container...
Pulling docker image ruby:2.4 ...
Using docker image ruby:2.4 ID=sha256:713da53688a6446953cb7c0260d03a65a115cb60b727a890142628a4622642c7 for build container...
Running on runner-e347b3ac-project-532-concurrent-0 via xxxx...
Cloning repository...
Cloning into '/builds/Sandbox/blackbox_with_gitlab_demo'...
Checking out e44b7d1f as master...
Skipping Git submodules setup
$ ruby test.rb
Success
Job succeeded
```

In this example I committed the secret unencrypted into the repository. This is just for later showing how blackbox works. You should not do this because Git never forgets. When you accidentally committed a secret without encryption in your repository replace the secret with new one. Just adding encryption afterwards is not a solution.

## Setup Blackbox
Now we’re going to setup blackbox for this project. If you not already have done this, install blackbox on your development machine. This is a quick and dirty tutorial. Please read the full documentation on the blackbox website.

We will do the following steps:

* initialize blackbox in our repo
* add yourself as admin user
* Create a new key for the deploy-user
* Add a password-less subkey for the deploy-user.
* add the deploy-user-key to the blackbox admins list
* encrypt the file ./secret

Run the blackbox_initialize command and commit.

```
blackbox_initialize

Enable blackbox for this git repo? (yes/no) yes

git commit -m'INITIALIZE BLACKBOX' keyrings .gitignore
```

Add myself add blackbox admin

```
blackbox_addadmin pim.the.backbox.admin@domain.com

git commit -m "add myself as admin" -a
```

Notice that email addresses are used as the identifier for a GPG-key. Make sure
that the are unique when you use more than one key.

Create a new key for the automated user.

```
gpg --gen-key

Real name: gitlab-deploy-user
Email address: gitlab-deploy-user@domain.com
```

And create a password less subkey for this key.

```
gpg --edit-key gitlab-deploy-user@domain.com
gpg> addkey
...   (6) RSA (encrypt only)
...    0 = key does not expire
Is this correct? (y/N) y
Really create? (y/N) y

# key is created now set empty password

gpg> key 2

# the second key gets a * which means it's selected
ssb* rsa2048/B799C28A639D8F3A

gpg> password

# first enter the main key password
# then enter the empty password
# on my Mac I had to do this 3 times!

gpg> save
```

Ready? Good job, we’re making progress! Now add this deployment key(s) to the blackbox admin list.

```
blackbox_addadmin gitlab-deploy-user@domain.com

git commit -m "add gitlab deploy user as admin" -a
```

Great now let’s encrypt ./secret.

```
blackbox_register_new_file secret
========== PLAINFILE secret
========== ENCRYPTED secret.gpg
========== Importing keychain: START

gpg: Total number processed: 7
gpg:              unchanged: 7
========== Importing keychain: DONE
========== Encrypting: secret
========== Encrypting: DONE
========== Adding file to list.
========== CREATED: secret.gpg
========== UPDATING REPO:
rm 'secret'
NOTE: "already tracked!" messages are safe to ignore.
[master 8f4b498] registered in blackbox: secret
 2 files changed, 1 insertion(+)
 create mode 100644 secret.gpg
========== UPDATING VCS: DONE
Local repo updated.  Please push when ready.

git push
```

./secret is now replaces with ./secret.gpg.

Commit these changes.

```
git commit -m "encrypt secret" -a
```

So it’s been a while lets push this series of commit’s.

```
git push
```

The GitLab pipeline will fail now because it cannot read secret.

```
Running with gitlab-ci-multi-runner 9.5.0 (413da38)
  on xxx (e347b3ac)
Using Docker executor with image ruby:2.4 ...
Using docker image sha256:b967f6d2a7a1fbba2c3b476a9554635e95d45f207f1dd61a5bc7db34cecfbad7 for predefined container...
Pulling docker image ruby:2.4 ...
Using docker image ruby:2.4 ID=sha256:713da53688a6446953cb7c0260d03a65a115cb60b727a890142628a4622642c7 for build container...
Running on runner-e347b3ac-project-532-concurrent-0 via xxx...
Fetching changes...
HEAD is now at e44b7d1 initial import of project with gitlab-ci
From https://xxx/Sandbox/blackbox_with_gitlab_demo
   e44b7d1..9d38513  master     -> origin/master
Checking out 9d385138 as master...
Skipping Git submodules setup
$ ruby test.rb
test.rb:1:in `read': No such file or directory @ rb_sysopen - secret (Errno::ENOENT)
 from test.rb:1:in `<main>'
ERROR: Job failed: exit code 1
```

Make Gitlab able to decrypt secret

In the last steps of this tutorial we will make GitLab able to decrypt all
blackbox’ed at deployment.

First we’re going to export the key of the deploy-user.

```
gpg -a — export-secret-keys gitlab-deploy-user@domain.com > /tmp/gitlabkey
```

Now copy the contents of /tmp/gitlabkey and paste in the GitLab project settings as Secret Variable. Name the variable GPG_PRIVATE_KEY

Afterward rm the exported key:

```
rm /tmp/gitlabkey
```

Now add the following code lines to .gitlab-ci.yml to install blackbox and let
it import the key at deployment.

```yaml
- apt-get update -qq && apt-get -yqq install gnupg2
  - git clone https://github.com/mipmip/blackbox
  - cd blackbox
  - make manual-install
  - cd ..
  - gpg2 -v --import <(echo "$GPG_PRIVATE_KEY")
  - GPG=gpg2 blackbox_postdeploy
```

In this tutorial I’m forced to install and use gpg2 because this Ubuntu
container does not support gpg2 out of the box. This may be unnecessary in your
situation.

The command blackbox_postdeploy decrypts all files for deployment

The complete .gitlab-ci.yml hould look like this:

```
image: "ruby:2.4"simple_test:
  script:
  - apt-get update -qq && apt-get -yqq install gnupg2
  - git clone https://github.com/StackExchange/blackbox.git
  - cd blackbox
  - make manual-install
  - cd ..
  - gpg2 -v --import <(echo "$GPG_PRIVATE_KEY")
  - GPG=gpg2 blackbox_postdeploy
  - ruby test.rb
```

Commit and push it.

```
git commit -m "enable blackbox for gitlab-ci" .gitlab-ci.yml
git push
```

So, if all went well the GitLab pipeline should succeed again and you have successfully implemented blackbox in your continuous integration workflow.

The pipeline output should look like this:

```
Running with gitlab-ci-multi-runner 9.5.0 (413da38)
  on xxx (e347b3ac)
Using Docker executor with image ruby:2.4 ...
Using docker image sha256:b967f6d2a7a1fbba2c3b476a9554635e95d45f207f1dd61a5bc7db34cecfbad7 for predefined container...
Pulling docker image ruby:2.4 ...
Using docker image ruby:2.4 ID=sha256:713da53688a6446953cb7c0260d03a65a115cb60b727a890142628a4622642c7 for build container...
Running on runner-e347b3ac-project-532-concurrent-0 via xxx...
Fetching changes...
Removing blackbox/
HEAD is now at b1b55e1 enable blackbox for gitlab-ci
From https://xxx/Sandbox/blackbox_with_gitlab_demo
   b1b55e1..5606980  master     -> origin/master
Checking out 56069805 as master...
Skipping Git submodules setup
$ apt-get update -qq && apt-get -yqq install gnupg2
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libassuan0:amd64.
(Reading database ...

TLTR

$ git clone https://github.com/StackExchange/blackbox.git
Cloning into 'blackbox'...
$ cd blackbox
$ make manual-install
Symlinking files from ./bin to /usr/local/bin
Done.
$ cd ..
$ gpg2 -v --import <(echo "$GPG_PRIVATE_KEY")
gpg: directory `/root/.gnupg' created
gpg: new configuration file `/root/.gnupg/gpg.conf' created
gpg: WARNING: options in `/root/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/root/.gnupg/secring.gpg' created
gpg: keyring `/root/.gnupg/pubring.gpg' created
Comment: GPGTools - http://gpgtools.org
gpg: armor header:
gitlab-deploy-user <gitlab-deploy-user@domain.com>gpg: sec  2048R/C57BF429 2018-01-05
gpg: key C57BF429: secret key imported
gpg: pub  2048R/C57BF429 2018-01-05  gitlab-deploy-user <gitlab-deploy-user@domain.com>
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: using PGP trust model
gpg: key C57BF429: public key "gitlab-deploy-user <gitlab-deploy-user@domain.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
$ GPG=gpg2 blackbox_postdeploy
========== Importing keychain: START
gpg: keyblock resource `/builds/Sandbox/blackbox_with_gitlab_demo/keyrings/live/pubring.kbx': Not supported
gpg: Note: This version of GPG does not support the Keybox format
gpg: key C57BF429: "gitlab-deploy-user <gitlab-deploy-user@domain.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
========== Importing keychain: DONE
========== Decrypting new/changed files: START
========== EXTRACTED secret
========== Decrypting new/changed files: DONE
$ ruby test.rb
Success
Job succeeded
```

I hope this gives you inspiration and let’s you never worry about commiting
secret stuff into GitLab again.
