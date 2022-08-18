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
    git_branch: 'main'
  build:
    npm_dir: 'path/in/repo/to/package.json'
    npm_install_cmd: 'ci'
    npm_build_script_name: 'build'
  target:
    ssh_key_base64: 'abcdef1234567890'
    ssh_pub_base64: 'abcdef1234567890'
    pantheon_machine_token: ''
    site_id: ''
    env_id: ''
    repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
    git_branch: 'master'
    git_commit_message: "Made with <3 by robots"
```

Where:

* `pantheon_deploy.source.git_dir` is the path to locally cloned git repository from the external git host. Required.
* `pantheon_deploy.source.git_branch` is the branch to use in the locally cloned git repository from the exteneral git host.. Required.
* `pantheon_deploy.build.npm_dir` is the path inside the repo to package.json. Optional, defaults to the `package.json` in the root of the repo.
* `pantheon_deploy.build.npm_install_cmd` is the install command to use with `npm`, including switches. Optional, defaults to `ci`.
* `pantheon_deploy.build.npm_build_script_name` is the name of the script to invoke with `npm run`. Optional, defaults to `build`.
* `pantheon_deploy.target.ssh_key_base64` is the Base64 encoded SSH private key used to communicate with Pantheon. Required.
* `pantheon_deploy.target.ssh_pub_base64` is the Base64 encoded SSH public key used to communicate with Pantheon. Required.
* `pantheon_deploy.target.repo_url` is the SSH URL to the Pantheon site repository. Required.
* `pantheon_deploy.target.git_branch` is the git branch to push to the Pantheon site repository. Required.
* `pantheon_deploy.target.git_commit_message` is the git commit message to use when pushing to the Pantheon site repository. Optional, defaults to "Commit by ten7.pantheon_deploy".

### Secrets

Sometimes you may need to template files during the build process which contain sensitive information such as API keys, passwords, or certificates. You can accomplish this using `secrets`.

```yaml
pantheon_deploy:
  source:
    git_dir: 'path/to/source/git/dir'
    git_branch: 'main'
  build:
    npm_dir: 'path/in/repo/to/package.json'
    npm_build_script_name: 'build'
  target:
    ssh_key_base64: 'abcdef1234567890'
    ssh_pub_base64: 'abcdef1234567890'
    pantheon_machine_token: ''
    site_id: ''
    env_id: ''
    repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
    git_branch: 'master'
    git_commit_message: "Made with <3 by robots"
  secrets:
    - path: "web/private/secrets/super_secret_stuff.txt"
      value: "catsAreCute"
```

Where:

* `pantheon_deploy.secrets` is a list of secrets to add to the Pantheon repository. Optional.

Each item in `pantheon_deploy.secrets` has the following values:

* `path` is the path to write the secret, relative to the root of the Pantheon repository. Required.
* `value` is the value to write. Required.

When running this role from a CI system, you may have secrets available as environment variables. In that case, you can use the following to write them to the file:

```yaml
pantheon_deploy:
...
  secrets:
    - path: "web/private/secrets/super_secret_stuff.txt"
      value: "{{ lookup('env', 'ENVVAR_FROM_CI') }}"
```

As an added option you may export secrets in a JSON file:

```yaml
pantheon_deploy:
...
  secrets:
    json:
      - path: "web/private/secrets/super_secret_data.json"
        values:
          secret_stuff: "stuff"
          secret_things: "things"
```

Which would produce the following JSON:
```json
{"secret_stuff":"stuff","secret_things":"things"}
```


### Post Deploy commands

This role has the ability to execute Drush commands post deploy through `terminus`. 

Note that this functionality may be useful in some cases where [Quicksilver hooks](https://pantheon.io/docs/quicksilver#hooks) cannot be used.

```yaml
pantheon_deploy:
  source:
    git_dir: 'path/to/source/git/dir'
    git_branch: 'main'
  build:
    npm_dir: 'path/in/repo/to/package.json'
    npm_build_script_name: 'build'
  target:
    ssh_key_base64: 'abcdef1234567890'
    ssh_pub_base64: 'abcdef1234567890'
    pantheon_machine_token: ''
    site_id: ''
    env_id: ''
    repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
    git_branch: 'master'
    git_commit_message: "Made with <3 by robots"
  post_deploy:
    drush_commands:
      - "updb -y"
      - "cim -y"
      - "cr"
```

Where:

* `pantheon_deploy.target.pantheon_machine_token` is a Pantheon Machine Token used for Terminus commands. Required for post-deploy commands.
* `pantheon_deploy.target.site_id` is the Pantheon Site ID. Required for post-deploy commands.
* `pantheon_deploy.target.env` is the Pantheon environment name. Required for post-deploy commands.
* `pantheon_deploy.post_deploy.drush_commands` is a list of Drush commands to execute on the target environment.


### Custom tasks

Often, you may wish to execute custom CI code during the build and deployment process. You can accomplish that with `include_tasks`:

```yaml
pantheon_deploy:
  source:
    git_dir: 'path/to/source/git/dir'
    git_branch: 'main'
  build:
    npm_dir: 'path/in/repo/to/package.json'
    npm_build_script_name: 'build'
    include_tasks:
      - "path/to/my/build.yml"
  target:
    ssh_key_base64: 'abcdef1234567890'
    ssh_pub_base64: 'abcdef1234567890'
    pantheon_machine_token: ''
    site_id: ''
    env_id: ''
    repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
    git_branch: 'master'
    git_commit_message: "Made with <3 by robots"
  post_deploy:
    include_tasks:
      - "path/to/my/post_deploy.yml"
```

Where:

* `pantheon_deploy.build.include_tasks` is a list of paths to an Ansible tasks file to execute during the build but before the deploy. Optional.
* `pantheon_deploy.post_deploy.include_tasks` is a list of paths to an Ansible tasks file to execute after the deploy. Optional.

The paths for `include_tasks` can be absolute, or relative to the playbook from which this role is executing.

## Dependencies

* ansible.posix
* The `terminus` command must be installed to use Post Deploy tasks.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
    - hosts: servers
      vars:
        pantheon_deploy:
          source:
            git_dir: 'path/to/source/git/dir'
            git_branch: 'main'
          build:
            npm_dir: 'path/in/repo/to/package.json'
            npm_build_script_name: 'build'
          target:
            ssh_key_base64: 'abcdef1234567890'
            ssh_pub_base64: 'abcdef1234567890'
            repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
            git_branch: 'master'
            git_commit_message: "Made with <3 by robots"
      roles:
         - { role: ten7.pantheon_deploy }
```

## License

GPL v3

## Author Information

This role was created by [TEN7](https://ten7.com/).
