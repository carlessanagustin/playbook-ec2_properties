# Onto the Playbook

Now that we have the back story out of the way, let's get into the playbooks. Several attendees asked how to spin up multiple ec2 instances, all with differing tags.

Extrapolating from that question the user wants/concerns are:

The ec2 doesn't "count" (spins up multiple identical instances)
Run tasks/plays against spun up instances (obviously)
Assign different properties to each instance (i.e. tags)
From the above requirements I will demonstrate a general design that can be applied to (1) other cloud modules host bring ups with (2) associated per-host attributes (i.e. tags, security groups, etc).

Note: Ensure that environment variables AWS_ACCESS_KEY and AWS_SECRET_KEY are set and that you have the Boto python library installed.

Playbook time!
https://github.com/chrismeyersfsu/playbook-ec2_properties

```
├── README.md
├── inherit.yml    <-- showcase add_host
├── inv_trick.yml  <-- using dummy inventory entries
├── inventory      <-- [ec2_instances] dummy group to be filled in with ec2 instances
└── new_group.yml  <-- express ec2 instances and tags w/ variables
```

```
ansible-playbook -i inventory inherit.yml
```

inherit.yml is equivalent to a "hello world" for this design. You can play around with hostvars and add_host without the time consuming part of spinning up ec2 instances and without the side effect of creating too many instances in ec2. It uses the special Ansible variable play_hosts to execute the add_host task for each play in the playbook and accesses variables associated with the play, defined in the inventory file.

Now let's throw ec2 in the mix.

```
ansible-playbook -i inventory inv_trick.yml
ansible-playbook -i inventory new_group.yml
```

Both inv_trick.yml and new_group.yml accomplish the same thing using different designs.

new_group.yml defines an array of dictionaries that is looped over to start ec2 instances. The dictionary entries contain attributes that are "forwarded" to the ec2 module.

inv_trick.yml uses host variables from the ec2_instances group defined in the inventory file and "forwards" them to the ec2 module.

Both designs create a dynamic_group in the playbook from the ec2 results. The final play in the playbook then runs the ping task to demonstrate that tasks can be performed on the newly created dynamic_group.

from: https://www.ansible.com/blog/ansible-ec2-tags
