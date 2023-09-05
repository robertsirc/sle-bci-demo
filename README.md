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

