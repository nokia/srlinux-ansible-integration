# SR Linux Ansible Collection

Ansible Collection to manage Nokia SR Linux devices. Documentation is provided at <https://learn.srlinux.dev/ansible/collection/>

## Quick start for developers

Start with cloning the repo:

```bash
git clone git@github.com:nokia/srlinux-ansible-integration.git
cd srlinux-ansible-integration
```

Deploy the lab to support the tests:

```bash
./run.sh deploy-lab
```

Run the automated suite of tests to make sure nothing is missing. This will also prepare a dev environment (you have to make sure the venv with ansible is activated or ansible-playbook is in your path):

```bash
./run.sh test
```

To validate that the code passes Ansible's sanity check, run:

```bash
./run.sh sanity-test
```

### Running individual tests

To run an individual test, first make sure that the local code base is used by ansible. This can be done by running:

```bash
./run.sh prepare-dev-env
```

Then either run one of the provided test playbooks (defined in [run.sh](run.sh)) or run your own:

```bash
./run.sh test-set-leaves
```

## Contributing to this collection

The content of this collection is made by people like you, a community of individuals collaborating on making the world better through developing automation software.

We are actively accepting new contributors and all types of contributions are very welcome.

Don't know how to start? Refer to the [Ansible community guide](https://docs.ansible.com/ansible/devel/community/index.html)!

Want to submit code changes? Take a look at the [Quick-start development guide](https://docs.ansible.com/ansible/devel/community/create_pr_quick_start.html).

We also use the following guidelines:

* [Collection review checklist](https://docs.ansible.com/ansible/devel/community/collection_contributors/collection_reviewing.html)
* [Ansible development guide](https://docs.ansible.com/ansible/devel/dev_guide/index.html)
* [Ansible collection development guide](https://docs.ansible.com/ansible/devel/dev_guide/developing_collections.html#contributing-to-collections)

## Code of Conduct

We follow the [Ansible Code of Conduct](https://docs.ansible.com/ansible/devel/community/code_of_conduct.html) in all our interactions within this project.

If you encounter abusive behavior, please refer to the [policy violations](https://docs.ansible.com/ansible/devel/community/code_of_conduct.html#policy-violations) section of the Code for information on how to raise a complaint.

## Communication

Join the Ansible forum:

* [Get Help](https://forum.ansible.com/c/help/6): get help or help others. Please add appropriate tags if you start new discussions.
* [Posts tagged with 'srlinux'](https://forum.ansible.com/tag/srlinux): subscribe to participate in SR Linux Ansible collection/technology-related conversations.
* [Social Spaces](https://forum.ansible.com/c/chat/4): gather and interact with fellow enthusiasts.
* [News & Announcements](https://forum.ansible.com/c/news/5): track project-wide announcements including social events. The [Bullhorn newsletter](https://docs.ansible.com/ansible/devel/community/communication.html#the-bullhorn), which is used to announce releases and important changes, can also be found here.

## Licensing

BSD 3-Clause License

See [LICENSE](LICENSE) to see the full text.
