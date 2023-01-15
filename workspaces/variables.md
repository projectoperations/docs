# Environment variables

Coder injects a standard set of environment variables that allow you to access
contextual information about your workspace.

To obtain a list of environment variables and their values, launch the
**Terminal** via the Coder Dashboard and run:

```console
env | grep CODER_
```

## Available environment variables

<!-- markdownlint-disable MD044 -->
<table>
    <tr>
        <th>Environment variable</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><code>CODER_ASSETS_ROOT</code></td>
        <td>The directory where coder adds Coder-specific assets during
            workspace creation, such as the <code>coder-cli</code> binary</td>
    </tr>
    <tr>
        <td><code>CODER_CPU_LIMIT</code></td>
        <td>The CPU core limit given to your workspace</td>
    </tr>
    <tr>
        <td><code>CODER_IMAGE_DIGEST</code></td>
        <td>The content-addressable identifier for your image</td>
    </tr>
    <tr>
        <td><code>CODER_IMAGE_TAG</code></td>
        <td>The image tag used to create your workspace</td>
    </tr>
    <tr>
        <td><code>CODER_IMAGE_URI</code></td>
        <td>The URI of the image used to build the workspace</td>
    </tr>
    <tr>
        <td><code>CODER_MEMORY_LIMIT</code></td>
        <td>The memory limit given to your workspace in GB</td>
    </tr>
    <tr>
        <td><code>CODER_ORGANIZATION_ID</code></td>
        <td>The ID of the organization to which the workspace belongs.</td>
    </tr>
    <tr>
        <td><code>CODER_RUNTIME</code></td>
        <td>The container runtime used to start the workspace (either
        <code>kubernetes/default</code> or <code>kubernetes/sysbox</code>
        if the workspace is a CVM</td>
    </tr>
    <tr>
        <td><code>CODER_URL</code></td>
        <td>The base URL of your Coder deployment</td>
    </tr>
    <tr>
        <td><code>CODER_USER_EMAIL</code></td>
        <td>Your email address</td>
    </tr>
    <tr>
        <td><code>CODER_USERNAME</code></td>
        <td>Your user name</td>
    </tr>
    <tr>
        <td><code>CODER_WORKSPACE_ID</code></td>
        <td>The unique ID of your workspace</td>
    </tr>
    <tr>
        <td><code>CODER_WORKSPACE_NAME</code></td>
        <td>The name of your workspace</td>
    </tr>
    <tr>
        <td><code>CODER_WP_NAME</code></td>
        <td>The name of the workspace provider hosting the workspace</td>
    </tr>
</table>

<!-- markdownlint-restore -->

## Other variables

Coder uses the following environment variables for its internal operation:

> **Important**: These variables may be modified or removed in future releases
> without prior notice and are not covered by Coder's backward compatibility
> policy. These are documented here for your information only.

<!-- markdownlint-disable MD044 -->
<table>
    <tr>
        <th>Environment variable</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><code>CODER_AGENT_TOKEN</code></td>
        <td>The token used by the <a href="../setup/architecture.md">Coder Agent</a>
            to authenticate with <code>coderd</code>. The Coder Agent handles
            tunnelled connections, collects workspace statistics (such as
            processor and memory utilization), and manages programs, such as
            editors.</td>
    </tr>
    <tr>
        <td><code>CODER_AGENT_URL</code></td>
        <td>The URL that the <code>coder agent</code> process uses
            to connect to <code>coderd</code>. If this is not set, the agent
            will connect to <code>CODER_URL</code> instead, enabling the
            <code>coder agent</code> and <code>coder</code> command-line tool
            to connect to different installations.</td>
    </tr>
    <tr>
        <td><code>CODER_BOOTSTRAP_SCRIPT</code></td>
        <td>The script used to initiate the workspace startup sequence when the
            <a href="../admin/workspace-management/self-contained-builds.md">
            self-contained workspace builds</a> feature is active; begins by
             downloading the "bootstrap agent" from the <code>coderd</code>
             process.</td>
    </tr>
</table>
<!-- markdownlint-restore -->
