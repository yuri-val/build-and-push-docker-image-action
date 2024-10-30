# Build and Push Docker Image Action

This GitHub Action builds a Docker image, pushes it to Docker Hub, creates a GitHub release, and sends Telegram notifications throughout the process.

## Features

- Builds Docker image from your repository
- Pushes the image to Docker Hub with date-based and 'latest' tags
- Creates a GitHub release with an auto-generated changelog
- Sends Telegram notifications for start, success, and failure events

## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `telegram_to` | Telegram recipient/chat ID | Yes |
| `telegram_token` | Telegram bot token | Yes |
| `docker_hub_username` | Docker Hub username | Yes |
| `docker_hub_access_token` | Docker Hub access token | Yes |
| `docker_repo_name` | Docker Hub repository name | Yes |

## Usage

To use this action in your workflow, create a `.github/workflows/docker-build-push.yml` file in your repository with the following content:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main  # or any branch you want to trigger the action

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build and Push Docker Image
        uses: yuri-val/build-and-push-docker-image-action@v1
        with:
          telegram_to: ${{ secrets.TELEGRAM_TO }}
          telegram_token: ${{ secrets.TELEGRAM_TOKEN }}
          docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
          docker_hub_access_token: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
          docker_repo_name: your-repo-name
```

Make sure to set up the following secrets in your repository:

- `TELEGRAM_TO`: Your Telegram recipient/chat ID
- `TELEGRAM_TOKEN`: Your Telegram bot token
- `DOCKER_HUB_USERNAME`: Your Docker Hub username
- `DOCKER_HUB_ACCESS_TOKEN`: Your Docker Hub access token

## Requirements

- Your repository must contain a `Dockerfile` in the root directory.
- You need to have a Docker Hub account and create an access token.
- You need to create a Telegram bot and obtain its token.

## How it works

1. The action checks out your repository.
2. It logs in to Docker Hub using the provided credentials.
3. The Docker image is built and pushed to Docker Hub with two tags:
   - A date-based tag (e.g., `23.05.15.1234`)
   - The `latest` tag
4. A new GitHub release is created with an auto-generated changelog.
5. Telegram notifications are sent at the start of the process, on successful completion, and in case of failure.

## Author

Yuri V <yuri.valigursky@gmail.com> (@yuri-val)

## License

This project is licensed under the [MIT License](LICENSE).
