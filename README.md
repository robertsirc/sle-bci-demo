# sle-bci-demo

## Introduction

In this we are going to build a small .NET application using dev containers with SLE BCI. Then we are going to pull the same image from two different sources then compare them.

## Prereq

- [Rancher Desktop](https://rancherdesktop.io/) installed
- [VS Code](https://code.visualstudio.com/) installed
- [Git](https://git-scm.com/) installed (only if you are going to clone this repo)
- [trivy](https://github.com/aquasecurity/trivy) installed

## Developing with SLE BCI

### Create Project Folder

Create a directory call it `sle-bci-demo` or whatever you want really this will be the overall directory we are going to put our app in:

``` bash
mkdir sle-bci-demo
```

Now we are going to change our current directory to the one we just created:

``` bash
cd sle-bci-demo
```

Open this directory with `VS Code`.

### Dev Container File

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

### Connecting to the devcontainer

You will need to make sure that [Rancher Desktop](https://rancherdesktop.io/) is up and running. The container run time needs to be `Docker (Moby)`.

In the `VS Code` UI there is a button on the bottom left corner that will let you connect to a new devcontainer.

Once connected in to the devcontainer the terminal prompts will turn red indicating you are working within it.

### Working within the devcontainer

While in the devcontainer we can work with in the base directory we created earlier, for this demo that was called `sle-bci-demo`. We are going to create an ASP.NET webapp:

``` bash
dotnet new webapp -n hello-world
```

We can work with in the devcontainer making code changes. But before we close out we are going to want to do a publish prior to closing the devcontainer:

``` bash
dotnet publish -c Release -o out
```

### The Dockerfile

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

Now we can build our container.

## Comparing Images from Two Sources

### Pull image from registry.suse.com

With this command we are going to pull a python image (note the version might be different):

``` bash
docker pull registry.suse.com/bci/python:3.11
```

You will have an output similar to this:

``` bash
3.11: Pulling from bci/python
862c563194ac: Pull complete
e4dff7394e3c: Pull complete
Digest: sha256:13f6918b937e83620b9da568682c77534d05dfd1d192596526b9935fc9f41f6a
Status: Downloaded newer image for registry.suse.com/bci/python:3.11
registry.suse.com/bci/python:3.11
```

### Pull image from docker hub

``` bash
docker pull python:3.11
```

You wil have an output similar to this:

``` bash
3.11: Pulling from library/python
8cd46d290033: Pull complete
2e6afa3f266c: Pull complete
2e66a70da0be: Pull complete
1c8ff076d818: Pull complete
a0ff605e08b8: Pull complete
a040235ac426: Pull complete
bf8c0a4030de: Pull complete
608bb063c2b6: Pull complete
Digest: sha256:5977e1c5c86bfa34df0265327d0ed02514a8afbe9d96555c1f7679da9a370518
Status: Downloaded newer image for python:3.11
docker.io/library/python:3.11
```

### Scan Image from the SUSE Registry

Run the following command:

``` bash
trivy image --severity HIGH,CRITICAL registry.suse.com/bci/python
```

You should see results like this:

``` bash
2024-09-05T14:52:20.115-0400	INFO	Vulnerability scanning is enabled
2024-09-05T14:52:20.115-0400	INFO	Secret scanning is enabled
2024-09-05T14:52:20.115-0400	INFO	If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-09-05T14:52:20.115-0400	INFO	Please see also https://aquasecurity.github.io/trivy/v0.50/docs/scanner/secret/#recommendation for faster secret detection
2024-09-05T14:52:32.796-0400	INFO	License acquired from METADATA classifiers may be subject to additional terms for [argcomplete:3.2.1]
2024-09-05T14:52:32.797-0400	INFO	License acquired from METADATA classifiers may be subject to additional terms for [click:8.1.7]
2024-09-05T14:52:32.797-0400	INFO	License acquired from METADATA classifiers may be subject to additional terms for [pip:23.2.1]
2024-09-05T14:52:32.980-0400	INFO	Detected OS: suse linux enterprise server
2024-09-05T14:52:32.980-0400	WARN	This OS version is not on the EOL list: suse linux enterprise server 15.6
2024-09-05T14:52:32.981-0400	INFO	Detecting SUSE vulnerabilities...
2024-09-05T14:52:32.995-0400	INFO	Number of language-specific files: 0

registry.suse.com/bci/python (suse linux enterprise server 15.6)

Total: 0 (HIGH: 0, CRITICAL: 0)
```

### Scan Image from Docker Hub

Run the following command:

``` bash
trivy image --severity HIGH,CRITICAL python
```

You should see results like this:

``` bash
2024-09-05T14:45:20.747-0400	INFO	Vulnerability scanning is enabled
2024-09-05T14:45:20.747-0400	INFO	Secret scanning is enabled
2024-09-05T14:45:20.747-0400	INFO	If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-09-05T14:45:20.747-0400	INFO	Please see also https://aquasecurity.github.io/trivy/v0.50/docs/scanner/secret/#recommendation for faster secret detection
2024-09-05T14:45:21.766-0400	INFO	Detected OS: debian
2024-09-05T14:45:21.767-0400	INFO	Detecting Debian vulnerabilities...
2024-09-05T14:45:21.936-0400	INFO	Number of language-specific files: 1
2024-09-05T14:45:21.936-0400	INFO	Detecting python-pkg vulnerabilities...

python (debian 12.7)

Total: 102 (HIGH: 88, CRITICAL: 14)

┌───────────────────────┬────────────────┬──────────┬──────────────┬─────────────────────────┬───────────────┬──────────────────────────────────────────────────────────────┐
│        Library        │ Vulnerability  │ Severity │    Status    │    Installed Version    │ Fixed Version │                            Title                             │
├───────────────────────┼────────────────┼──────────┼──────────────┼─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ git                   │ CVE-2024-32002 │ CRITICAL │ affected     │ 1:2.39.2-1.1            │               │ git: Recursive clones RCE                                    │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-32002                   │
│                       ├────────────────┼──────────┤              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-25652 │ HIGH     │              │                         │               │ git: by feeding specially crafted input to `git apply        │
│                       │                │          │              │                         │               │ --reject`, a path...                                         │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-25652                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-29007 │          │              │                         │               │ git: arbitrary configuration injection when renaming or      │
│                       │                │          │              │                         │               │ deleting a section from a...                                 │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-29007                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-32004 │          │              │                         │               │ git: RCE while cloning local repos                           │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-32004                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-32465 │          │              │                         │               │ git: additional local RCE                                    │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-32465                   │
├───────────────────────┼────────────────┼──────────┤              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│ git-man               │ CVE-2024-32002 │ CRITICAL │              │                         │               │ git: Recursive clones RCE                                    │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-32002                   │
│                       ├────────────────┼──────────┤              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-25652 │ HIGH     │              │                         │               │ git: by feeding specially crafted input to `git apply        │
│                       │                │          │              │                         │               │ --reject`, a path...                                         │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-25652                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-29007 │          │              │                         │               │ git: arbitrary configuration injection when renaming or      │
│                       │                │          │              │                         │               │ deleting a section from a...                                 │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-29007                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-32004 │          │              │                         │               │ git: RCE while cloning local repos                           │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-32004                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-32465 │          │              │                         │               │ git: additional local RCE                                    │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-32465                   │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ krb5-multidev         │ CVE-2024-26462 │          │              │ 1.20.1-2+deb12u2        │               │ krb5: Memory leak at /krb5/src/kdc/ndr.c                     │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-26462                   │
├───────────────────────┼────────────────┼──────────┤              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libaom3               │ CVE-2023-6879  │ CRITICAL │              │ 3.6.0-1+deb12u1         │               │ aom: heap-buffer-overflow on frame size change               │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-6879                    │
│                       ├────────────────┼──────────┤              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-39616 │ HIGH     │              │                         │               │ AOMedia v3.0.0 to v3.5.0 was discovered to contain an        │
│                       │                │          │              │                         │               │ invalid read mem...                                          │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-39616                   │
├───────────────────────┼────────────────┤          ├──────────────┼─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libbluetooth-dev      │ CVE-2023-44431 │          │ fix_deferred │ 5.66-1+deb12u2          │               │ bluez: AVRCP stack-based buffer overflow remote code         │
│                       │                │          │              │                         │               │ execution vulnerability                                      │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-44431                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-51596 │          │              │                         │               │ bluez: phone book access profile heap-based buffer overflow  │
│                       │                │          │              │                         │               │ remote code execution vulnerability...                       │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-51596                   │
├───────────────────────┼────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│ libbluetooth3         │ CVE-2023-44431 │          │              │                         │               │ bluez: AVRCP stack-based buffer overflow remote code         │
│                       │                │          │              │                         │               │ execution vulnerability                                      │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-44431                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-51596 │          │              │                         │               │ bluez: phone book access profile heap-based buffer overflow  │
│                       │                │          │              │                         │               │ remote code execution vulnerability...                       │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-51596                   │
├───────────────────────┼────────────────┼──────────┼──────────────┼─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libexpat1             │ CVE-2024-45490 │ CRITICAL │ affected     │ 2.5.0-1                 │               │ libexpat: Negative Length Parsing Vulnerability in libexpat  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-45490                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-45491 │          │              │                         │               │ libexpat: Integer Overflow or Wraparound                     │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-45491                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-45492 │          │              │                         │               │ libexpat: integer overflow                                   │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-45492                   │
│                       ├────────────────┼──────────┤              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52425 │ HIGH     │              │                         │               │ expat: parsing large tokens can trigger a denial of service  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52425                   │
├───────────────────────┼────────────────┼──────────┤              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│ libexpat1-dev         │ CVE-2024-45490 │ CRITICAL │              │                         │               │ libexpat: Negative Length Parsing Vulnerability in libexpat  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-45490                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-45491 │          │              │                         │               │ libexpat: Integer Overflow or Wraparound                     │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-45491                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-45492 │          │              │                         │               │ libexpat: integer overflow                                   │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-45492                   │
│                       ├────────────────┼──────────┤              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52425 │ HIGH     │              │                         │               │ expat: parsing large tokens can trigger a denial of service  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52425                   │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libgssapi-krb5-2      │ CVE-2024-26462 │          │              │ 1.20.1-2+deb12u2        │               │ krb5: Memory leak at /krb5/src/kdc/ndr.c                     │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-26462                   │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libgssrpc4            │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libharfbuzz0b         │ CVE-2023-25193 │          │              │ 6.0.0+dfsg-3            │               │ harfbuzz: allows attackers to trigger O(n^2) growth via      │
│                       │                │          │              │                         │               │ consecutive marks                                            │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-25193                   │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libheif1              │ CVE-2023-49460 │          │              │ 1.15.1-1                │               │ libheif v1.17.5 was discovered to contain a segmentation     │
│                       │                │          │              │                         │               │ violation via ...                                            │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-49460                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-49462 │          │              │                         │               │ libheif v1.17.5 was discovered to contain a segmentation     │
│                       │                │          │              │                         │               │ violation via ...                                            │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-49462                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-49463 │          │              │                         │               │ libheif v1.17.5 was discovered to contain a segmentation     │
│                       │                │          │              │                         │               │ violation via ...                                            │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-49463                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-49464 │          │              │                         │               │ libheif v1.17.5 was discovered to contain a segmentation     │
│                       │                │          │              │                         │               │ violation via ...                                            │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-49464                   │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libk5crypto3          │ CVE-2024-26462 │          │              │ 1.20.1-2+deb12u2        │               │ krb5: Memory leak at /krb5/src/kdc/ndr.c                     │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-26462                   │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libkadm5clnt-mit12    │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libkadm5srv-mit12     │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libkdb5-10            │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libkrb5-3             │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libkrb5-dev           │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libkrb5support0       │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libldap-2.5-0         │ CVE-2023-2953  │          │              │ 2.5.13+dfsg-5           │               │ openldap: null pointer dereference in ber_memalloc_x         │
│                       │                │          │              │                         │               │ function                                                     │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-2953                    │
├───────────────────────┼────────────────┼──────────┤              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libopenexr-3-1-30     │ CVE-2023-5841  │ CRITICAL │              │ 3.1.5-5                 │               │ OpenEXR: Heap Overflow in Scanline Deep Data Parsing         │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-5841                    │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libopenexr-dev        │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┼────────────────┼──────────┤              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libopenjp2-7          │ CVE-2021-3575  │ HIGH     │              │ 2.5.0-2                 │               │ openjpeg: heap-buffer-overflow in color.c may lead to DoS or │
│                       │                │          │              │                         │               │ arbitrary code execution...                                  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2021-3575                    │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libopenjp2-7-dev      │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libperl5.36           │ CVE-2023-31484 │          │              │ 5.36.0-7+deb12u1        │               │ perl: CPAN.pm does not verify TLS certificates when          │
│                       │                │          │              │                         │               │ downloading distributions over HTTPS...                      │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-31484                   │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libpython3.11-minimal │ CVE-2024-6232  │          │              │ 3.11.2-6+deb12u3        │               │ python: cpython: tarfile: ReDos via excessive backtracking   │
│                       │                │          │              │                         │               │ while parsing header values                                  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-6232                    │
│                       ├────────────────┤          ├──────────────┤                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-7592  │          │ fix_deferred │                         │               │ cpython: python: Uncontrolled CPU resource consumption when  │
│                       │                │          │              │                         │               │ in http.cookies module                                       │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-7592                    │
├───────────────────────┼────────────────┤          ├──────────────┤                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│ libpython3.11-stdlib  │ CVE-2024-6232  │          │ affected     │                         │               │ python: cpython: tarfile: ReDos via excessive backtracking   │
│                       │                │          │              │                         │               │ while parsing header values                                  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-6232                    │
│                       ├────────────────┤          ├──────────────┤                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-7592  │          │ fix_deferred │                         │               │ cpython: python: Uncontrolled CPU resource consumption when  │
│                       │                │          │              │                         │               │ in http.cookies module                                       │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-7592                    │
├───────────────────────┼────────────────┤          ├──────────────┼─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libsqlite3-0          │ CVE-2023-7104  │          │ affected     │ 3.40.1-2                │               │ sqlite: heap-buffer-overflow at sessionfuzz                  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-7104                    │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libsqlite3-dev        │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libtiff-dev           │ CVE-2023-52355 │          │              │ 4.5.0-6+deb12u1         │               │ libtiff: TIFFRasterScanlineSize64 produce too-big size and   │
│                       │                │          │              │                         │               │ could cause OOM                                              │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52355                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52356 │          │              │                         │               │ libtiff: Segment fault in libtiff in TIFFReadRGBATileExt()   │
│                       │                │          │              │                         │               │ leading to denial of...                                      │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52356                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-7006  │          │              │                         │               │ libtiff: NULL pointer dereference in tif_dirinfo.c           │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-7006                    │
├───────────────────────┼────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│ libtiff6              │ CVE-2023-52355 │          │              │                         │               │ libtiff: TIFFRasterScanlineSize64 produce too-big size and   │
│                       │                │          │              │                         │               │ could cause OOM                                              │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52355                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52356 │          │              │                         │               │ libtiff: Segment fault in libtiff in TIFFReadRGBATileExt()   │
│                       │                │          │              │                         │               │ leading to denial of...                                      │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52356                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-7006  │          │              │                         │               │ libtiff: NULL pointer dereference in tif_dirinfo.c           │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-7006                    │
├───────────────────────┼────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│ libtiffxx6            │ CVE-2023-52355 │          │              │                         │               │ libtiff: TIFFRasterScanlineSize64 produce too-big size and   │
│                       │                │          │              │                         │               │ could cause OOM                                              │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52355                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52356 │          │              │                         │               │ libtiff: Segment fault in libtiff in TIFFReadRGBATileExt()   │
│                       │                │          │              │                         │               │ leading to denial of...                                      │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52356                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-7006  │          │              │                         │               │ libtiff: NULL pointer dereference in tif_dirinfo.c           │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-7006                    │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libxml2               │ CVE-2024-25062 │          │              │ 2.9.14+dfsg-1.3~deb12u1 │               │ libxml2: use-after-free in XMLReader                         │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-25062                   │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ libxml2-dev           │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┼────────────────┤          ├──────────────┼─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ linux-libc-dev        │ CVE-2013-7445  │          │ will_not_fix │ 6.1.106-3               │               │ kernel: memory exhaustion via crafted Graphics Execution     │
│                       │                │          │              │                         │               │ Manager (GEM) objects                                        │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2013-7445                    │
│                       ├────────────────┤          ├──────────────┤                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2019-19449 │          │ fix_deferred │                         │               │ kernel: mounting a crafted f2fs filesystem image can lead to │
│                       │                │          │              │                         │               │ slab-out-of-bounds read...                                   │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2019-19449                   │
│                       ├────────────────┤          ├──────────────┤                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2019-19814 │          │ affected     │                         │               │ kernel: out-of-bounds write in __remove_dirty_segment in     │
│                       │                │          │              │                         │               │ fs/f2fs/segment.c                                            │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2019-19814                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2021-3847  │          │              │                         │               │ kernel: low-privileged user privileges escalation            │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2021-3847                    │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2021-3864  │          │              │                         │               │ kernel: descendant's dumpable setting with certain SUID      │
│                       │                │          │              │                         │               │ binaries                                                     │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2021-3864                    │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52452 │          │              │                         │               │ kernel: bpf: Fix accesses to uninit stack slots              │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52452                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52590 │          │              │                         │               │ kernel: ocfs2: Avoid touching renamed directory if parent    │
│                       │                │          │              │                         │               │ does not change                                              │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52590                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52591 │          │              │                         │               │ kernel: reiserfs: Avoid touching renamed directory if parent │
│                       │                │          │              │                         │               │ does not change                                              │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52591                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52596 │          │              │                         │               │ kernel: sysctl: Fix out of bounds access for empty sysctl    │
│                       │                │          │              │                         │               │ registers                                                    │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52596                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52751 │          │              │                         │               │ kernel: smb: client: fix use-after-free in                   │
│                       │                │          │              │                         │               │ smb2_query_info_compound()                                   │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52751                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2023-52827 │          │              │                         │               │ kernel: wifi: ath12k: fix possible out-of-bound read in      │
│                       │                │          │              │                         │               │ ath12k_htt_pull_ppdu_stats()                                 │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-52827                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-21803 │          │              │                         │               │ kernel: bluetooth: use-after-free vulnerability in           │
│                       │                │          │              │                         │               │ af_bluetooth.c                                               │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-21803                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-25742 │          │              │                         │               │ hw: amd: Instruction raise #VC exception at exit             │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-25742                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-25743 │          │              │                         │               │ hw: amd: Instruction raise #VC exception at exit             │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-25743                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-26669 │          │              │                         │               │ kernel: net/sched: flower: Fix chain template offload        │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-26669                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-26913 │          │              │                         │               │ kernel: drm/amd/display: Fix dcn35 8k30 Underflow/Corruption │
│                       │                │          │              │                         │               │ Issue                                                        │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-26913                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-26930 │          │              │                         │               │ kernel: scsi: qla2xxx: Fix double free of the ha-&gt;vp_map  │
│                       │                │          │              │                         │               │ pointer                                                      │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-26930                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-26952 │          │              │                         │               │ kernel: ksmbd: fix potencial out-of-bounds when buffer       │
│                       │                │          │              │                         │               │ offset is invalid                                            │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-26952                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-36013 │          │              │                         │               │ kernel: Bluetooth: L2CAP: Fix slab-use-after-free in         │
│                       │                │          │              │                         │               │ l2cap_connect()                                              │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-36013                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-38570 │          │              │                         │               │ kernel: gfs2: Fix potential glock use-after-free on unmount  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-38570                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-39479 │          │              │                         │               │ kernel: drm/i915/hwmon: Get rid of devm                      │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-39479                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-39508 │          │              │                         │               │ kernel: io_uring/io-wq: Use set_bit() and test_bit() at      │
│                       │                │          │              │                         │               │ worker->flags                                                │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-39508                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-41013 │          │              │                         │               │ kernel: xfs: don&#39;t walk off the end of a directory data  │
│                       │                │          │              │                         │               │ block...                                                     │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-41013                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-41061 │          │              │                         │               │ kernel: drm/amd/display: Fix array-index-out-of-bounds in    │
│                       │                │          │              │                         │               │ dml2/FCLKChangeSupport                                       │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-41061                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-41071 │          │              │                         │               │ kernel: wifi: mac80211: Avoid address calculations via out   │
│                       │                │          │              │                         │               │ of bounds array indexing...                                  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-41071                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-41096 │          │              │                         │               │ kernel: PCI/MSI: Fix UAF in msi_capability_init              │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-41096                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-42162 │          │              │                         │               │ kernel: gve: Account for stopped queues when reading NIC     │
│                       │                │          │              │                         │               │ stats                                                        │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-42162                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-42228 │          │              │                         │               │ kernel: drm/amdgpu: Using uninitialized value *size when     │
│                       │                │          │              │                         │               │ calling amdgpu_vce_cs_reloc                                  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-42228                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-42314 │          │              │                         │               │ kernel: btrfs: fix extent map use-after-free when adding     │
│                       │                │          │              │                         │               │ pages to compressed bio...                                   │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-42314                   │
│                       ├────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-44942 │          │              │                         │               │ kernel: f2fs: fix to do sanity check on F2FS_INLINE_DATA     │
│                       │                │          │              │                         │               │ flag in inode...                                             │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-44942                   │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ perl                  │ CVE-2023-31484 │          │              │ 5.36.0-7+deb12u1        │               │ perl: CPAN.pm does not verify TLS certificates when          │
│                       │                │          │              │                         │               │ downloading distributions over HTTPS...                      │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-31484                   │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ perl-base             │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ perl-modules-5.36     │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
├───────────────────────┼────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ python3.11            │ CVE-2024-6232  │          │              │ 3.11.2-6+deb12u3        │               │ python: cpython: tarfile: ReDos via excessive backtracking   │
│                       │                │          │              │                         │               │ while parsing header values                                  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-6232                    │
│                       ├────────────────┤          ├──────────────┤                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-7592  │          │ fix_deferred │                         │               │ cpython: python: Uncontrolled CPU resource consumption when  │
│                       │                │          │              │                         │               │ in http.cookies module                                       │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-7592                    │
├───────────────────────┼────────────────┤          ├──────────────┤                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│ python3.11-minimal    │ CVE-2024-6232  │          │ affected     │                         │               │ python: cpython: tarfile: ReDos via excessive backtracking   │
│                       │                │          │              │                         │               │ while parsing header values                                  │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-6232                    │
│                       ├────────────────┤          ├──────────────┤                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                       │ CVE-2024-7592  │          │ fix_deferred │                         │               │ cpython: python: Uncontrolled CPU resource consumption when  │
│                       │                │          │              │                         │               │ in http.cookies module                                       │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-7592                    │
├───────────────────────┼────────────────┼──────────┼──────────────┼─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ wget                  │ CVE-2024-38428 │ CRITICAL │ affected     │ 1.21.3-1+b2             │               │ wget: Misinterpretation of input may lead to improper        │
│                       │                │          │              │                         │               │ behavior                                                     │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2024-38428                   │
├───────────────────────┼────────────────┤          ├──────────────┼─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ zlib1g                │ CVE-2023-45853 │          │ will_not_fix │ 1:1.2.13.dfsg-1         │               │ zlib: integer overflow and resultant heap-based buffer       │
│                       │                │          │              │                         │               │ overflow in zipOpenNewFileInZip4_6                           │
│                       │                │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-45853                   │
├───────────────────────┤                │          │              │                         ├───────────────┤                                                              │
│ zlib1g-dev            │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
│                       │                │          │              │                         │               │                                                              │
└───────────────────────┴────────────────┴──────────┴──────────────┴─────────────────────────┴───────────────┴──────────────────────────────────────────────────────────────┘


```
