# Ansible Role: Build

Builds a PHP, Ruby, Python, or Nodejs software project using tools such as GNU Make, Phing, Composer, NPM, Rake, Grunt, or shell scripts.

Stderr and stdout from each build tool can be presented in-line as your playbook runs, or sent separately to a file or logstash endpoint with each line tagged with useful metadata.

Output is a deployment artefact of the following types:
- a tarball for use on production servers, in conjunction with webops-role-deploy
- a docker image, for use in local development or on a PaaS for testing 

All dependencies should be explicitly declared, and the *molecule* test script can test for correctness.

---
## Useful variables
*(Role defaults, used for testing, are noted when appropriate)*

    repo: git@github.com:github/testrepo.git
the url of a git repo, in either https or ssh format

    version: master
the commit id, tag, or branch to build. If a branch is specified, we use the current HEAD

    workdir: /srv/build
the working directory, in which the project will be checked out and built

    artefact:
      type: tarball
      name: "{{ version }}"
details about the artefact to produce. "type:" can currently be either 'tarball' or 'docker', and pointing 'name:' at the contents of the version var from above is usually the right thing to do.

    ci_key: "{{ lookup('env', 'CI_KEY') }}" 
the private key needed to access a private git repo with ssh. **DO NOT include this as an unencrypted variable** - either use an ansible secret, or do a lookup from an env var, Consul, or other secure source. 

    build_tasks:
      - target:
        command:
        output:
a list of build steps to run, in sequential order

    default_task_command:
    default_task_output:
a default command and/or output which is applied to any targets where one is not explicitly set

    needs_gem:
    needs_composer:
    needs_npm:
    needs_pip:
    needs_rake:
if any of these is set to be true, the latest release of the appropriate build tool will be installed.

    apt_packages:
    npm_packages:
    gem_packages:
    pip_packages:
    composer_packages:
each takes a list of packages to be installed system-wide with the appropriate tool.

    gem_local_paths:
    npm_local_paths:
    composer_local_paths
a list of paths relative to the workdir containing a Gemfile, package.json, or composer.json respectively. Used for installing local dependencies into the project. *Note: if "gem_local_paths:" is used, bundler must be installed using either "apt_packages:" or "gem_packages:"* 

    pre_build_files:
      - source:
        dest:
    post_build_files:
      - source:
        dest:
a list of files to be installed on the system either before the build begins, or after it completes. "source:" should be filename within the "files/" directory of your playbook, and can be a plain file or a jinja2 template.

    post_build_remove:
a list of files or directories to be removed after the build has run

    post_build_symlinks:
      - to:
        from:
a list of symlinks to create after the build has run. The value of 'to:' is where the symlink points to, and should usuall be in the form of a relative path.  