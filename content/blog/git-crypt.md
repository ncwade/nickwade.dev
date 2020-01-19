---
title: "Storing secrets safely in git"
date: 2020-01-19T14:56:00-05:00
draft: false
---

We were looking for a method that allowed us to store sensitive data alongside our infrastructure definitions in git. This is an overview on why we chose git-crypt.

## Goals
* Allow us to version control secrets.
* Minimize workflow impact.
* Infrastructure as code!

## Why git-crypt?
* Implemented using [git filters](https://www.juandebravo.com/2017/12/02/git-filter-smudge-and-clean/).
* Simple workflow.

## Downsides
* No real windows support. Partial support for MinGW compilation, generally buggy.
* Requires a semi-modern version of WSL(18.04 or later). Unless you want to compile by hand.
* Need to manage a certificate for Jenkins.
* Rotating keys/secrets is a pain.

## How to install
* MacOS
  ```sh
  brew install git-crypt
  ```
* Windows

  1. [Install Ubuntu 16.04 or later](https://docs.microsoft.com/en-us/windows/wsl/install-manual).
  2. Run these commands.
  ```sh
  sudo apt update
  sudo apt install git-crypt
  ```

## Example Workflow
```sh
# Create our test repository.
cd ~ && rm -rf test.repo && mkdir -p test.repo && cd test.repo && clear
git init
git-crypt init

# Add users to git-crypt.
gpg --list-keys
git-crypt add-gpg-user --trusted me@ncwade.com

# Create filter definitions.
echo "*.pem filter=git-crypt diff=git-crypt" > .gitattributes

# Create and add secret.
echo "dangerous secret regarding soft serve ice cream" > my-secret.pem
git add my-secret.pem
git-crypt status -e
git commit -m "Capture secret definition"

# Verifying the file is marked for encryption.
git-crypt lock
cat my-secret.pem
git-crypt unlock
cat my-secret.pem
```

### Changing the trusted users.
```sh
# Generate a new default key.
mkdir tmp.key && cd tmp.key
git init && git-crypt init
mv .git/git-crypt/keys/default ~/default.git-crypt.key
cd - && rm -rf tmp.key/
# Re-initializing the trusted users.
cd <your repo>
git-crypt unlock
mv ~/default.git-crypt.key .git/git-crypt/keys/default
rm -rf .git-crypt/keys/default/0/*.gpg
# Repeat this step for each user who needs access.
git-crypt add-gpg-user --trusted <your key or email>
git add .
git commit --amend
git push
```
