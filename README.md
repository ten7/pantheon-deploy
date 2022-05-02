# Pantheon Deploy

An Ansible role to deploy a site to Pantheon from an external git repo.

# Overview

Pantheon has the ability to deploy sites over standard git. Often, however, you may wish to maintain your site in your own repository to take advantage of additional tooling such as Pull Requests, code reviews, and issue tracker integration.

Beyond integration, you may also want to:
* Use a different branch strategy such as Git Flow.
* Use `main` instead of `master` for your primary branch.
* Delegate CSS pre-processing to a Continuous Integration process.

This role allows you to do all of these things.

## How it works

This role does *not* configure a multi-remote git repository. Instead, the Pantheon repository is cloned in a temporary directory and then changes are rsynced to it from the source repository. This avoids git conflicts during the build process.

Furthermore, it allows some differences to exist between the two repos in key ways.

### Pantheon-specific gitignore

In your source repository, you can define a `.gitignore-pantheon` file. This file will not be used for ignores by git in your source respository. During the build, however, it is copied to `.gitignore` in the Pantheon repository, allowing you to create different ignores for the two sites. It is highly recommended to copy the `.gitignore` supplied by Pantheon when initially setting up your site.

### CSS preprocessing

If `package.json` is in the root of your repository (or you defined `pantheon_deploy.build.npm_dir`), this role will run `npm ci` and `npm run build`. If you specified `pantheon_deploy.build.npm_build_script_name`, the specified command name will be run instead.

## Requirements

* The ansible.posix collection must be installed.
* rsync must be installed
* git must be installed
* If using npm, npm must be installed.
* The target pantheon site must be in git mode.

## Role Variables

```yaml
pantheon_deploy:
  source:
    git_dir: 'path/to/source/git/dir'
  build:
    npm_dir: 'path/in/repo/to/package.json'
    npm_build_script_name: 'build'
  target:
    ssh_key_base64: 'abcdef1234567890'
    ssh_pub_base64: 'abcdef1234567890'
    repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
```

Where:

* `pantheon_deploy.source.git_dir` is the path to locally cloned git repository from the external git host. Required.
* `pantheon_deploy.build.npm_dir` is the path inside the repo to package.json. Optional, defaults to the `package.json` in the root of the repo.
* `pantheon_deploy.build.npm_build_script_name` is the name of the script to invoke with `npm run`. Optional, defaults to `build`.
* `pantheon_deploy.target.ssh_key_base64` is the Base64 encoded SSH private key used to communicate with Pantheon.
* `pantheon_deploy.target.ssh_pub_base64` is the Base64 encoded SSH public key used to communicate with Pantheon.
* `pantheon_deploy.target.repo_url` is the SSH URL to the Pantheon site repository.

## Dependencies

* ansible.posix

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
    - hosts: servers
      vars:
        pantheon_deploy:
          source:
            git_dir: 'path/to/source/git/dir'
          build:
            npm_dir: 'path/in/repo/to/package.json'
            npm_build_script_name: 'build'
          target:
            ssh_key_base64: 'abcdef1234567890'
            ssh_pub_base64: 'abcdef1234567890'
            repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
      roles:
         - { role: ten7.pantheon_deploy }
```

## License

GPL v3

## Author Information

This role was created by [TEN7](https://ten7.com/).
