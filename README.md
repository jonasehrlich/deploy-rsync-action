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

| Input                | Description                                 | Required | Default                                               |
| -------------------- | ------------------------------------------- | -------- | ----------------------------------------------------- |
| `remote_host`        | Host for deployment                         | Yes      |                                                       |
| `remote_user`        | User for deployment                         | Yes      |                                                       |
| `remote_key`         | Private key for deployment                  | Yes      |                                                       |
| `remote_path`        | Directory to deploy to on the remote server | Yes      |                                                       |
| `path`               | Local source directory to deploy            | Yes      |                                                       |
| `rsync_switches`     | rsync command switches                      | No       | `--verbose --archive --compress --recursive --delete` |
| `rsync_add_switches` | Additional rsync command switches           | No       |                                                       |

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

### Example 2: Deploy to custom directory

```yaml
- name: Deploy to pages
  uses: jonasehrlich/deploy-rsync-action@v1
  with:
    remote_host: ${{ secrets.DEPLOY_HOST }}
    remote_user: ${{ secrets.DEPLOY_USER }}
    remote_key: ${{ secrets.DEPLOY_KEY }}
    remote_path: "/var/www/html/sites/myproject"
    path: "_build/"
```

### Example 3: Deploy with custom rsync options

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

## Path Configuration

The action requires you to specify the complete remote path where files will be deployed:

```yaml
remote_path: "/var/www/html/my-site"
```

The specified directory will be created on the remote server if it doesn't exist.

## Required Secrets

Set up the following secrets in your repository:

- `DEPLOY_HOST`: The SSH hostname or IP address
- `DEPLOY_USER`: SSH username for authentication
- `DEPLOY_KEY`: SSH private key for authentication

## Security Considerations

- Store SSH private keys as repository secrets
- Use dedicated deployment keys with minimal required permissions
- Consider using SSH key passphrases for additional security
- Regularly rotate deployment keys

## License

This action is provided under the MIT License.
