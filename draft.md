# This is a work in progress and content is likely to change.


# Introduction
This repository reflects my personal perspective and design choices for building `Containerfiles` and the resulting `image` usage. I am open to community input and suggestions for changes or improvements.

I will not repeat anything that has already been covered in [`man Containerfile`](https://github.com/containers/common/blob/main/docs/Containerfile.5.md), so please also consult it in conjuction with this guide when crafting your own `Containerfile`.

## FROM
1. Fully qualified registry name
    * Using a shortname works if an accompanying `shortname.conf` is configured on your system, but you can not guarantee that every system that uses the `Containerfile` will have the same configuration. Verbosity is necessary here.
2. A valid tag/digest
    * Using the `latest` tag, which is implied in the absence of anything, is not advisable. The `latest` may have breaking changes that you are not prepared for, therefore always include an accompanying tag/digest.
3. Use digests instead of tags
    * I recommend only using the digest to ensure that the referenced image is identical each and every day that you deploy your `Containerfile`. I like to think of an image as a snapshot in time when everything worked. This allows isolating issues more easily as well as having an accurate [SBOM](https://google.com).
4. Use the `minimal/micro/etc` variant of an image.
    * This recommendation is twofold. On one hand it reduces the overall image size, saving space. On the other, it also reduces the footprint of things to check when performing upgrades; fewer items means less work.

Examples:
```docker
FROM my-image #violates 1,2,3

FROM my-image:latest #violates 1,3

FROM example.com/images/my-image #violates 2,3

FROM example.com/images/my-image:latest #violates 3

FROM example.com/images/my-image:universal-prod-ready #violates 3,4

FROM example.com/images/my-image@sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855 #excellent!
```

## MAINTAINER
Add a URL for that points to the repository.
```docker
MAINTAINER Name @ https://example.com/repos/my-image
```
## RUN
* List privilege possible when executing any command
    ```docker
    #=======#
    # Wrong #
    #=======#
    USER root #set active user to root
    RUN wget https://example.com/file.zip #downloads the file as the root user

    #=========#
    # Correct #
    #=========#
    USER 1000 #set active user to a non-privileged account
    RUN wget https://example.com/file.zip #downloads the file as a non-privileged account
    ```
* Use an empheremal `VOLUME` when performaning transactions with a system package manager:
    ```docker
    # apt
    RUN --mount=type=tmpfs,destination=/var/lib/apt/lists apt update
    or
    # dnf
    RUN --mount=type=tmpfs,destination=/var/cache/dnf dnf makecache
    ```
    * `pip`
        ```docker
        RUN pip install --no-cache-dir pip
        ```

## CMD
## LABEL
## EXPOSE
## ENV
## ADD
## ENTRYPOINT
## VOLUME
## USER
## WORKDIR
## ARG


### References:
* https://github.com/containers/common/blob/main/docs/Containerfile.5.md
* https://docs.docker.com/build/building/best-practices/
