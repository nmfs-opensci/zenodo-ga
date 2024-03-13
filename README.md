# Zenodo Action
This is a variant of [zenodo-release](https://github.com/rseng/zenodo-release) Copyright © 2022, Vanessa Sochat that is focused specifically on creating a Zenodo release when a GitHub release is made.

This is a GitHub Action to allow you to automatically update a package on zenodo
when a GitHub release is created, and without needing to enable admin webhooks. To get this working you will need to:

1. Create an account on Zenodo
2. Under your name -> Applications -> Developer Applications -> Personal Access Tokens -> +New Token and choose both scopes for deposit
3. Add the token to your repository secrets as `ZENODO_TOKEN`
4. Create a .zenodo.json file for the root of your repository (see [template](.zenodo.json))
5. Add the example action (modified for your release) to your GitHub repository in the .github/workflows directory.

**Important** You CANNOT create a release online first and then try to upload to the same DOI.
If you do this, you'll get:

```python
{'status': 400,
 'message': 'Validation error.',
 'errors': [{'field': 'metadata.doi',
   'message': 'The prefix 10.5281 is administrated locally.'}]}
```

I think this is kind of silly, but that's just me. So the way to go is likely to:

1. Create a new DOI on the first go when you don't have one
2. Add the "all releases" DOI to your action to update that one for all releases after that!

## Usage


### GitHub Action

After you complete the steps above to create the metadata file, you have two options.

#### Existing DOI

If you have an existing DOI that is of the **all versions** type meaning we can update it, you should provide it to the action.
The example below shows running a release workflow and providing an archive to update to a new version (**released under the same DOI**)

```yaml
name: Zenodo Release

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-20.04

    steps:
    - uses: actions/checkout@v3
    - name: download archive to runner
      env:
        zipball: ${{ github.event.release.zipball_url }}
      run: |
        name=$(basename ${zipball})        
        curl -L $zipball > $name
        echo "archive=${name}" >> $GITHUB_ENV

    - name: Run Zenodo Deploy
      uses: nmfs-opensci/zenodo-release@main
      with:
        token: ${{ secrets.ZENODO_TOKEN }}
        version: ${{ github.event.release.tag_name }}
        zenodo_json: .zenodo.json   # optional if adding to an existing record in Zenodo
        archive: ${{ env.archive }}

        # Optional DOI for all versions. Leaving this blank (the default) will create
        # a new DOI on every release. Use a DOI that represents all versions will
        # create a new version for this existing DOI.
        doi: '10.5281/zenodo.6326822'
```


