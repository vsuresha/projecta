name: CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Login to JFrog Artifactory
      env:
        JFROG_USERNAME: ${{ secrets.JFROG_USERNAME }}
        JFROG_PASSWORD: ${{ secrets.JFROG_PASSWORD }}
      run: |
        echo $JFROG_PASSWORD | docker login <artifactory_url> -u $JFROG_USERNAME --password-stdin

    - name: Trigger Polaris Scan
      uses: pru-gtpt/devsecops-shared-workflows/actions/polaris-scan@main
      with:
        POLARIS_PROJECT_NAME: "‹POLARIS_PROJECT_NAME›"
        GIT_BRANCH_NAME: ${{ github.ref_name }} # OPTIONAL
        POLARIS_ACCESS_TOKEN: ${{ secrets.POLARIS_API_TOKEN }}
        USERNAME_PRUREGISTRY: ${{ secrets.USERNAME_PRUREGISTRY }}
        PASSWORD_PRUREGISTRY: ${{ secrets.PASSWORD_PRUREGISTRY }}

    - name: Download Polaris zip from Artifactory
      run: |
        curl -u "${{ secrets.JFROG_USERNAME }}:${{ secrets.JFROG_PASSWORD }}" \
          -o polaris.zip \
          https://antifactory-new.pru.intranet.asia:8443/antifactory/generic-nits-devsecops/polaris.zip

    - name: Extract Polaris zip
      run: |
        unzip polaris.zip -d polaris
      shell: bash

    - name: Run custom script
      run: /home/runner/kBs/index.js
      shell: bash

    - name: Create directory SHONE/bin
      run: mkdir -p SHONE/bin

    - name: Run Polaris CLI
      run: |
        cd polaris
        ./polaris
      shell: bash
