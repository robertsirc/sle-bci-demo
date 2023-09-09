# sle-bci-demo

## Introduction

In this we are going to build a small .NET application using dev containers with SLE BCI.

## Prereq

- [Rancher Desktop](https://rancherdesktop.io/) installed
- [VS Code](https://code.visualstudio.com/) installed
- [Git](https://git-scm.com/) installed (only if you are going to clone this repo)

## Create Project Folder

Create a directory call it `sle-bci-demo` or whatever you want really this will be the overall directory we are going to put our app in:

``` bash
mkdir sle-bci-demo
```

Now we are going to change our current directory to the one we just created:

``` bash
cd sle-bci-demo
```

Open this directory with `VS Code`.

## Dev Container File

Within `VS Code` we need to create a directory called `.devcontainer` and within that directory a file called `devcontainer.json`.

In this file we need to add the base brackets for json:

``` json
{

}
```

Few fields we are going to need to add to this json file:

``` json
{
    "name": "",
    "image": "",
    "forwardPorts": [  ],
    "customizations": { }
}
```

The `devcontainer.json` supports more fields but we need these for this project:

- `name` - name of the devcontainer.
- `image` - location of the container image.
- `forwardPorts` - ports that need to beforwarded on from the container.
- `customizations` - VS Code customization for the devcontainer.

And these are the following values:

``` json
{
    "name": "c#-dotnet-sdk-7.0",
    "image": "registry.suse.com/bci/dotnet-sdk:7.0",
    "forwardPorts": [ 3000 ],
    "customizations": {
        "vscode": {
          "settings": {},
          "extensions": ["ms-dotnettools.csharp", "ms-dotnettools.csdevkit", "ms-dotnettools.vscode-dotnet-runtime"]
        }
      }
}
```

## Connecting to the devcontainer

You will need to make sure that [Rancher Desktop](https://rancherdesktop.io/) is up and running. The container run time needs to be `Docker (Moby)`.

In the `VS Code` UI there is a button on the bottom left corner that will let you connect to a new devcontainer.

Once connected in to the devcontainer the terminal prompts will turn red indicating you are working within it. 

## Working within the devcontainer

While in the devcontainer we can work with in the base directory we created earlier, for this demo that was called `sle-bci-demo`. We are going to create an ASP.NET webapp:

``` bash
dotnet new webapp -n hello-world
```

We can work with in the devcontainer making code changes. But before we close out we are going to want to do a publish prior to closing the devcontainer:

``` bash
dotnet publish -c Release -o out
```

## The Dockerfile

Within out `hello-world` project file we are going toneed to create a file called `Dockerfile`. We are also going to create a file called  `.dockerignore`.

Within the `.dockerignore` we are going to want to add the following:

``` dockerfile
**/bin/
**/obj/
```

For the `Dockerfile` it should look something like this:

``` dockerfile
FROM registry.suse.com/bci/dotnet-aspnet:7.0
WORKDIR /publish
COPY /publish .
EXPOSE 80
ENTRYPOINT [ "dotnet", "hello-world.dll" ]
```

One of the big things you should notice is the fact we are using the __ASP.NET__ runtime and not the sdk as we did in the devcontainer.

Now we can build our container