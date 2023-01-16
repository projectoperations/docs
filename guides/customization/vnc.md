# VNC

This guide will show you how to set up a VNC in Coder.

Coder does not have a specific set of VNC providers it supports. Coder will
render the VNC, as long as it is installed on the image used to create the
workspace.

## Step 1: Create the Dockerfile

To begin, create a Dockerfile that you'll use to build an
[image](../../images/index.md) with a VNC provider installed.

Be sure to set the `HOME`, `USER`, and `PORT` environment variables in the
Dockerfile:

```text
HOME=/home/coder
USER coder
PORT 1234
```

**Note:** Set `PORT` to the appropriate port number for your VNC instance.

> To help you get started, see this
> [sample image](https://github.com/coder/enterprise-images/tree/main/images/vnc)
> that uses [noVNC](https://github.com/novnc/noVNC) as the client and
> [TurboVNC](https://turbovnc.org/) as the server.

## Step 2: Build and push the image to Docker Hub

Once you've created your image, build and push it to Docker Hub:

```console
docker build . -t <yourusername>/vnc
docker push <yourusername>/vnc
```

## Step 3: Import the image into Coder

Now that your image is available via Docker Hub, you can import it for use in
Coder.

1. Log in to Coder and go to **Images** > **Import Image**

1. Import or select a [registry](../../admin/registries/index.md).

1. Provide the **Repository** and **Tag** of the VNC image. Optionally, you can
   include a **Description** and the **Source Repo URL** that refers to the
   image's source.

1. Set the recommended resources (CPU cores, memory, disk space) for your VNC
   instance.

1. Click **Import Image**.

## Step 4: Create a workspace with the image

Once you've imported your image into Coder, you can use it to create an
workspace.

1. In the Coder UI, go to the **workspace overview** page. Click **New
   Workspace** and choose **Custom Workspace**

1. Provide a **Workspace Name**, and indicate that your **Image Source** is
   **Existing**.

1. Select your **Image** and associated **Tag**.

1. Click **Create Workspace**

## Connecting to Coder

There are two ways you can connect to your workspace:

- Connect via the web
- Connect using a local VNC client

### Option 1: Connect via web

If your image includes [noVNC](https://github.com/novnc/noVNC), or another
web-based client, you can use a [dev URL](../../workspaces/devurls.md) to access
it securely.

1. From the **workspace overview** page, click **Add URL**

1. Provide the **Port** number that the VNC web client is running on (this
   information is defined in the image you used to build this workspace).

1. Provide a **name** for the dev URL.

1. Click **Save**.

You can now access the VNC in Coder by clicking the **Open in Browser** icon
(this will launch a separate window).

### Option 2: Connect using a local VNC client

If your Coder deployment has
[ssh](https://coder.com/docs/admin/workspace-management/ssh-access) enabled, you
can also connect to Coder using a local client with SSH port forwarding.

You will need to install [coder-cli](https://github.com/coder/coder-cli) and a
VNC client on your local machine.

Run the following commands on your local machine to connect to the VNC server.
Replace `[vnc-port]` with the port on which the server is running and
`[workspace-name]` with the workspace you created in **Step 4**.

```console
# Ensure the workspace you created is an SSH target
coder config-ssh

# Forward the remote VNC server to your local machine
# Note that you will not see any output if this succeeds
ssh -L -N [vnc-port]:localhost:localhost:[vnc-port] coder.[workspace-name]

# At this point, you can connect your VNC client to localhost:[vnc-port]
```
