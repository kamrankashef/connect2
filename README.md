# connect2

A simple way to use aliased hosts and ports to:

- [ssh](#ssh-into-a-host)
- [scp](#scp-a-file)
- [Execute an inlined command on multiple servers](#remote-execute-a-command)
- [Execute a local script on multiple servers](#run-a-local-script-remotely)


### Installation and Usage
```
$ curl -O https://raw.githubusercontent.com/kamrankashef/connect2/master/connect2 
$ chmod +x connect2 
$ ./connect2 -h
Usage: connect2 [options]
    -c, --command [command_to_run]   command
    -t, --scp                        Transfer file (SCP)
    -r, --recursive                  SCP recursive
    -a, --action [action]            Action (post or get)
    -l, --localpath [localpath]      Path to local file or directory
    -s, --serverpath [serverpath]    Path to remote file or directory
    -p [scipt_path],                 Script to execute remotely
        --path_to_script
    -m [':' sperated list machine aliases],
        --machines                   Machine aliases to connect to
    -h, --help                       Show this message
```

### Configure Aliases
Create `~/.connect2` and specify remote hosts following this example:
```
{
 :machines => [
 {:alias => "aws_ubuntu1", :host=>"ubuntu-user@50.1.10.115", :port=>"1507"},
 {:alias => "webserver", :host=>"admin@10.1.10.115", :port=>22}
 ]
}
```

### Example Usage

#### ssh into a host

```
$ ./connect2 
No end point specified
[1] aws_ubuntu1 => ubuntu-user@50.1.10.115:1507
[2] webserver => admin@10.1.10.115:22
Select your host(s): 1
Connection to ubuntu-user@10.1.10.115
Running: ssh ubuntu-user@10.1.10.115 -p 1507
Welcome to Ubuntu 17.04 (GNU/Linux 4.10.0-21-generic x86_64)
ubuntu-user@webserver:~
```

#### scp a file
scp file (use `-r` for recursive copies):
```
$ ./connect2 -t -a post -m aws_ubuntu1 -l my_file.txt -s /tmp
Running: scp -P 1507 "my_file.txt" ubuntu-user@50.1.10.115:"/tmp"
```

Validate it was copied using `connect2`:
```
$ ./connect2  -m aws_ubuntu1 -c "ls -l /tmp/my_file.txt"
Runnig ssh ubuntu-user@50.1.10.115 -p 1507 'source .bashrc; ls -l /tmp/my_file.txt'
-rwxr-xr-x 1 ubuntu-user ubuntu-user 4603 Aug 11 15:59 /tmp/my_file.txt
```

#### Remote execute a command

Execute `ls` on aws_ubuntu1 and webserver

```
$ ./connect2 -m aws_ubuntu1:webserver -c ls
Runnig ssh ubuntu-user@50.1.10.115 -p 1507 'source .bashrc; ls' 
bin
opt
r_scripts
Runnig ssh admin@10.1.10.115 -p 22 'source .bashrc; ls' 
bin
tmp
```

#### Run a local script remotely

Create `sample.sh` script to get the remote server's date and `ls /tmp`:

```
$ echo "date
> ls /tmp" > /tmp/sample.sh

$ ./connect2 -m aws_ubuntu1 -p /tmp/sample.sh 
Running script /tmp/sample.sh on aws_ubuntu1
Fri Aug 11 15:32:22 EDT 2017
catalina.out
foo
hsperfdata_root
hsperfdata_ubuntu-user
master.log
23igane.log
systemd-private-ff7d61c228b28a6097b7a172-systemd-resolved.service-kHttp
```
