+++
title = "On AWS, no need for SSH"
date = 2023-01-03
[taxonomies] 
tags=["TIL"]
+++

## AWS SSM and SSH

There’s a way to get most of the goodness of SSH in AWS **without opening ports, using public IP addresses, and modifying security groups or ACLs**.
This is also a way of executing scripts directly against an instance, and of [SSH tunneling][tunneling] via AWS SSM.

### SSM

Starting an SSM session on the command line is straightforward:

```bash
aws ssm start-session --target i-0662da266763d5ea0
```

However, you can use SSH the old fashioned way...

### SSH over SSM

Add this to your SSH config ($HOME/.ssh/config):

```bash
# SSH over Session Manager
host i-* mi-*
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```

This allows the usual SSH invocation:

```bash
ssh -i ~/.ssh/jump.pem ubuntu@i-0662da266763d5ea0
```

This allows running a script on the remote AWS instance like:

```bash
ssh -i ~/.ssh/jump.pem ubuntu@i-0662da266763d5ea0 'bash -s' < /tmp/something.sh
```

You can copy files to/from the machine:

```bash
scp -i ~/.ssh/jump.pem ubuntu@i-0662da266763d5ea0:folder/something.json /tmp/something.json
```

Or better:

```bash
rsync -e 'ssh -i ~/.ssh/jump.pem' ubuntu@i-0662da266763d5ea0:folder/something.json /tmp/something.json
```

### References

- [Installing][plugin] the SSM plugin into the AWS CLI
- [Getting started][getting-started] SSH over SSM
- SSH [tunneling][tunneling] via SSM


[tunneling]: https://aws.amazon.com/premiumsupport/knowledge-center/systems-manager-ssh-vpc-resources/
[plugin]: https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html
[getting-started]: https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started-enable-ssh-connections.html