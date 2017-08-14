# connect2

A simple script to use aliased host names and ports to login, copy files and execute commands remotely in batch.

- [Most Basic Example(#most-basic-example)
- [Installation](#installation)
- [Configure Aliases](#configure-aliases)
  - [Initial Configuration](#initial-configuration)
  - [Adding Entries](#adding-entries)
  - [Deleting Entries](#deleting-entries)
  - [Config File Format](#config-file-format)
- [Example Usage](#example-usage)
  - [ssh](#ssh-into-a-host)
  - [scp](#scp-a-file)
  - [Execute an inlined command on multiple servers](#remote-execute-a-command)
  - [Execute a local script on multiple servers](#run-a-local-script-remotely)

### Most Basic Example
 
Instead of remembering the username, host and port of a instance:

`ssh ubuntu-user@ec2-somelong-host-that-will-prob-change.com -P 1000`

Simply call:

`./connect2 dev-machine`

Similarly, you can copy files, and quickly run inline-commands or local scripts remotely.  Configuration is easy and can be done interactively.

### Installation
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

#### Initial Configuration

If you do not have a `~/.connect2` the first time you run `connect2` you will be prompted to create an alias:

```
$ ./connect2 
No config found, create new machine alias
Alias Name: local
User Name: kkashef
Host: localhost
Port: 22
Added: {:alias=>"local", :host=>"kkashef@localhost", :port=>"22"}
```

#### Adding Entries

```
$ ./connect2 --configure
Add, delete or quit entry? [a,d,q]: a
Alias Name: ec2-instance  
User Name: ubuntu-user
Host: ec2-2idaegae.aws.amazon.com
Port: 1000
Added: {:alias=>"ec2-instance", :host=>"ubuntu-user@ec2-2idaegae.aws.amazon.com", :port=>"1000"}
Add, delete or quit entry? [a,d,q]: q
$
```

#### Deleting Entries

```
$ ./connect2 --configure
Add, delete or quit entry? [a,d,q]: d
[1] local => kkashef@localhost:22
[2] ec2-instance => ubuntu-user@ec2-2idaegae.aws.amazon.com:1000
Select your host(s): 2
Deleting:
[{:alias=>"ec2-instance",
  :host=>"ubuntu-user@ec2-2idaegae.aws.amazon.com",
  :port=>"1000"}]
Add, delete or quit entry? [a,d,q]: q
$
```

#### Config File Format

The undelying confguration is a Ruby map stored in `~/.connect2` in this format:

```
{:machines=>
  [{:alias=>"local", :host=>"kkashef@localhost", :port=>"22"},
   {:alias=>"ec2-instance", :host=>"ubuntu-user@ec2-2idaegae.aws.amazon.com", :port=>"1000"},
   {:alias=>"node1", :host=>"ubuntu-user@localhost", :port=>"1001"},
   {:alias=>"webserver", :host=>"ec2-user@ec2-2idaegae.aws.amazon.com", :port=>"1000"},
   {:alias=>"local-docker", :host=>"kkashef@localhost", :port=>"4000"}]}
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
