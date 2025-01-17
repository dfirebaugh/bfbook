
# Deploy

Here is a simple bash script that can deploy to gh-pages.

## Download

`./scripts/download_bbook.sh`
```bash
#!/bin/bash

mkdir -p bin

if [ ! -f bin/bbook ]
then
    echo "downloading banana-book"
    curl -sSL https://github.com/dfirebaugh/bbook/releases/download/v0.0.0/bbook-x86_64_unknown-linux.tar.gz | tar -xz --directory=bin
else
    echo "bin/bbook already exists"
fi
```

## Build

`./scripts/build_docs.sh`
```bash
#!/bin/bash

source ./scripts/download_bbook.sh

mkdir -p .dist/web/

cd docs/user_docs
../../bin/bbook build
cp -r .book/* ../../.dist/web/
cd ../../
```

## Deploy

`./scripts/deploy_docs.sh`
```bash
#!/bin/bash

source ./scripts/build_docs.sh

GIT_REPO_URL=$(git config --get remote.origin.url)

cd .book
git init .
git remote add github $GIT_REPO_URL
git checkout -b gh-pages
git add .
git commit -am "Static site deploy"
git push github gh-pages --force
cd ..
```


## Github Action

```bash
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '^1.19.x'

    - name: Run bbook
      run: |
        cd docs
        bbook build

    - name: Deploy
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./docs/.book
```
