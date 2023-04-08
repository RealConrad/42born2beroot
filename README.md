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
  16. Reboot for the changes to take effect: `sudo reboot`
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
  # Clears out any variables that the user may have set in the terminal environment to start fresh
  Defaults	env_reset
  # Sends a mail of bad sudo password attempts
  Defaults	mail_badpass
  # Secure paths for the sudo user
  Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/bin:/sbin:/bin"
  # Message when entering a wrong password
  Defaults	badpass_message="Incorrect password"
  # Max number of password attempts
  Defaults	passwd_tries=3
  # Defining a to store sudo commands
  Defaults	logfile="/var/log/sudo.log"
  # Define that input and output should be locked
  Defaults	log_input, log_output
  # Requires the user to be logged into a terminal to run the sudo command
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
   cpu_per=$(top -bn1 | grep '^$Cpu' | cut -c 9- | xargs | awk '{print("%.1f"), $1 + $3')
   
   # Get the date/time of last reboot
   lrb=$(who -b | awk '{print $3, $4}')
   
   # Determine whether or not the LVM is active
   lvm=$(if [ $(lblk | grep "lvm" | wc -l) eq 0 ]; then echo no; else echo yes; fi)
   
   # Get the number of active connections (TCP)
   tcp=$(ss -tuna state established | wc -l)
   
   # Get the number of users using the server
   users=$(who | wc -l)
   
   # Get the IPv4 and MAC address
   ipv4=$(hostname -I)
   mac=$(ip link show | grep "ether" | awk '{print $2}')
   
   # Get the number of commands executed with the sudo program
   scmd=$(journalctl _COMM=sudo | grep "COMMAND" | wc -l)
   
   wall "	#Architecture:		${ak}
	  	#CPU physical:		${pcpu}
	  	#vCPU:			${vcpu}
	 	#Memory Usage:		${uram}/${tram}MB (${pram}%)
        	#Disk Usage:		${udisk}/${tdisk}GB ($p{disk}%)
	   	#CPU load:		${cpu_per}
          	#Last boot:		${lrb}
		#LVM use:		${lvm}
		#Connections TCP:	${tcp} 
		#User log:		${users}
		#Network:		IP ${ipv4} ${mac}
		#Sudo:			${scmd}
        "
   ```
   ## Explanations:
   #### General commands:
   >  1. `grep` is used to search for a specified pattern or regular expression in a file or set of files 
   > 2. `awk` reads each line of the input file(s) and applies the specified pattern and action(s) to each line that matches the pattern. \
   > Basic sytax is: `awk 'pattern' { action }' file`
   #### 1. Get the system architecture and kernal version
   
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
  `cpu_per=$(top -bn1 | grep '^$Cpu' | cut -c 9- | xargs | awk '{print("%.1f"), $1 + $3')`
  > 1. `top -bn1`: the top command is used to display real-time information about the system's processes and resource usage. The -b option is used to run top in batch mode, and the -n1 option is used to run it for a single iteration, without continuously updating the display 
  > 2. `grep '^%Cpu'`: the output of top is piped to the `grep` command, which is used to filter the output to only display the lines starting with %Cpu 
  > 3. `cut -c 9-`: the output of grep is piped to the cut command, which is used to remove the first eight characters of each line. These characters contain the %Cpu(s): label, which is not needed for the calculation 
  > 4. `xargs`: the output of cut is piped to the xargs command, which is used to remove any leading or trailing whitespace characters 
  > 5. `awk '{print("%.1f"), $1 + $3}'`: the output of `xargs` is piped to the `awk` command, which is used to calculate the current CPU utilization as a percentage. The print function is used to print the result, formatted to one decimal place, and the `$1 + $3` expression is used to add the values of the CPU utilization percentages for user and system processes
  
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
   
   #### 10. Get the number of users using the server
   `users=$(who | wc -l)`
   > 1. `who`: the `who` command is used to display information about the currently logged-in users on the system 
   > 2. ` wc -l`: Counts the number of lines of the ouput of `who`

   #### 11. Get the IPv4 and MAC address
   ```
   ipv4=$(hostname -I)
   mac=$(ip link show | grep "ether" | awk '{print $2}')
   ```
   > 1. `ipv4=$(hostname -I)`: the `hostname` command is used to retrieve the hostname of the system, and the -I option is used to retrieve the IP address(es) assigned to the system's network interfaces 
   > 2. `mac=$(ip link show | grep "ether" | awk '{print $2}')`: the `ip` command is used to display information about the network interfaces on the system, including their MAC addresses. The `link show` subcommand is used to show the link-layer information for all interfaces, and the output is piped to the `grep` command to filter out all lines except those containing the string "ether". The resulting output is then piped to the `awk` command, which is used to print the second field of each line


   #### 12. Get the number of commands executed by the sudo program
   `scmd=$(journalctl _COMM=sudo | grep "COMMAND" | wc -l)`
   > 1. `journalctl _COMM=sudo`: the `journalctl` command is used to query the system journal for logs. The `_COMM` option is used to filter the logs by the specified process name, which in this case is "sudo" 
   > 2. `grep "COMMAND"`: the output of journalctl is piped to the `grep` command, which is used to filter the output to only display logs containing the string "COMMAND" 
   > 3. `wc -l`: the output of grep is piped to the wc command with the `-l` option, which counts the number of lines in the output

## Crontab setup
1. Exit out your iTerm and go back to your virutal machine (if not already)
2. Type `sudo visudo` to open your sudoers file
3. Add in this line `your_username ALL=(ALL) NOPASSWD: /usr/local/bin/monitoring.sh` under where its written `%sudo ALL=(ALL:ALL) ALL`
4. Save and exit
5. Reboot sudo with `sudo reboot`
6. Run `sudo /usr/local/bin/monitoring.sh` to execute your script
7. Type `sudo crontab -u root -e` to open the crontab and add the rule `*/10 * * * * /usr/local/bin/monitoring.sh` at the bottom of the file
> The above rule means that the script will run every 10 minutes \
> crontab takes 5 inputs: `minutes, hour, day (month), month, and day (week)`

## Signiture file setup
1. Open iTerm and navigate to where you saved your VM
2. Once your in the correct directory type `shasum {name}.vdi`
> Replace `name` with whatever you named your VM
3. Copy the output number and create a signature.txt file and paste that number in the file.
4. Push that file to the repo.


# Evaluation
#### Why did I choose Debian?
Easier to install and setup. PDF reccommended it

#### Difference Debian and CentOS
1. Package Management: Rocky Linux uses the `dnf` package manager, which is a newer package management tool that is compatible with the RHEL and Fedora distributions. Debian, on the other hand, uses the `apt` package manager, which is a well-established tool that has been used by Debian and its derivatives for many years.
2. Default Configuration: Rocky Linux is configured with a minimal set of packages, which makes it more suitable for server installations. Debian, on the other hand, includes a more complete set of packages by default, which makes it more suitable for desktop installations.

#### What is a Virtual Machine?
Computer in a computer essentially. Its a resource that uses software instead of a physically computer to run programs. Works by the VM borrowing resources from the main PC. You can use a VM to test applications/software in a safe environment as it wont affect your main PC.

#### What is the difference between aptitude and APT (Advanced Packaging Tool)?
1. Aptitude is a high-level package manager while APT is lower level which can be used by other higher level package managers
Aptitude is smarter and will automatically remove unused packages or suggest installation of dependent packages
2. Apt will only do explicitly what it is told to do in the command line

#### What is AppArmor?
Linux security system that provides Mandatory Access Control (MAC) security. Allows the system admin to restrict the actions that processes can perform. It is included by default with Debian. Run aa-status to check if it is running.

#### Password Rules
For the password rules, we use the password quality checking library and there are two files the `common-password` file which sets the rules like upper and lower case characters, duplicate characters, digits etc.. and the `login.defs` file which stores the password expiration rules (30 days etc). `Sudo nano /etc/login.defs` `Sudo nano /etc/pam.d/common-password`

#### What is LVM
1. Logical Volume Manager – allows us to easily manipulate the partitions or logical volume on a storage device.
2. Partition is a logical section of a hard disk drive or other storage device that is created to store data. 
3. A logical volume, on the other hand, is a virtualized partition that is created from one or more physical partitions or disks.

#### UFW (Uncomplicated Firewall)
UFW is a interface to modify the firewall of the device without compromising security. You use it to configure which ports to allow/close connections. This is useful in conjunction with SSH, can set a specific port for it to work with.

#### What is SSH?
SSH (Secure Shell) is a secure method of communication between a client and a host that encrypts all communication to ensure data is transmitted securely.

#### What is Cron?
Cron or cron job is a command line utility to schedule commands or scripts to happen at specific intervals or a specific time each day. Useful if you want to set your server to restart at a specific time each day.

> `cd /usr/local/bin` – to show `monitoring.sh` \
> `sudo crontab -u root -e` – to edit the cron job \
> change script to `*/1 * * * * sleep 30s && script path` – to run it every 30 seconds, delete the line to stop the job from running.

#### Evaluation Commands for UFW, Group, Host, lsblk and SSH
`sudo ufw status` - Get firewall status
`sudo systemctl status ssh` - Used to check if ssh is running, any errors/warnings and info about service
`getent group sudo` - Gets the group `sudo`
`getent group user42`
`sudo adduser new username`
`sudo groupadd groupname`
`sudo usermod -aG groupname username` - Adds a user to a group
`sudo chage -l username` - check password expire rules
`hostnamectl` - view hostname
`hostnamectl set-hostname new_hostname` - to change the current hostname
Restart your Virtual Machine.
`sudo nano /etc/hosts` - change current hostname to new hostname
`lsblk` to display the partitions
`dpkg -l | grep sudo` – to show that sudo is installed
`sudo ufw status numbered`
`sudo ufw allow port-id`
`sudo ufw delete rule number`
`ssh your_user_id@127.0.0.1 -p 4242` - do this in terminal to show that SSH to port 4242 is working
