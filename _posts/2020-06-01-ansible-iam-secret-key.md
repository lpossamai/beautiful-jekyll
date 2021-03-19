---
layout: post
title: Creating AWS IAM Access and Secret Access key with Ansible
---

As per May 2020, the IAM Ansible module allows you to create the [AWS IAM Access Key](https://docs.ansible.com/ansible/latest/modules/iam_module.html#parameter-access_key_state), but unfortunately, [it does not allow you to retrieve the nearly created Secret Access Key](https://stackoverflow.com/questions/61623654/creating-iam-users-with-ansible-getting-the-cli-credentials/61624771#61624771). I've created a [feature request](https://github.com/ansible-collections/amazon.aws/issues/47) already.

I was trying to automate the creation of my IAM users and had to come up with a workaround.

You can [check the Github Repository here](https://github.com/lpossamai/ansible-iam-cli-keys).

### Ansible IAM module
As per May 2020, the IAM Ansible module allows you to create the AWS IAM Access Key, but unfortunately, it does not allow you to retrive the nearly created Secret Access Key.

I was trying to automate the creation of my IAM users and had to come up with a workaround.

### This repository
Using this provided code, you'll be able to:

- Creates IAM Users and their Access and Secret Access Keys for CLI usage
- Creates FullAdmin (`AdministratorAccess`) and ReadOnly (`ReadOnlyAccess`) groups for IAM users
- Creates and applies the [Force MFA policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_users-self-manage-mfa-and-creds.html) to the groups
- Creates and applies an `IAM Password Policy`

> This is a work in progress

### USAGE
1. Set the `AWS_PROFILE` in your environment - I'm using Arch Linux here: `export AWS_PROFILE=test-profile`
2. Clone the repository
3. Change the variables `vars/user-list.yml`
4. Update the usernames in the groups for `task/create-group.yml`

**Note**: The password to access the Console will be saved in the `passwordfile` file.