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
Example usage in a Dockerfile : retrieving and installing the latest version of UnRAR.
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

# You MUST install required libraries or else you'll run into linked libraries loading issues
RUN apk add --no-cache libstdc++ libgcc
```

## Dependencies ?
Since Alpine Linux uses `musl` instead of `glibc`, things are a bit different.  
The basic thing you could do is `apk add --no-cache gcc`, or if you're really tryhard on your container size, install only the required dependencies with `apk add --no-cache libstdc++ libgcc`.

> [!TIP]  
> If you don't trust me despite the numerous ways to verify the integrity of the binaries, you can always use the original binaries (on [this page](https://www.rarlab.com/download.htm), under the name "RAR for Linux x64"), with a twist :
> ```dockerfile
> FROM alpine:latest
>
> # ...
>
> RUN apk update && \
>     apk add --no-cache curl libc6-compat
>
> # ...
>
> RUN curl -LsSf https://www.rarlab.com/rar/rarlinux-x64-712.tar.gz > /tmp/rarlinux-x64-712.tar.gz && \
>     mkdir /tmp/unrar && \
>     tar xf /tmp/rarlinux-x64-712.tar.gz -C /tmp/unrar --strip-components=1 && \
>     install -v -m755 /tmp/unrar/unrar /usr/local/bin
>
> # You MUST install required libraries as well
> RUN apk add --no-cache libstdc++ libgcc
> ```
> Here, we are using the original binaries provided by RARLAB, which are built for `glibc`, and we're using a compatibility layer to run them on Alpine Linux.

## About the releases
All available versions along with their download URLs are available in [`versions.json`](versions.json).  
Each release contains the SHA-256 checksum of the built binary, as well as the SHA-256 checksum of the original files provided by RARLAB.  
You can also verify the release by going to [Attestations](https://github.com/EDM115/unrar-alpine/attestations), then select the correct release. Copy the provided Verify command and point it to the `unrar` binary from the release.  
Finally, the releases are made immutable, meaning nobody can edit the files after the release is made.

> [!IMPORTANT]  
> If a version is present in `versions.json` but **not** in the releases, it means that code simply didn't compiled.

> [!NOTE]  
> I initially wanted to provide ARM64 versions as well but it isn't possible due to current GitHub restrictions.

## Versioning scheme
The versions you see in the releases tab as tags are extracted from the original download link.  
*However*, the `unrar` binary will likely **not** return the same version.  
I do not know why RARLAB does this.  
For the sake of convenience, releases do not get the actual version from the `unrar` binary, as sometimes they go in the past and sometimes there are duplicates.  
The body of the release will precise :
- The version from the download link
- The version from the `unrar` binary
- The date at which it was released
- The checksum of the built `unrar`
- The checksum of the downloaded source code archive
- The size in bytes of the built `unrar`
- The date at which this release was built

This repo will check every day at 8 AM UTC if a new version is available, and if so, will build and release it.  
In the very unlikely event that RARLAB releases 2 UnRAR versions the same day, [open an issue](https://github.com/EDM115/unrar-alpine/issues) and I will manually add it.  

## License
Code written in this repo by myself is licensed under the MIT License.  
The original `unrar` code and its resulting binaries are freeware and property of RARLAB/WinRAR GmbH., please read the files in the [`unrar/`](./unrar/) folder for more info.  
Thanks to https://github.com/aawc/unrar for the idea of a repo releasing binaries.  
Thanks to Eugene Roshal for creating the RAR format and the UnRAR utility, as well as adding this very repo to the [list of user-contributed addons](https://www.rarlab.com/rar_add.htm).
