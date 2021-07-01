# Using Ansible SSH Connection through The Bastion

The three scripts in this directory are a wrapper around Ansible native SSH
connection, so that [The Bastion](https://github.com/ovh/the-bastion/) can be transparently used along with Ansible.
You have to set some os SSH Ansible variables as defined in
https://docs.ansible.com/ansible/latest/plugins/connection/ssh.html in addition
with `BASTION_USER`, `BASTION_PORT` and `BASTION_HOST`. It can also rely on
`ansible-inventory` to identify `bastion_user`, `bastion_host`, `bastion_port`.
`ansible-inventory` takes precedences over environment variables as this will
allow to use different bastion for different hosts.

## Note of the guy who forked the original repository (aka me)

Unfortunately I was unable to get it working with the original docs and I have yet to determine why.


Here are a few ideas to help you debug, if you're stuck at the same place:

### python3

I have changed the shebang line of each python script to `#!/usr/bin/env python3`.

The reason for this was Ubuntu 20.04 not symlinking `python` to `python3`.

Fortunately the code just works under python3 without any modification (assuming it was written for python2 (please don't hurt me, if I am wrong)).

### ANSIBLE_* environment variables

I also exported the variables listed in the original docs, but added `ANSIBLE_PYTHON_INTERPRETER`.

My variables look like this:

```bash
export BASTION_USER="mybastionusername"
export BASTION_HOST="bastion.example.net"
export BASTION_PORT=22
export ANSIBLE_PIPELINING=1
export ANSIBLE_SCP_IF_SSH="True"
export ANSIBLE_PRIVATE_KEY_FILE="${HOME}/.ssh/id_rsa"
export ANSIBLE_SSH_EXECUTABLE="${THE_LOCATION_WHERE_YOU_CLONED_IT}/the-bastion-ansible-wrapper-python3/sshwrapper.py"
export ANSIBLE_SCP_EXECUTABLE="${THE_LOCATION_WHERE_YOU_CLONED_IT}/the-bastion-ansible-wrapper-python3/scpbastion.sh"
export ANSIBLE_PYTHON_INTERPRETER="/usr/bin/python3"
```

It seems like using the bastion in the middle kinda breaks the Python interpreter discovery of Ansible.

So I set `ANSIBLE_PYTHON_INTERPRETER` as a workaround.

My error looked like this (if you want to compare):

```S
TASK [Gathering Facts] *********************************************************
[WARNING]: Unhandled error in Python interpreter discovery for host test.example.net: unexpected output from Python interpreter
discovery
fatal: [test.example.net]: FAILED! => {"ansible_facts": {}, "changed": false, "failed_modules": {"ansible.legacy.setup": {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "failed": true, "module_stderr": "/bin/sh: 1: /usr/bin/python: not found\n", "module_stdout": "", "msg": "The module failed to execute correctly, you probably need to set the interpreter.\nSee stdout/stderr for the exact error", "rc": 127, "warnings": ["Platform unknown on host test.example.net is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information."]}}, "msg": "The following modules failed to execute: ansible.legacy.setup\n"}
```

The docs linked in the error message didn't really got me anywhere but setting the interpreter path.

It's working for me now and I only used the environment variables listed above.

If you feel like giving it a try, I think it's worth.

[...]

# License

Copyright OVH SAS

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
