# Opportunities for development teams considering native z/OS containers

The IBM z/OS Container Platform (zOSCP) makes native z/OS containers a reality but how do you make this most of this new capability? In this article we'll focus in on development teams and explore the various opportunities that containers brings for developers on z/OS.

### Self-service instant access to products and tools

Developers often face friction and delays that frustrate their goals. A request for the install of a new software product or an upgrade to a new release to assess new capabilities can often mean delays getting started on a new project or careful planning. 

With zOSCP, your products and tools are stored as container images in an image repository. Your enterprise very likely already has one. Container images don't need to be installed and no-one has to understand SMP/e. 

To use these products and tools, you can browse the available container images in your enterprise image repository and choose the version you want to use. For example, if you don't have bash or git installed on your LPAR but you have a development project that you'd like those tools, the latest [IBM Open Enterprise Foundation for z/OS](https://community.ibm.com/community/user/ibmz-and-linuxone/blogs/igor-todorovski/2024/08/21/oef-container-release?communityKey=e7b7d299-8509-4572-8cf1-c1112684644f) image has that covered. Simply run that image directly `podman run -it --rm <image-location>` and you'll find yourself looking at a bash prompt with tools like git, curl, perl and vim available. If the image wasn't already "pulled" (container lingo for copied) to the LPAR, `podman` will first copy then run the container in a single command.

```sh
$ podman run -it --rm <image-location>/oeftools:latest
bash-5.2$ git version
git version 2.45.2
```

You aren't limited to using simple shell tools either. There is a rapidly growing collection of container images from [IBM Open Enterprise SDK for Go](https://www.ibm.com/products/open-enterprise-sdk-go-zos) and [IBM Java](https://www.ibm.com/docs/en/zoscp/1.1.0?topic=uzcp-tutorial-building-running-helloworld-java-application-that-uses-supplied-java-image) to the latest [IBM z/OS Connect (OpenAPI 3)](https://www.ibm.com/docs/en/zos-connect/zos-connect/3.0?topic=zos-connect-server-image). You also aren't limited to IBM provided images and can even provide your own images. More on that shortly!

### Fearless experimentation

With your container up and running, you may start to wonder what else you can do. If you aren't familiar with containers already, a container is isolated from the host system and is unable to see or access the host. Things like processes, file systems and networking are visible inside the container but the container can only see processes running inside the container, the container network and a rather empty looking filesystem containing only what the container image provided. 

As each container is completely isolated from each other and from the host, you can feel free to create, modify or delete anything inside the container Unix filesystem without fear that you will damage the host LPAR. 

Once you know you aren't going to break anything, you'll be able to use containers to experiment without fear. Create your own files and directories wherever you need. It's not uncommon for containerised products and tools to create directories they need directly off the root file system for convenience, e.g. `/logs`, `/output`.

### Customise products and tools with ease for your teams needs

While having quick access to container images without having to install first, significantly helps shorten the gap to accessing the products and tools development teams need, there is always going to be a desire to customise these images to provide additional tooling or custom configuration. Don't worry though because containers have a really easy answer for this in the form of Containerfiles (also sometimes called Dockerfiles).

Let's pick on IBM z/OS Connect for a moment. For zOSCP, the z/OS Connect product provides a containerised server image. The server image however, requires the z/OS Connect API project is built into a WAR file and provided to the server via the `/config/dropins` directory. 

To provide the WAR file, the z/OS Connect Designer builds the WAR file as well as a small Containerfile that you can build with a `podman build -f Containerfile -t <new-image-name> .` command.

Example Containerfile:
```
FROM <internal-registry-location>/ibm-zcon-server:3.0.85

# customise server.xml via dropins
COPY --chmod=755 src/main/liberty/config /config/configDropins/overrides

# install war
COPY --chmod=755 build/libs /config/dropins
```

This Containerfile contains simply instructions to build a new customised container image that is identical to the z/OS Connect server image except where the instructions in the Containerfile have made changes.

The Containerfile is fairly simple (most are):
- The `FROM` instruction on the first line, tells `podman` to use the z/OS Connect server image as the base image for all further instructions.
- The first `COPY` instruction, copies any Liberty configuration you have into the container Liberty configuration directory. Note that a `COPY` instruction copies files from the build filesystem into the container. The `--chmod=755` sets the permissions for all directories and files copied.
- The second `COPY` instruction copies the WAR file into the `dropins` directory.

When the `podman build` command completes a new container image should be visible in the `podman images` list. When a container is instantiated from this new container image, it will be fully configured with all of the specified changes. Containerfiles support a few other instructions as well for running commands inside the container and setting environment variables in the container. 

### Share your creations with others

### Always be ready to scale up

### Configuration as Code

### Skills

### Other considerations

- Data sets and SAF security is not namespaced among others
- For images with extended attributes, an image administrator may need to pull the image onto the LPAR