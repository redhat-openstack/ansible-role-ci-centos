ansible-role-ci-centos
----------------------
This role wraps around the Ansible module provided by python-cicoclient_ to
request and release bare metal servers from the `ci.centos.org`_ system, Duffy_.

.. _python-cicoclient: https://github.com/dmsimard/python-cicoclient
.. _ci.centos.org: https://ci.centos.org/
.. _Duffy: https://wiki.centos.org/QaWiki/CI/Duffy

Role variables
~~~~~~~~~~~~~~

For default values, see `defaults/main.yml`_

* ``api_key``: **(Required)** API Key to the ci.centos.org provisioning system
  (defaults to ``CICO_API_KEY`` environment variable)
* ``private_key``: **(Required)** Path to the SSH key to log on to ci.centos.org CI nodes
  (defaults to ``CICO_SSH_KEY`` environment variable)
* ``endpoint``: API endpoint for the ci.centos.org Duffy provisioning system
* ``arch``: Architecture of the requested nodes (ex: x86_64)
* ``release``: Release of the requested nodes (ex: 7)
* ``count``: Amount of nodes requested
* ``ci_environment``: Key that is set for traceability that execution is driven by a CI environment
* ``rsync_server``: rsync server to upload job artifacts to
* ``rsync_password``: Password to the artifact rsync server (usually determined from ``api_key``)
* ``jenkins_url``: URL of the Jenkins instance (provided by Jenkins as an environment variable)
* ``build_url``: URL of the Jenkins build (provided by Jenkins as an environment variable)
* ``job_name``: Name of the Jenkins job (provided by Jenkins as an environment variable)
* ``build_number``: Number of the Jenkins build (provided by Jenkins as an environment variable)
* ``log_root``: Root directory where the debug logs are located (so we can upload them)
* ``log_destination``: Subdirectory where the debug logs are located (so we can upload them)

.. _defaults/main.yml: https://github.com/redhat-openstack/ansible-role-ci-centos/blob/master/defaults/main.yml

Included tasks
~~~~~~~~~~~~~~

* ``provision``: Requests a node of the provided specifications
* ``release``: Release the nodes tied to the current session

Example playbook
~~~~~~~~~~~~~~~~

.. code-block:: yaml

    - name: Provision a ci.centos.org node
      hosts: localhost
      roles:
        - { role: "ci-centos", action: "provision" }

    # Do something with it

    - name: Release the provisioned ci.centos.org nodes
      hosts: openstack_nodes
      roles:
        - { role: "ci-centos", action: "release" }

Important notes
~~~~~~~~~~~~~~~

The ci.centos.org provisioning system may only be used from within their network infrastructure.
This means you can only run this role on, for example, a Jenkins slave that was provided by their team.

If you provision a node, you need to destroy it in the same playbook or it will leak.
This means you have to release it as the last step of the playbook (as shown above) **AND** handle errors in your playbooks.

WeIRDO uses this role and handles errors with the block_ feature from Ansible 2.
The implementation looks like this:

- Catch errors: https://github.com/redhat-openstack/ansible-role-weirdo-packstack/blob/6869dcd80f34c7c5be679404795d0a2f9964fd27/tasks/run.yml#L19-L34
- Notify a handler to release the CI node: https://github.com/redhat-openstack/ansible-role-weirdo-common/blob/c865feb873817f08eed71858b15571214bbd6e00/tasks/rescue.yml#L24
- Include the release task from ci-centos: https://github.com/redhat-openstack/ansible-role-weirdo-common/blob/c865feb873817f08eed71858b15571214bbd6e00/handlers/main.yml#L31-L33
- Actually release the CI node: https://github.com/redhat-openstack/ansible-role-ci-centos/blob/8f28b723e2d80b66e8e9e52c7fa5c25421d789d8/tasks/release.yml

There is an issue_ opened with Ansible to try and improve how rescue blocks can be use to make this use case far simpler.

.. _block: http://docs.ansible.com/ansible/playbooks_blocks.html
.. _WeIRDO: https://github.com/redhat-openstack/weirdo
.. _issue: https://github.com/ansible/ansible/issues/13587