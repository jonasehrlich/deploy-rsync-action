# Deploy via SSH and Rsync

> [!NOTE]
> This action runs steps using Docker. Make sure the steps can be run in your actions runner.

A GitHub composite action that sets up a remote directory via SSH and deploys files using rsync.
This action combines directory creation and file synchronization into a single reusable workflow
step.

## Features

- Creates target directory structure on remote server via SSH
- Deploys files using rsync with customizable options
- Supports flexible path configuration (absolute paths or root + relative paths)
- Configurable rsync switches for different deployment scenarios

## Inputs

| Input                | Description                                                                                 | Required | Default                                               |
| -------------------- | ------------------------------------------------------------------------------------------- | -------- | ----------------------------------------------------- |
| `remote_host`        | Host for deployment                                                                         | Yes      |                                                       |
| `remote_user`        | User for deployment                                                                         | Yes      |                                                       |
| `remote_key`         | Private key for deployment                                                                  | Yes      |                                                       |
| `remote_path`        | Directory to deploy to on the remote server. The path will be created if it does not exist. | Yes      |                                                       |
| `path`               | Local source directory to deploy                                                            | Yes      |                                                       |
| `rsync_switches`     | rsync command switches                                                                      | No       | `--verbose --archive --compress --recursive --delete` |
| `rsync_add_switches` | Additional rsync command switches                                                           | No       |                                                       |

## Usage

### Deploy to remote path

```yaml
name: Deploy Documentation
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build documentation
        run: |
          # Your build steps here
          mkdir -p _build
          echo "Built documentation" > _build/index.html

      - name: Deploy to server
        uses: jonasehrlich/deploy-rsync-action@v1
        with:
          remote_host: ${{ secrets.DEPLOY_HOST }}
          remote_user: ${{ secrets.DEPLOY_USER }}
          remote_key: ${{ secrets.DEPLOY_KEY }}
          remote_path: "/var/www/html/docs"
          path: "_build/"
```

### Deploy to directory including `owner/repo`

```yaml
- name: Deploy to server
  uses: jonasehrlich/deploy-rsync-action@v1
  with:
    remote_host: ${{ secrets.DEPLOY_HOST }}
    remote_user: ${{ secrets.DEPLOY_USER }}
    remote_key: ${{ secrets.DEPLOY_KEY }}
    remote_path: "${{ secrets.DEPLOY_ROOT }}/${{ github.repository }}$"
    path: "_build/"
```

### Deploy with custom rsync options

```yaml
- name: Deploy with custom rsync options
  uses: jonasehrlich/deploy-rsync-action@v1
  with:
    remote_host: ${{ secrets.DEPLOY_HOST }}
    remote_user: ${{ secrets.DEPLOY_USER }}
    remote_key: ${{ secrets.DEPLOY_KEY }}
    remote_path: "/var/www/html/app"
    path: "build/"
    rsync_switches: "--verbose --archive --compress --recursive --delete --exclude=*.log --exclude=.env"
```

## Required Secrets

Set up the following secrets in your repository:

- `DEPLOY_HOST`: The SSH hostname or IP address
- `DEPLOY_USER`: SSH username for authentication
- `DEPLOY_KEY`: SSH private key for authentication

It is also recommended to store the remote path (or its root) as a secret.

## License

This action is provided under the MIT License.
