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

    repo:
the url of a git repo, in either https or ssh format

    version:
the commit id, tag, or branch to build. If a branch is specified, we use the current HEAD

    workdir:
the working directory, in which the project will be checked out and built

    artefact:
      - type:
        name:
a list of artefacts to produce. "type:" can currently be either 'tarball' or 'docker', and pointing 'name:' at the contents of the version var from above is usually the right thing to do.

    ci_key: 
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

---
## Example playbook

    - hosts: all
      roles:
        - webops-role-build
      vars_files:
        - config/build_public.yml
      gather_facts: False # For compatibility with newer Linux distributions without python 2
      pre_tasks:
      - name: Install python 2
        raw: "test -e /usr/bin/python || (sudo apt-get -q -y update && sudo apt-get -q install -y python-minimal)"
      - name: Gather Facts # Resume Ansible fact-gathering from here
        setup:	

## Example vars file
*(Used to build our rnd17 tarball for deployment to production)*

    version: 1.29.1
    repo: git@github.com:github/testrepo.git # replaces the real repo for documentation purposes
    workdir: /srv/build
    ci_key: "{{ lookup('env', 'CI_KEY') }}"
    artefact:
      type: tarball
      name: "{{ version }}.tar.gz"
    default_task_command: phing -S
    build_tasks:
    - target: /usr/bin/nodejs /usr/bin/node
      command: "[ -h /usr/bin/node ] || /bin/ln -s" # phing expects 'nodejs' to be called 'node'
    - target: "'drush.bin = {{ansible_user_dir}}/vendor/bin/drush' > build.properties"
      command: /bin/echo
    - target: build:prepare:deploy
    - target: grunt:build
    post_build_remove: .git
    post_build_symlinks:
    - source: ../sites
      dest: web/sites
    - source: ../modules
      dest: web/modules
    - source: ../themes
      dest: web/themes
    - source: ../pinterest-742e7.html
      dest: web/pinterest-742e7.html
    - source: ../../cr
      dest: web/profiles/cr
    apt_repos:
      - ppa:brightbox/ruby-ng
      - deb https://deb.nodesource.com/node_6.x trusty main
      - ppa:ondrej/php
    apt_repo_keys:
      - https://deb.nodesource.com/gpgkey/nodesource.gpg.key
      - https://packages.sury.org/php/apt.gpg
    apt_packages:
      - nodejs
      - ruby2.1
      - ruby2.1-dev # needed for json gem
      - php5.6
      - php5.6-dev
      - php5.6-cli
      - php5.6-xml
      - php5.6-json
      - php5.6-mbstring
      - php5.6-bcmath
      - php-amqp
    needs_gem: true
    needs_composer: true
    needs_npm: true
    needs_pip: false
    gem_packages:
      - bundler
    composer_packages:
      - drush/drush:8.1
      - phing/phing:2.15.0
    system_apt_packages:
      - git
      - unzip
      - python-setuptools # needed to install pip
      - build-essential
      - checkinstall
      - libpcre3-dev