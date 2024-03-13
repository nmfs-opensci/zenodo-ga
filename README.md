# Zenodo Action
This is a variant of [zenodo-release](https://github.com/rseng/zenodo-release) Copyright © 2022, Vanessa Sochat. This variant is focused specifically on creating a Zenodo release when a GitHub release is made.

This is a GitHub Action to allow you to automatically update a package on Zenodo
when a GitHub release is created, and without needing to enable admin webhooks. To get this working you will need to:

1. Create an account on Zenodo
2. Under your name -> Applications -> Developer Applications -> Personal Access Tokens -> +New Token and choose both scopes for deposit
3. Add the token to your repository secrets as `ZENODO_TOKEN`. Settings > Secrets > Actions > Repository Secret
4. Create a .zenodo.json file (or CITATION.cff) for the root of your repository (see [template](.zenodo.json))
5. Add the example action to your GitHub repository in the .github/workflows directory before your first release. Alternatively, create a release manually on Zenodo.

## New Release (no DOI)

You have no entry on Zenodo yet. Use this workflow and then make a release.

```
name: Zenodo Release

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-20.04

    steps:
    - uses: actions/checkout@v3
    - name: Run Zenodo Deploy
      uses: nmfs-opensci/zenodo-ga@main
      with:
        token: ${{ secrets.ZENODO_TOKEN }}
        zenodo_json: .zenodo.json   # either a .zenodo.json file or CITATION.cff
```

## Existing Release (has DOI on Zenodo)

Add the `doi: ` line and make sure it is the parent DOI or the "cite all versions by using the DOI". Now when you make releases, they will be added to the Zenodo record.

```
name: Zenodo Release

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-20.04

    steps:
    - uses: actions/checkout@v3
    - name: Run Zenodo Deploy
      uses: nmfs-opensci/zenodo-ga@main
      with:
        token: ${{ secrets.ZENODO_TOKEN }}
        zenodo_json: .zenodo.json   # either a .zenodo.json file or CITATION.cff
        doi: 10.5281/zenodo.10811321 # update with your DOI
```

## Customizing your Zenodo entry infomation

This is taken from the `.zenodo.json` or `CITATION.cff` file. You need to fill these in with the author infomation. `CITATION.cff` is the better option because it can be used more widely, but these files are finicky so definitely set up your CITATION file using a cff validator. They are available in Python and R. Do a Google search. Note the the cff version (in the cff file) is important for the validator as versions 1.1, 1.2 and 1.3 have different schema.

