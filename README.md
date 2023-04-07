# 42born2beroot Guide

## Starting Virtual Machine Up:
  1. Install software (Debian, VirtualBox etc)
  2. Once installed, open VB and create a new VM by clicking 'new'
  3. Follow the steps.
  4. Once done, select the VM you just created and click 'Start'.
  5. Onces started, use your arrow keys and select 'Install' (once hover over it, click 'Enter')
  6. Go through the installation procedure.
  7. Log into it via passphrase and login details you created when setting up
  8. Type `lsblk` in your Virtual Machine to see the partition
 
 ## Configuring your Virtual Machine:
  1. Go to the root directory using `su -` and login
  2. Install Sudo `apt install sudo`
  > "Apt" is a command-line package manager used in Debian-based Linux systems. It is short for "Advanced Package Tool" and is used to install,          upgrade, and remove software packages on the system
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
  9. Change the port to `4242` (remember to remove `#`) and `#PermitRootLogin` to  `no` in `sshd_config` file via the command `vim /etc/ssh/sshd_config`
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
  15. Edit the file `vim /etc/login.defs` with the following as per instructions:
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
  ## Crontab and Monitoring script Configuation
   > Crontab is used to automate repetitive tasks, such as system backups, log rotation, and software updates, without requiring manual intervention
   1. Install net-tools `apt-get install -y net-tools`
   2. Then navigate to `/usr/local/bin/`
   3. Create your script file `touch monitoring.sh`
   > You can check with `ls`if it created the file
   4. Give the file the correct executables: `chmod 777 monitoring.sh`
   > The command gives full permissions to the owner, group, and other users, allowing them to read, write, and execute the "monitoring.sh" file
   5. Open the script file via `vim` or `nano`
   6. Add the following:
   ```
   #!/bin/bash
   
   # Get the system arhitecture an kernal version
   ak=$(uname -a)
   
   # Get number of physical processors
   pcpu=$(nproc)
   
   # Get the numbr of virtual processors
   vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)
   
   # Get the curent RAM on server and its utilization rate as a percentage
   tram=$(free -m | awk '$1 == "Mem:" {print $2}')
   uram=$(free -m | awk '$1 == "Mem:" {print $3}')
   pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
   
   # Get current avaliable memory on server an its utilization as a percentage
   tdisk=$(df -BG | grep '^/dev/' | grep -v '/boot$' | awk '{total += $2} END {print total}')
   udisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{used += $3} END {print used}')
   tdisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{total += $2} {used += $3} END {printf("%d"),used/total*100}')
   
   # Get current utilization rate of processors as percentage
   
   
   # Get the date/time of last reboot
   lrb=$(who -b | awk '{print $3, $4}')
   
   # Determine whether or not the LVM is active
   lvm=$(if [ $(lblk | grep "lvm" | wc -l) eq 0 ]; then echo no; else echo yes; fi)
   
   # Get the number of active connections (TCP)
   tcp=$(ss -tuna state established | wc -l)
   
   wall "	#Architecture:		${ak}
	        #CPU physical:		${pcpu}
	        #vCPU:			${vcpu}
	        #Memory Usage:		${uram}/${tram}MB (${pram}%)
          	#Disk Usage:		${udisk}/${tdisk}GB ($p{disk}%)
	        #CPU load:		${}
          	#Last boot:		${lrb}
	        #LVM use:		${lvm}
	        #Connections TCP:	${tcp} 
	        #User log:		${}
	        #Network:		${}
	        #Sudo:			${}
        "
   ```
   ## Explanations:
   ### General commands:
   > 'grep' is used to search for a specified pattern or regular expression in a file or set of files \
   > 
   #### 1. Get the system arhitecture an kernal version
   
   `ak=$(uname -a)`
   > 1. `uname` Used to print system information about the current operating system
   > 2. `-a` Is used to display all information
   #### 2. Get number of physical processors
   
   `pcpu=$(nproc)`
   > 1. `nrpoc` Outputs the number of available processing units to the standard output
   #### 3. Get the numbr of virtual processors
   
   `vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)`
   > 1. `grep "^processor" /proc/cpuinfo`: the `grep` command is used to search for lines in the `/proc/cpuinfo` file that start with the string "processor", which is used to denote each CPU thread on the system.
   > 2. `wc -l`: the output of grep is piped to the `wc` (word count) command with the `-l` option, which counts the number of lines in the output

  #### 4. Get the curent RAM on server andd its utilization rate as a percentage
   ```
   tram=$(free -m | awk '$1 == "Mem:" {print $2}')
   uram=$(free -m | awk '$1 == "Mem:" {print $3}')
   pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
   ```
   > 1.`free -m`: the free command is used to display information about the memory usage on the system, including the total amount of memory available, used, and free. The -m option is used to display the memory usage in megabytes. \
   > 2. `awk '$1 == "Mem:" {print $2}'`: the output of `free -m` is piped to the `awk` command, which is used to search for the line that starts with the string "Mem:" and print the second field \
   > 3. `awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}'`: The output of `free` is piped to the `awk` command, which is used to search for the line that starts with the string "Mem:" and print the percentage of memory used. This is calculated by dividing the third field (memory used) by the second field (total memory), multiplying the result by 100, and formatting it to two decimal places using the printf function.

  #### 5. Get current avaliable memory on server an its utilization as a percentage
   ```
   tdisk=$(df -BG | grep '^/dev/' | grep -v '/boot$' | awk '{total += $2} END {print total}')
   udisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{used += $3} END {print used}')
   tdisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{total += $2} {used += $3} END {printf("%d"),used/total*100}')
   ```
  #### 6. Get current utilization rate of processors as percentage
  
  #### 7. Get the date/time of last reboot
  `lrb=$(who -b | awk '{print $3, $4}')`
  > 1. `who -b`: the who command is used to display information about the currently logged in users on the system. The `-b` option is used to display information about the system boot time 
  > 2. `awk '{print $3, $4}'`: the output of who `-b` is piped to the `awk` command, which is used to print the third and fourth fields of the output. These fields represent the date and time of the system boot


#### 8.Determine whether or not the LVM is active
   `lvm=$(if [ $(lblk | grep "lvm" | wc -l) eq 0 ]; then echo no; else echo yes; fi)`
   > 1. `lsblk`: the lsblk command is used to display information about the block devices on the system, including disks and partitions 
   > 2. `grep "lvm"`: the output of lsblk is piped to the grep command, which is used to search for lines containing the string "lvm". If LVM is active on the system, there will be one or more lines containing this string 
   > 3. `wc -l`: the output of grep is piped to the wc command with the `-l` option, which counts the number of lines in the output. If LVM is active on the system, there will be one or more lines containing the string "lvm", resulting in a non-zero line count 
   > 4. `if [ $(lblk | grep "lvm" | wc -l) eq 0 ]; then echo no; else echo yes; fi`: this is an if statement that checks whether the line count from the previous command is equal to zero. If the line count is zero, then LVM is not active on the system, and the script echoes "no" using the echo command. If the line count is non-zero, then LVM is active on the system, and the script echoes "yes" using the echo command.


#### 9. Get the number of active connections (TCP)
   `tcp=$(ss -tuna state established | wc -l)`
   > 1. `ss`: is used to display information about active network connections 
   > 2. `-tuna`: specifies the options to the ss command to show only TCP (-t) and UDP (-u) connections in numeric format (-n) and all listening and non-listening sockets (-a) 
   > 3. `|`: pipes the output of the ss command to the next command 
   > 4. `wc`: count lines, words, and characters in the input 
   > 5. `-l` specifies the option to the wc command to count only the number of lines in the input.
   
   
