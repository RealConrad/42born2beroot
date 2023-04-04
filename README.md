# 42born2beroot Guide

## Starting Virtual Machine Up:
  1. Install software (Debian, VirtualBox etc)
  2. Setup OS on Virtual Machine
  3. Log into it via passphrase and login details you created when setting up
  4. Type `lsblk` in your Virtual Machine to see the partition
 
 ## Configuring your Virtual Machine:
  1. Go to the root directory using `su -` and login
  2. Install Sudo `apt install sudo`
  3. Add a group called `user42` using `groupadd`
  4. Add the user to the newly created group (adds user to 2 groups) `usermod -aG user42,sudo cwenz`
  > a = append user to group \
  > g = the group to which the user gets added to \
  > To remove a user from a group: `deluser ${user} ${group}
  6. To view if the user was added: `getent group user42`
  7. Install `Git` using `apt-get install git -y`
  > You should see a version number if its installed correctly by using: `git --version`
  8. Install SSH via `sudo apt install openssh-server`
  > Check to see if it installed correctly: `systemctl status ssh`
  9. Change the `#port` to `4242` (remember to remove `#`) and `#PermitRootLogin` to  `no` in `sshd_config` file via the command `vim /etc/ssh/sshd_config`
  > Once in VIM, press 'i' to go into 'insert mode' \
  > To exit 'Insest mode' press 'ESC' and to save and quit `:wq` \
  > Type `sudo grep Port /etc/ssh/sshd_config` to check if the port settings are correct, it should be `Port 4242`
  10. Restart the ssh server via `systemctl restart ssh`
