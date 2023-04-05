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
  > The `-y` tag means that you agree to all prompts that you would normally have to manually agree to \
  > You should see a version number if its installed correctly by using: `git --version`
  8. Install SSH via `sudo apt install openssh-server`
  > Check to see if it installed correctly: `systemctl status ssh`
  9. Change the `#port` to `4242` (remember to remove `#`) and `#PermitRootLogin` to  `no` in `sshd_config` file via the command `vim /etc/ssh/sshd_config`
  > Once in VIM, press 'i' to go into 'insert mode' \
  > To exit 'Insest mode' press 'ESC' and to save and quit `:wq` \
  > Type `sudo grep Port /etc/ssh/sshd_config` to check if the port settings are correct, it should be `Port 4242`
  10. Restart the ssh server via `systemctl restart ssh`
  > Check the status via `systemctl ssh status`
  11. Now you can connect your own PC's terminal to the VM. Run `ssh username@127.0.0.1 -p 4242`
  > If you get any errors, try: `rm ~/.ssh/known_hosts` and try again
  12. Run `exit` to close your connection
  13. Edit the file via `vim /etc/security/pwquality.conf` according to the subject
  14. Include the following at the file `vim /etc/pam.d/common-password`:
  > \# Words are spaces with tabs \
  > password   requisite   pam_pwquality.so \
  > The above takes the changes at `pwquality.conf` and applies it to the password policy
  15. Edit the file `vim /etc/login.defs`
  > PASS_MAX_DAYS 30 \
  > PASS_MIN_DAYS 2 \
  > PASS_WARN_AGE 7
  16. Rebott for the changes to take effect: `sudo reboot`
  17. this change only takes effect for newly created users. You will have to manually change it for the existing user via:
  > \# -m for PASS_MIN_DAYS -M for PASS_MAX_DAYS and -W for PASS_WARN_AGE
  > chage -m 2 -M 30 -W 7 user42 \
  > chage -m 2 -M 30 -W 7 root \
  > \# Check if rules have been applied. \
  > chage -l user42 \
  > chage -l root
 ## Groups
  > to view all users: `cut -d: -f1 /etc/passwd` \
  > To add a new user: `sudo adduser new_username`\
  > To view which groups your user is part of: `groups`
 ## Configure Sudo Policy
  1. Run `visudo`
  2. Add the following:
  ```
  # Resets terminal environment to remove any user variable.
  Defaults	env_reset
  # Sends a mail of bad sudo password attempts.
  Defaults	mail_badpass
  # Secure paths for the sudo user.
  Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/bin:/sbin:/bin"
  # Message because of entering a wrong password.
  Defaults	badpass_message="Computer says no."
  # Max password tried when using sudo.
  Defaults	passwd_tries=3
  # Defining a logfile for commands used with sudo.
  Defaults	logfile="/var/log/sudo.log"
  # Define that input and output should be locked.
  Defaults	log_input, log_output
  # Requires the user to be logged into a terminal to run the sudo command.
  Defaults	requiretty

  # User privilege specifications (add your user)
  user42  ALL=(ALL:ALL) ALL
  ```
  ## Crontab Configuation
   > Crontab is used to automate repetitive tasks, such as system backups, log rotation, and software updates, without requiring manual intervention
   1. Install net-tools `apt-get install -y net-tools`
   2. Then navigate to `/usr/local/bin/`
   3. Create your script file `touch monitoring.sh`
   > You can check with `ls`if it created the file
   4. Give the file the correct executables: `chmod 777 monitoring.sh`
   > The command gives full permissions to the owner, group, and other users, allowing them to read, write, and execute the "monitoring.sh" file
   
