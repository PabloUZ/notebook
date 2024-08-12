[Back](../Deploy.md)

# Deploying with Git

This deploy allows you to push the code changes into the serve instead a cloud repository as github. In this way, you can finish a feature and upload your changes into your server.

### 1. Set up Apache as a proxy
[Link](#)

### 2. Set permissions to users and groups
Before creating the repository, we should create a structure inside the main folder

```bash
📂my-project
├─ 📂 git
└─ 📂 www
```
`git` directory is the one in charge of version controlling, such as the known `.git`.

`www` is the directory in which we are going to save the code.

Create the user in charge of deploying and maintenance of the server.

Add the permissions to access to the directory previously created.

### 3. Install Git
```bash
sudo apt install git
```

### 4. Create the repository in the server
Then we have to init git so it will be able to receive files when pushing from the development environment.

To do that, we have to use the command `git init` with the `--bare` option, and specify the path to save changes (In this case is the directory `git`).

```bash
git init --bare <git path> --share=group
```

### 5. Create the script to manage push
We have to create a script to manage the code when it is received.

> `git/hooks/post-receive`
```bash
#!/bin/bash
BASE_DIR="<Full path of the root directory>"
TARGET_DIR="$BASE_DIR/www"
GIT_DIR="$BASE_DIR/git"
BRANCH="<Branch to track>"

while read oldrev newrev ref
do
    if [ "$ref" = "refs/heads/$BRANCH" ];
    then
        echo "Ref $ref received. Deploying ${BRANCH} branch to production..."
        git --work-tree=$TARGET_DIR --git-dir=$GIT_DIR checkout -f $BRANCH
    else
        echo "Ref $ref received. Doing nothing: only the ${BRANCH} branch may be deployed on this server."
    fi
done
```

After creating the script, we have to give it execution permissions.


### 6. Add production remote
Now we can add the production remote in the development environment.

```bash
git remote add production <USER>@<SERVER>:<PATH>
```

|VARIABLE|DESCRIPTION|
|-|-|
|USER|The server maintainer|
|SERVER|The IP or Domain Name of the server|
|PATH|The repository path (same as `GIT_DIR`)|