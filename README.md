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
| `github_token` | GitHub token for authentication | Yes |
| `telegram_to` | Telegram recipient/chat ID | Yes |
| `telegram_token` | Telegram bot token | Yes |
| `docker_hub_username` | Docker Hub username | Yes |
| `docker_hub_access_token` | Docker Hub access token | Yes |
| `docker_repo_name` | Docker Hub repository name | Yes |
| `cache_tag` | Tag in the same Docker Hub repository that stores the BuildKit layer cache (`type=registry,mode=max`). Default: `buildcache` | No |

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
    # The action pushes a tag and creates a release.
    permissions:
      contents: write
    steps:
      - name: Build and Push Docker Image
        uses: yuri-val/build-and-push-docker-image-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
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
- Self-hosted runners need Actions Runner v2.327.1 or later (Node 24 actions).
- Add a `.dockerignore` that excludes at least `.git` and `.env*`. The build context is the
  whole checkout, so a `COPY . .` without it ships your git history and local secrets inside
  the image. (The action no longer leaves a git token in the checkout, but history and
  untracked files are still there.)

## Security

- Every action the composite uses is pinned to a commit SHA; Dependabot keeps the pins current.
- Telegram is called directly with `curl`, so the bot token is only ever sent to
  `api.telegram.org`. The tag is created with the first-party `actions/github-script`; no
  third-party code receives the GitHub token except the pinned `ncipollo/release-action`.
- The checkout runs with `persist-credentials: false`, so no token is left in `.git/config`
  inside the Docker build context.

## How it works

1. The action checks out your repository.
2. It logs in to Docker Hub using the provided credentials.
3. The Docker image is built and pushed to Docker Hub with two tags:
   - A date-based tag (e.g., `23.05.15.1234`)
   - The `latest` tag

   BuildKit layer cache is read from and written to `<docker_hub_username>/<docker_repo_name>:<cache_tag>`
   (`type=registry,mode=max`), so dependency layers are reused across runs and across
   runners — GitHub-hosted and self-hosted alike. The tag shows up in Docker Hub next to
   the image tags; it is not a runnable image.
4. A new GitHub release is created. Its body is `git log --no-merges --pretty='- %s'`
   between the previous tag reachable from `HEAD` and `HEAD` itself, and the same range
   is attached as a `.diff` artifact. Both need real history, which is why the action
   checks out with `fetch-depth: 0` — it runs after the caller's checkout and would
   otherwise shallow the workspace back to a single commit.
5. Telegram notifications are sent at the start of the process, on successful completion, and in case of failure.

## Author

Yuri V <yuri.valigursky@gmail.com> (@yuri-val)

## License

This project is licensed under the [MIT License](LICENSE).
