# Ansible Elixir Role

Simple deployment role for Exrm-based releases

# Installation

`ansible-galaxy install andrewvy.elixir`

Or manually, by cloning the repo. :)

# Setup

First, you need to make sure what your host layout looks like.

I'm running this role with a dedicated build server that separates the compile process from the actual running server environment.

**You must make sure that the server that does the compiling has git access to your repo.**

This is because the compiled server will commit and tag the released version to your repo.

You can achieve this either through SSH Agent Forwarding, or by registering your build server's SSH keys with GitHub.

Next, you need to make sure that your compile server has erlang and elixir installed. This role doesn't handle those dependencies, I highly recommend using [ohr486.elixir](https://github.com/ohr486/ansible-elixir)

However, the `setup` action will install Hex and Rebar for you. Which ohr486 does not do.

*setup-playbook.yml*

```
---
- hosts: build_servers
  sudo: yes
  roles:
    - { role: ohr486.elixir, erlang_version: 18.1, elixir_version: v1.1.1 }
    - { role: "andrewvy.elixir", action: "setup" }
```


Next, you can create your deploy playbook. Here's my example:


```
---
- hosts: build_servers
  sudo: no
  vars:
    repo_url: git@github.com:CHANGEME
    app_name: CHANGEME
  roles:
    - { role: "andrewvy.elixir", action: "build" }

- hosts: production
  sudo: no
  vars:
    app_name: CHANGEME
  roles:
    - { role: "andrewvy.elixir", action: "start" }
```

But, you can easily do compiling + deploying on the same server.

```
---
- hosts: production
  sudo: no
  vars:
    repo_url: git@github.com:CHANGEME
    app_name: CHANGEME
  roles:
    - { role: "andrewvy.elixir", action: "build" }
    - { role: "andrewvy.elixir", action: "start" }
```

**You must define** a `repo_url` and `app_name`. And you **must** define the build role first.

After you have first successfully deployed with the `action: start`, you can then swap it out for `upgrade` for hot-code upgrades. :)


Actions:
---------

Here are the different actions:

`setup` - will initially setup the server for compiling.

`build` - will compile the release tar, commit and tag it, and push it up.

`start` - will use the compiled release tar, and restart the server with it.

`upgrade` - will use the compiled release tar, and perform hot-code upgrade.


Variables
---------

`repo_url` - (REQUIRED) Used to pull down the repo when compiling.

`app_name` - (REQUIRED) This is what is defined in your *mix.exs* file.

`app_port` - (OPTIONAL) This is what port your app will be exposed as. *Default: 3000*

`git_ref` - (OPTIONAL) This is what branch is pulled when compiling. *Default: master*

`projects_dir` - (OPTIONAL) This is where we temporarily pull down repos and compile releases. *Default: /tmp/projects*

`mix_env` - (OPTIONAL) This is what environment your app will be compiled to. *Default: prod*
