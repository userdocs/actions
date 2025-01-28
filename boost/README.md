# A composite action to determine a valid boost download url

In this order of preference:

[boost github](https://github.com/boostorg/boost/releases/latest) > [archive.io](https://archives.boost.io/release/) > [sourceforge](https://sourceforge.net/projects/boost/files/) > [userdocs mirror](https://github.com/userdocs/boost/releases/latest)

It requires `git` `curl` `xz-utils` `coreutils` to function.

> [!WARNING]
> These dependencies should all be available on the main runners but if you use a container you need to make sure they are available to the action.

It is a POSIX shell script.

How to use:

```yaml
- name: Docker - Bootstrap the boost files
  id: boost_info
  uses: userdocs/actions/boost@main
  with:
    boost_version: ""
    download_archive: false
    extract_archive: false
    working_dir: ""
```

These are the outputs:

```yaml
${{ steps.boost_version_info.outputs.BOOST_WORKING_DIR }}
${{ steps.boost_version_info.outputs.BOOST_VERSION }}
${{ steps.boost_version_info.outputs.BOOST_MAJOR_VERSION }}
${{ steps.boost_version_info.outputs.BOOST_MINOR_VERSION }}
${{ steps.boost_version_info.outputs.BOOST_PATCH_VERSION }}

${{ steps.boost_version_info.outputs.BOOST_URL }}
${{ steps.boost_version_info.outputs.BOOST_ARCHIVE_NAME }}
${{ steps.boost_version_info.outputs.BOOST_FOLDER_NAME }}
${{ steps.boost_version_info.outputs.BOOST_ARCHIVE_SHA1SUM }}
${{ steps.boost_version_info.outputs.BOOST_ARCHIVE_SHA256SUM }}
```

Will output something like this for github links:

```shell
BOOST_VERSION: 1.87.0
BOOST_MAJOR_VERSION: 1
BOOST_MINOR_VERSION: 87
BOOST_PATCH_VERSION: 0
BOOST_URL: https://github.com/boostorg/boost/releases/latest/download/boost-1.87.0-b2-nodocs.tar.xz
BOOST_ARCHIVE_NAME: boost-1.87.0.tar.xz
BOOST_FOLDER_NAME: boost-1.87.0
Downloading https://github.com/boostorg/boost/releases/latest/download/boost-1.87.0-b2-nodocs.tar.xz to boost-1.87.0.tar.xz
BOOST_ARCHIVE_SHA1SUM=45aea302751c712c81064ba00140d391ca745037  boost-1.87.0.tar.xz
BOOST_ARCHIVE_SHA256SUM=3abd7a51118a5dd74673b25e0a3f0a4ab1752d8d618f4b8cea84a603aeecc680  boost-1.87.0.tar.xz
Extracting boost-1.87.0.tar.xz to boost-1.87.0
```

For non github links

```shell
BOOST_VERSION: 1.78.0
BOOST_MAJOR_VERSION: 1
BOOST_MINOR_VERSION: 78
BOOST_PATCH_VERSION: 0
BOOST_URL: https://archives.boost.io/release/1.78.0/source/boost_1_78_0.tar.gz
BOOST_ARCHIVE_NAME: boost-1.78.0.tar.gz
BOOST_FOLDER_NAME: boost_1_78_0
Downloading https://archives.boost.io/release/1.78.0/source/boost_1_78_0.tar.gz to boost-1.78.0.tar.gz
BOOST_ARCHIVE_SHA1SUM=ff717ea23af2900e0ad3cae616b3e4ae43e68ad7  boost-1.78.0.tar.gz
BOOST_ARCHIVE_SHA256SUM=94ced8b72956591c4775ae2207a9763d3600b30d9d7446562c552f0a14a63be7  boost-1.78.0.tar.gz
Extracting boost-1.78.0.tar.gz to boost_1_78_0
```

You can provide a version of boost to use, with `boost_version` otherwise it defaults to latest version.

```yaml
with:
  boost_version: "1.84.0" # default is latest boost version tag
```

You can get it to download the archive

```yaml
with:
  download_archive: true # default false
```

extract the archive. use `${{ steps.boost_info.outputs.BOOST_FOLDER_NAME }}` to know the name of this extracted folder.

```yaml
with:
  extract_archive: true # default false
```

```yaml
with:
  working_dir: "my/dir" # default to empty
```

## Demo workflow

```yaml
name: boost action tests

on:
  workflow_dispatch:

jobs:
  build-1:
    runs-on: ubuntu-24.04-arm
    permissions:
      contents: read
    strategy:
      fail-fast: false
      matrix:
        boost_version: ["", "1.78.0", "1.84.0"]
        download_archive: ["true", "false"]
        extract_archive: ["true", "false"]
        working_dirs: ["", "/tmp"]

    steps:
      - uses: userdocs/actions/boost@main
        id: boost_info
        with:
          boost_version: ${{ matrix.boost_version }}
          download_archive: ${{ matrix.download_archive }}
          extract_archive: ${{ matrix.extract_archive }}
          working_dir: ${{ matrix.working_dirs }}

      - run: |
          pwd
          ls -la

      - run: |
          echo ${BOOST_WORKING_DIR}
          echo ${BOOST_WORKING_DIR} >> $GITHUB_STEP_SUMMARY

          echo ${BOOST_VERSION}
          echo ${BOOST_VERSION} >> $GITHUB_STEP_SUMMARY

          echo ${BOOST_MAJOR_VERSION}
          echo ${BOOST_MAJOR_VERSION} >> $GITHUB_STEP_SUMMARY

          echo ${BOOST_MINOR_VERSION}
          echo ${BOOST_MINOR_VERSION} >> $GITHUB_STEP_SUMMARY

          echo ${BOOST_PATCH_VERSION}
          echo ${BOOST_PATCH_VERSION} >> $GITHUB_STEP_SUMMARY

          echo ${BOOST_URL}
          echo ${BOOST_URL} >> $GITHUB_STEP_SUMMARY

          echo ${BOOST_ARCHIVE_NAME}
          echo ${BOOST_ARCHIVE_NAME} >> $GITHUB_STEP_SUMMARY

          echo ${BOOST_FOLDER_NAME}
          echo ${BOOST_FOLDER_NAME} >> $GITHUB_STEP_SUMMARY

          echo ${BOOST_ARCHIVE_SHA1SUM}
          echo ${BOOST_ARCHIVE_SHA1SUM} >> $GITHUB_STEP_SUMMARY

          echo ${BOOST_ARCHIVE_SHA256SUM}
          echo ${BOOST_ARCHIVE_SHA256SUM} >> $GITHUB_STEP_SUMMARY
        env:
          BOOST_WORKING_DIR: ${{ steps.boost_version_info.outputs.BOOST_WORKING_DIR }}
          BOOST_VERSION: ${{ steps.boost_version_info.outputs.BOOST_VERSION }}
          BOOST_MAJOR_VERSION: ${{ steps.boost_version_info.outputs.BOOST_MAJOR_VERSION }}
          BOOST_MINOR_VERSION: ${{ steps.boost_version_info.outputs.BOOST_MINOR_VERSION }}
          BOOST_PATCH_VERSION: ${{ steps.boost_version_info.outputs.BOOST_PATCH_VERSION }}
          BOOST_URL: ${{ steps.boost_version_info.outputs.BOOST_URL }}
          BOOST_ARCHIVE_NAME: ${{ steps.boost_version_info.outputs.BOOST_ARCHIVE_NAME }}
          BOOST_FOLDER_NAME: ${{ steps.boost_version_info.outputs.BOOST_FOLDER_NAME }}
          BOOST_ARCHIVE_SHA1SUM: ${{ steps.boost_version_info.outputs.BOOST_ARCHIVE_SHA1SUM }}
          BOOST_ARCHIVE_SHA256SUM: ${{ steps.boost_version_info.outputs.BOOST_ARCHIVE_SHA256SUM }}
```
