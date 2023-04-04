# 42born2beroot Guide

## Starting Virtual Machine Up:
  1. Install software (Debian, VirtualBox etc).
  2. Setup OS on Virtual Machine.
  3. Log into it via passphrase and login details you created when setting up.
  4. Type `lsblk` in your Virtual Machine to see the partition.
 
 ## Configuring your Virtual Machine:
  1. Go to the root directory using `su -` and login.
  2. Install Sudo `apt install sudo`.
  3. Add a group called `user42`.
  4. After installing sudo, add the user to the group “usermod -aG sudo cwenz”
  > a = append user to group 
  g = the group to which the user gets added to 
  To remove a user from a group: `deluser ${user} ${group}
  5. To view if the user was added: `getent group user42`
