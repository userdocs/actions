# Docker Container Builder Action

Creates and configures Docker containers for CI/CD workflows with cross-platform support and user management.

## Inputs

| Input                    | Description                                                    | Required | Default    |
| ------------------------ | -------------------------------------------------------------- | -------- | ---------- |
| `distinct_id`            | Unique identifier for the container instance                   | false    | `""`       |
| `dockerfile`             | A url to raw or a filename assumed to be local  to the repo    | false    | `""`       |
| `use_host_env`           | Include host environment variables in container                | false    | `"false"`  |
| `use_root`               | Run container as root 0:0 with gh 1001:1001 available to use   | false    | `"false"`  |
| `os_id`                  | Docker image name/OS identifier (e.g., alpine, ubuntu, debian) | false    | `"alpine"` |
| `os_version_id`          | Docker image tag/OS version (e.g., edge, latest, 22.04)        | false    | `"edge"`   |
| `custom_docker_commands` | Additional Docker run arguments (space-separated)              | false    | `""`       |
| `additional_alpine_apps` | Extra Alpine packages to install (space-separated)             | false    | `""`       |
| `additional_debian_apps` | Extra Debian/Ubuntu packages to install (space-separated)      | false    | `""`       |

> [!note]
> using `dockerfile` will not configure the docker command other than to load host env or custom env if enabled/used.
>
> Only `use_host_env` or a `custom.env` file will have any effect on a dockerfile build.
>
> The assumption is that your dockerfile is configured how you need it and we are just using it.

## env created

| `GITHUB_ENV`     | Description                            |
| ---------------- | -------------------------------------- |
| `container_name` | `qbt_builder`                          |
| `wd`             | The working directory in the container |

## Usage Example

```yml
    # if the env.custom file exists, it will be used to pass as env to container.
    # You create env.custom file in the steps before this then it is appended to the host env if use_host_env is true
- uses: userdocs/actions/qbt_docker@main
  id: qbt_docker
  with:
    # distinct_id: "" # for codex-/return-dispatch action
    # dockerfile: "" # url to raw or a filename of a local file.
    # use_host_env # load the runner env into the container. This is what happens when you use container:
    # use_root: "false" # this starts the container as the root user but gh 1001:1001 is still available to use on demand
    os_id: ${{ matrix.os_id }} # this action works with ghcr.io/userdocs/* images and generic images like debian/ubuntu/alpine
    os_version_id: ${{ matrix.os_version_id }}
    # custom_docker_commands: "" # custom docker cli commands to run in the container
    # additional_alpine_apps: "curl git" # apk add this string
    # additional_debian_apps: "curl git" # apt install this string

    # These are already set to GITHUB_ENV via the action. Just an alternative way to access them
- name: Use qbt_docker outputs
  run: |
    printf '%s\n' "container_name=${{ steps.qbt_docker.outputs.container_name }}"
    printf '%s\n' "uid=${{ steps.qbt_docker.outputs.uid }}"
    printf '%s\n' "gid=${{ steps.qbt_docker.outputs.uid }}"
    printf '%s\n' "wd=${{ steps.qbt_docker.outputs.wd }}"
```