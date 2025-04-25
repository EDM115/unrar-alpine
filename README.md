<div align="center">

# unrar-alpine
## UnRAR, built for Alpine Linux. Automatically built and released at each new version

</div>

## Description
WinRAR is a powerful archive manager. It can create, manage, and extract compressed files.  
It is paid, but only for creating RAR archives. Extracting such archives is possible thanks to the UnRAR utility, which is freeware.  
UnRAR is provided free of charge by RARLAB/WinRAR GmbH. They are kind enough to provide the source code for us to built it ourselves on platforms where they don't provide precompiled binaries.

## Usage
You will find as [releases](https://github.com/EDM115/unrar-alpine/releases) of this repo all publicly available versions of UnRAR, built for Alpine Linux.  
You can use them in your Dockerfile as follows :
```dockerfile
FROM alpine:latest

# ...

RUN apk update && \
    apk add --no-cache curl jq

# ...

RUN curl -LsSf https://api.github.com/repos/EDM115/unrar-alpine/releases/latest \
    | jq -r '.assets[] | select(.name == "unrar") | .id' \
    | xargs -I {} curl -LsSf https://api.github.com/repos/EDM115/unrar-alpine/releases/assets/{} \
    | jq -r '.browser_download_url' \
    | xargs -I {} curl -Lsf {} -o /tmp/unrar && \
    install -v -m755 /tmp/unrar /usr/local/bin
```

All available versions along with their download URLs are available in [`versions.json`](versions.json).  
Each release contains the SHA-256 checksum of the built binary, as well as the SHA-256 checksum of the original files provided by RARLAB.  
You can also verify the release by going to [Attestations](https://github.com/EDM115/unrar-alpine/attestations), then select the correct release. Copy the provided command and point it to the `unrar` binary from the zip.

> [!IMPORTANT]  
> If a version is present in `versions.json` but **not** in the releases, it means that code simply didn't compiled.

> [!NOTE]  
> I initially wanted to provide ARM64 versions as well but it isn't possible due to current GitHub restrictions.

## Versioning scheme
The versions you see in the releases tab as tags are extracted from the original download link.  
*However*, the `unrar` binary will likely **not** return the same version.  
I do not know why RARLAB does this.  
For the sake of convenience, releases do not get the actual version from the `unrar` binary, as sometimes they go in the past and sometimes they are duplicates.  
The body of the release will precise :
- The version from the download link
- The version from the `unrar` binary
- The date at which it was released
- The checksum of the built `unrar`
- The checksum of the downloaded source code archive

This repo will check every day at 8 AM UTC if a new version is available, and if so, will build and release it.  
In the very unlikely event that RARLAB releases 2 unrar versions the same day, [open an issue](https://github.com/EDM115/unrar-alpine/issues) and I will manually add it.  

## License
Code written in this repo by myself is licensed under the MIT License.  
The original `unrar` code and its resulting binaries are freeware and property of RARLAB/WinRAR GmbH., please read the files in the [`unrar/`](./unrar/) folder for more info.  
Thanks to https://github.com/aawc/unrar for the idea of a repo releasing binaries.
