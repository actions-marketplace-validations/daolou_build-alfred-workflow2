# Build an Alfred workflow2
> forked by mperezi/build-alfred-workflow
Assemble the files within a source directory into an [Alfred workflow](https://www.alfredapp.com/workflows/) single package ready to be distributed.

The name of the output file is generated from the workflow name and version contained in the workflow's metadata.

## Usage

### Pre-requisites

This action needs a file named `info.plist` with the metadata of your workflow in the root of your repo. This file is automatically generated for you when [exporting workflows](https://www.alfredapp.com/help/workflows/advanced/sharing-workflows/) via the Alfred GUI.

### Inputs

* `workflow_dir`: Directory containing the sources of the workflow (defaults to `workflow`)
* `exclude_patterns`: List of excluded files/directories

### Outputs

* `workflow_file`: The name of the created .alfredworkflow file
* store symbolic links as the link instead of the referenced file

### Example workflow

On every `push` to a tag matching the pattern `v*`, [create a release](https://github.com/actions/create-release/) and [upload the workflow file](https://github.com/actions/upload-release-asset/) as an asset:
```yaml
name: Create Alfred Workflow

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Build Alfred Workflow
    	id: alfred_builder
      uses: mperezi/build-alfred-workflow@v1
      with:
        workflow_dir: src
        exclude_patterns: '*.pyc *__pycache__/*'
    - name: Create Release
      id: create_release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ github.ref }}
        release_name: ${{ github.ref }}
        draft: false
        prerelease: false
    - name: Upload Alfred Workflow
      uses: actions/upload-release-asset@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.create_release.outputs.upload_url }}
        asset_path: ${{ steps.alfred_builder.outputs.workflow_file }}
        asset_name: ${{ steps.alfred_builder.outputs.workflow_file }}
        asset_content_type: application/zip
```

## Additional files

In addition to the files and folders contained in the `workflow_dir` some other files are also implicitily shipped within the target `workflow_file`. These files are searched for in the root directory and are silently ignored in case they don't exist:

* `info.plist` (*required*)
* `icon.png`
* `README`, `README.md`

* `LICENSE`
