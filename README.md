Arteria-provisioning
--------------------

Note that this is not yet production ready, and should only be used for development purposes at the moment.

*Getting started*

Run the script ./dev-dependencies to set up required dependencies and validate your setup. It does the following:
* Installs the hostmanager plugin for Vagrant
  * This will manage the host files of the VMs and your hosting machine. Run `vagrant hostmanager` to update the hosts files.
* Checks out other arteria-project related repositories one directory up (i.e: ../arteria-core).
* Generates an ssh-key (without a password) that will be used to establish password less ssh between the hosts.
* Validates that you're running the expected version of Ansible (1.9.3)

Running `vagrant up` will produce a virtual machine which runs StackStorm, including the WebUI, listening on `stackstorm-master:8080`. Login with `user=testu password=testp`.

Vagrant also setups a virtual machine `testarteria1` where the various Arteria services gets installed. Note that the memory requirement in the Vagrantfile for this machine is at the moment mainly for the bcl2fastq service, so if you don't have enough memory on your machine, and you just want to be able to install it, then please go ahead and decrease the required memory.

The playbooks here are forked from: https://github.com/StackStorm/ansible-st2. In the future we'll probably move to
running the StackStorm services in Docker containers, and at that point this will change a lot, but for now this is
a working solution.

*Configuring mail sending*

There is a sample configuration for Postfix bundled, that relays mail via Gmail. This needs to be manually adopted and changed by the user for her specific needs, if she wants Stackstorm to be able to send notification emails when an error in the workflow has occurred.

Edit `/etc/postfix/sasl/sasl_passwd` on `stackstorm-master`. After saving run `postmap /etc/postfix/sasl/sasl_passwd`. Login to your Google account and go to https://myaccount.google.com. Navigate to /Sign-in & security -> Connected apps & sites/, and enable there the option /Allow less secure apps/.

Now verify that the email configuration works with e.g. `echo arteriatest | mail -s arteriatest user@host`, where `user@host` is the recipient address you want to try mailing. If it doesn't seem to work then an inspection of the log file `/var/log/mail.log` together with the Postfix manual is recommended.

*The arteria system*

The vagrant setup and ansible playbook here demonstrates a simple usage scenario where we have one central StackStorm
master (`stackstorm-mater`) which runs all the StackStorm services, and a node (`testarteria1`) which runs the different
 arteria services. This illustrates a scenario where data arrives at one processing machine (and get's picked up by the
 runfolder-ws), blc2fastq-ws can then be used to run Illumina bcl2fastq on this runfolder, and finally siswrap-ws can
  be used to run Sisyphus scripts on it for quality control, etc. Other deployment scenarios are of course also possible,
  but here we have chosen to demonstrate the simplest possible case.

On `testarteria1` the three services runfolder-ws, bcl2fastq-ws and siswrap-ws will start-up controlled by supervisord.
They will be available on the following ports:

    siswrap-ws -> 10700
    runfolder-ws -> 10800
    bcl2fastq-ws -> 10900

They all have a api-documentation which will be available under `testarteria1:<port>/api` so that you can e.g. curl
in this way:

    curl http://testarteria1:10700/api
    curl http://testarteria1:10800/api
    curl http://testarteria1:10900/api

to get current api documentation.

You can also play-around with triggering the workflows from StackStorm and in general get a feel for the system.

*More information*
More information on how to deploy Arteria for different scenarios will be available on: http://arteria-project.github.io
in the future.

*Synching with ansible-st2*
The folder ansible-st is a copy of https://github.com/stackstorm/ansible-st2. When getting latest from upstream
the directory ./ansible-st2 should be removed entirely and replaced with a copy of the repo maintained by stackstorm.
Then run ./ansible-st2-local/symlink to update with our additions.

*Fetch sources from other repos*

NOTE: Currently the repos are only configurable for the runfolder service. All other services are fetched
from the main git repos, https://github.com/arteria-project/*.git

You can fetch from different repos by overriding the following parameters:

| Variable            | Default       | Description  |
| ------------------- | ------------- | ----- |
| `arteria_runfolder_sources` | git | If set to git, the sources will be downloaded from the configured repos. If set to mount, the sources will instead be fetched from /arteria/*
| `arteria_runfolder_repo` | `https://github.com/withrocks/arteria-runfolder.git` |
| `arteria_runfolder_version` | `HEAD` | Git tag, branch name or a commit id
| `arteria_runfolder_core_repo`| `https://github.com/arteria-project/arteria-core.git` | NOTE: Will be discontinued, using only the requirements file of the library using it
| `arteria_runfolder_core_version`   | `HEAD` | Git tag, branch name or a commit id

*Validate*

For a basic smoke test for everything, run ./validate-setup. It will check if all expected st2 processes are running and listening on the expected ports.

