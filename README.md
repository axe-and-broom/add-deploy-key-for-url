# Add deploy key for url

This GitHub Action installs a deploy SSH key in `~/.ssh` only for a specific git URL. This allows you to use multiple keys for the same domain given that there are different URLs (e.g., deploy-key-1 for `git@github.com:user1/repo1.git` and deploy-key-2 for `git@github.com:user2/repo2.git`).

## Why?

Normally when you add a deploy key to the SSH config, it applies to all repositories on that domain meaning that you cannot use different deploy keys for different repositories on the same domain (e.g., github.com). The normal workaround is to create a bot user account and use that account's SSH key as a deploy key for multiple repositories. However, this has security and auditability drawbacks.

This action uses `git config url.insteadOf` with a random string per URL to create an alias for the specific git URL and then configures SSH to use the specific deploy key only for that alias. This allows you to use different deploy keys for different repositories on the same domain.

We have used this action successfully in production workflows ourselves to use unique deploy keys for multiple repositories on the github.com domain for different repositories with dependencies.

## Usage
```yaml
- uses: axe-and-broom/add-deploy-key-for-url@v1
  with:
    # The hostname of the git repository (example: github.com)
    # Required: true
    host: 'github.com'
        
    # The full url to the git repository (example: git@github.com/some-org/some-repo.git)
    # Required: true
    git-url: 'git@github.com/some-org/some-repo.git'

    # The private key to use with the git url
    # Required: true
    private-key: '${{ secrets.SOME_REPO_DEPLOY_SSH_KEY }}'
```


## Example workflow

```yaml
name: Example Workflow
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Add deploy key for git url
        uses: axe-and-broom/add-deploy-key-for-url@v1
        with:
          host: 'github.com'
          git-url: 'git@github.com/some-org/some-repo.git'
          private-key: '${{ secrets.SOME_REPO_DEPLOY_SSH_KEY }}'
      
      - name: Add other deploy key for another git url
        uses: axe-and-broom/add-deploy-key-for-url@v1
        with:
          host: 'github.com'
          git-url: 'git@github.com/another-org/another-repo.git'
          private-key: '${{ secrets.ANOTHER_REPO_DEPLOY_SSH_KEY }}'
      
      - name: Run build script that uses the git urls for dependencies
        run: ./build-and-deploy.sh
```

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.