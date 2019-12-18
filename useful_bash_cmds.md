# Useful `bash` commands

## Handle installed packages during a major Debian/Ubuntu release upgrade

### The situation
You want to upgrade your Debian/Ubuntu system to a major release upgrade. Such a release update includes that all data on your current disk will be overwritten. You need to back up your system, and want your new system to work just like the old system. So you have many packgages installed on the old system (manually, incl. PPAs, dpkg, apt-get, etc.), and now want the install those same packages on the new sytem. 

### Ways to achieve this

A pedestrian way to achieve this is to compile a list of all (manually) installed packages in a text file, and prepend each line with the installation command. Here are three commands that collect packages and could be used to in a bash script to re-install those packages. At the same time they illustrate the problem: they all return a different number of packages.

```bash
apt list --installed | wc -l
apt list --manual-installed | wc -l
aptitude search '~i !~M' -F '%p' --disable-columns | sort -u | wc -l
```
So what command returns the _right_  packages? There are multiple ways to achieve this. I chose one of the suggested procedures outline in that link:

1. Use `apt-clone`. Install it by 
```bash
sudo apt-get install apt-clone
```
2. Create a backup with
```bash
sudo apt-clone clone path-to/apt-clone-state-ubuntu-$(lsb_release -sr)-$(date +%F).tar.gz
```

This dynamically forms the release number in short form and appends it the name of the backup archive file (using the `lsb_release` command).

3. Restore backup
```bash
sudo apt-clone restore path-to/apt-clone-state-ubuntu.tar.gz
```

4. Restore to newer release

```bash
sudo apt-clone restore-new-distro path-to/apt-clone-state-ubuntu.tar.gz $(lsb_release -sc)
```

### Manually backup your data (WIP)

The home directory contains a lot of settings, data, perhaps executables, code, etc. WIP


## All things `grep`...

### Searching for strings in all or specific files
Search recursively through all files that end on `py` or `sh`, 
* `-R` (includes symbolic links) or `-r`: recursive
* `-n`: line number
* `-w`: match whole word
* `-l`: (lower case L) can be added to provide file name of matching file

`grep --include=\*.{py,sh} -rnw . -e '.boto.'`

## Git and GitHub

Create a new branch on local `feature/mjkrause`.
```bash
git branch feature/mjkrause
git checkout feature/mjkrause
```

### You wish to know the Git root directory name (the one which contains the file `.git`). 
This likely is the name of the repository it was cloned from.
```bash
basename `git rev-parse --show-toplevel`
```

A quick way to this is by 
```bash
git checkout -b feature/mjkrause
```

### Rebasing

#### The situation (1), the problem (2, 3) and the solution (4)


1. Originally, you branched off of develop:
```text
a -- b                           develop
       \
        f -- g                   your_branch
```

2. Then someone else branched off of develop, and merged branch1 back into develop before you did. Now develop is in a different state compared to you first branched off of it.
```text
a -- b -- c        d             develop
      \    \      /
       \    d -- e               branch1
        \
         f -- g                  your_branch
```

3. To have the features containde in branch1 into your feature branch you need to rebase on develop.
```text
a -- b -- c        d             develop
           \      /
            d -- e               branch1
```

4. The solution is rebasing your changes onto the new state of develop. This is done by taking off your changes, fetching the latest state of develop, and then replaying your changes back onto that new state of develop - done.
```text
a -- b -- c        d             develop
           \      / \ 
            d -- e   \           branch1
                      \
                       f -- g    your_branch
```

First, checkout out develop and make sure that your version (local) is the same as the remote (origin) version. Run `git status` and look for the message _your\_branch_ is up-to-date with _origin/your\_branch_. If not, make it so by
```bash
git reset --hard origin/develop
```

Note that this is going delete all changes you may have (accidentally?) added to branch `develop`. But if that's not a problem you can just go ahead. Now change back to your branch and start the rebase process by
```bash
git rebase develop
```

This hopefully will have no conflicts. If so, run again `git status`. That likely returns _Your your\_branch and 'origin/your\_branch' have diverged, and have _x_ and _y_ different commits each, respectively._ That's a problem. Now you need to re-write history by forcing a push. 
```bash
git push --force-with-lease
```
and run `git status` again. Confirm that the local and remote version of that branch do not diverge anymore. Your done.

### If you feel adventurous...

You can run instead of that last command 
```bash
git rebase -Xtheirs develop
```
and your feature branch will now contain those changes that have made on the original branch after you branched off of it to create your feature branch. This was introduced with git version 1.7.3. That replays again your changes onto the current state of `develop` and seems to skip a few steps. I would recommend trying this first on a new feature branch and convince myself that it works before trying it on _your\_branch_.


### Changing the base of your feature branch

Suppose `feature/my_feature` was based on `oldBase`, but now you want it to be based on `newBase` (e.g., `develop`). You can change the base of a feature branch by 
```bash
git rebase --onto newBase oldBase feature/branch
```

[TODO: In GitHub `upstream` usually is `develop`. Suppose you need to keep up to date with changes in a different branch, say `feature/commons`. In that case set your branch to that:
```bash
git branch --set-upstream-to origin/feature/commons feature/new_feature
```

It will print `Branch feature/new_feature set up to track remote branch feature/commons from origin.`] 

After that execute
```bash
git pull
```

Print history of commits, with the most recent one on top of the screen:
```bash
git log
```
If you use it with the `--graph --decorate` options the output includes the branch tree in color.

### List all branches and who last committed to that branch
```bash
git for-each-ref --format='%(committerdate) %09 %(authorname) %09 %(refname)' | sort -k5n -k2M -k3n -k4n
```
We can't infer who is the author of a git branch. Git branches are pointers to a commit. As such it is not possible (not to my knowledge) who created a branch.

### Show the the name or organization a repo was originally cloned from
```bash
xdg-open `git config --get remote.origin.url`
```
This opens the organization with the repository in a new browser window.

### Deleting a branch
To delete a branch push it to remote first, then delete on remote, and last delete it on local. Suppose the branch we want to delete is `feature/my-branch`. Most times the name of the remote is `origin`, but it doesn't need to be. Option `-D` is forced delete or `--delete --force`.

```bash
git push --delete origin feature/my-branch
git branch --delete feature/my-branch
git branch -D feature/my-branch
```

Following this execute
```bash
git fetch --prune origin
```

#### Updating remote branches on local
Once you have merged a branch on the webinterface and you deleted the feature branch right there, you will still see that feature branch after `git branch -r` on your local. To refresh the tree on your local issue
```bash
git -c core.quotepath=false fetch origin --progress --prune
```
If you check again for remote branches you should not see the current status of feature branches. You still need to delete that branch locally as described in the previous paragraph.

### Squashing commit messages when `Squash and merge` isn't made available in repo

To squash multiple commmits into a single commit first get an estimate on how many commits are affected. In the GitHub UI that number shows up in the tab. Suppose it's 64. In that case do

```bash
git rebase -i HEAD~70
```
where `-i` stands for _interactive_. That command opens the editor (in our case emacs) that contains your last 70 commits to that branch, in the time-ascending order (so the last commit is at the end of that list). Each line in that document is prefixed with `pick`. Find the first commit where you want to start squashing. Leave this line alone, and start replacing `pick` with `s` in each other line to the last line of the commits.

Save the file and close it. This will open a second editor window with the commit messages and some other information. Delete the entire content and replace by the commit message for the squashed commit. Save and close.

Now the local copy of the branch will differ from the remote copy. Therefore you need to force the push like so
```bash
git push origin feature/my-branch --force
```

The push copies your local branch into the remote repository. It is good practice to create a test branch, and perform the above steps on it. Suppose that branch is called `delme`. Once done with the process on `delme` do
```bash
git diff feature/my-branch delme
```
to see the difference between the two branches.

### Using `git apply`

This command can take a patch file, which contains changes in a feature branch in mailbox format, and apply it to the current branch. For instance, suppose you're in branch `my_botched_branch`. For some reason you can't merge it easily to a target branch. Suppose you have branched off `develop`. In that case you could pipe the changes in the files of `my_botched_branch` to a file
```bash
git diff develop my_botched_branch > my_patch.patch
```
Now checkout `develop` and create a new feature branch `my_branch`, checkout this branch, and apply the changes in `my_patch.patch`
```bash
git apply my_patch.patch
```
This method can be a good choice if you find yourself in some mess with rebase or merge with an existing branch.

### Showing only the headlines of commit messages
```bash
git log <some-old-commit>..<latest-commit> --pretty=oneline
```
The commits can also be tags or release numbers.


## Getting a specific file out of an archive
Suppose you have an archive in the directory `myarchive.gz`. The first three lines list all files in that archive. `myarchive.gz` could be buried deep in a file structure, for instance `root/files/foo/bar/`. You would learn about that path by listing the file. The is important as the entire path is the second argument for the specific file you wish to extract. The last line would extract `myarchivefile.gz` from the original archive into the present directory.

```bash
tar -tf /foo/bar/baz/myarchive.gz
tar -tf /foo/bar/baz/myarchive.gz | grep 'arch.*gz'
tar -tf /foo/bar/baz/myarchive.gz root/files/foo/bar/myarchive.gz
tar -xf /foo/bar/baz/myarchive.gz root/files/foo/bar/myarchive.gz .
```

### Copying a file from a VM to another server using `rsync`
Now suppose you want to copy the file you extracted above to the local server
```bash
rsync -avz username@server:/home/username/foo/bar/baz/root/files/foo/bar/myarchive.gz .
```
So again, you need to use the complete path information of the file (i.e., the location to which you wrote it to in the above step).

## Using `jq`
You can always use `set x` in any bash script to show on standard out what the script executes. Here I use the `jq` (not installed by default) to substitute a value in a JSON file `config.json`. We first need to copy the command into a temporary file, and then move it to the actual file. Copy this line into a bash script and execute it with a desired value as argument.
Use a temporary variable in order to do in-place editing.
```bash
tmp=$(mktemp)
jq --arg APP_NAME "$1" '.app_name |= $APP_NAME' .chalice/config.json > "$tmp" && mv "$tmp" .chalice/config.json
```
### Substitute a value for an existing key in a JSON file
Suppose you have `my_val="ABCD"` and you want to assign this as a value in an existing JSON file named `myfile.json`, which is an array at the first level. 
```bash
tmp=$(mktemp)
jq --arg my_val "$my_val" '.[].my_key1.my_key2 = $my_val <path_to_JSON> > "$tmp" && mv "$tmp" <path_to_JSON>
```

## Using `rsync`

Suppose we have a directory `dir1` on a remote machine and also local. To synchronize the content of that directory between the two locations, we run
```bash
rsync -avz user@machine:/home/user/dir1 dir1
```

The option `v` is verbose, `a` is ???

## Control daemons
Get the status of a daemon like so
```bash
$ sudo systemctl | grep NAME
```
Control the status with

```bash
sudo service <daemon_name> <option>
```
where options are start or stop (among many others, see help file). You can also use
```bash
sudo systemctl NAME OPTION
```
where frequently used options are `start`, `status`, `stop`, `restart`.

# AWS CLI commands

When working with AWS CLI we usually need to provide credentials. AWS' command-line utilty is `aws`, and `aws configure` can help setting up the credentials. Once set up they usually reside in `~/.aws/`.

To work with the `aws` utility set the profile configuration of your credentials into an environment variable. If the profile's name is `my-profile`, then run
```bash
export AWS_DEFAULT_PROFILE=my-profile
```
That way the `aws` utility can be used in a given shell without providing credentials with each command.

## List files in bucket `bucket_name`, which has prefix `my_prefix`:
```bash
aws s3 ls s3://bucket_name/my_prefix/
```

## Create a directory `my_new_dir` in a bucket `bucket_name`, which has prefix `my_prefix`:
```bash
aws s3api put-object --bucket bucket_name --key my_prefix/my_new_dir
```

## Remove a file `somefile.txt` from a bucket `bucket_name`, with key `my_key`:

```bash
aws s3 rm s3://bucket_name/my_key/somefile.txt
```

## Remove recursively `file1.txt`, `file2.txt`, ... in prefix key `some_prefix` in a bucket
```bash
aws s3 rm s3://bucket_name/my_key/some_prefix/ --recursive
```
Careful, though, I think it also deletes prefix key `some_prefix`.

# Installing Python-related bioinformatics graph tools

To create figures for the HCA project summary statistics I needed to generate t-Distributed Stochastic Neighbor Embedding (tSNE) figures, using the ScanPy package. This had two requirements: `louvain` and `igraph`. 

I could install `igraph` using `pip install python-igraph`.

[`louvain`](https://github.com/taynaud/python-louvain) is the implementation an algorithm of the same name, which maximizes modularity in graphs. Modularity here denotes groups, clusters of communities, so essentially a measure of similarity. The idea is that nodes which is similar to each other are connected by shorter edges than nodes that are different from each other. The algorithm is named by the work place of the researchers who discovered and published it [(Blondel et al. (2008)](http://doi.org/10.1088/1742-5468/2008/10/P10008), namely the University of Louvain in Leuven, Belgium. 

The Python package `louvain` contains a C implementation of that algorithm. But it has also [igraph](https://igraph.org/python/) as a requirement.

Both packages are dependent on a number of libraries: `build-essential`, `libxml2-dev`, `libglpk-dev`, `libgmp3-dev`, `libblas-dev`, `liblapack-dev`, `libarpack2-dev`, `python-dev`. I installed those using `apt-get`. 

Only after installing those librarries I was able to install 
```bash
pip install louvain
```

Before I installed `louvain` I was able to use `python:3.6.9-slim-stretch` to dockerize it. But I'm not sure if I can install those libraries as they require the `apt-get` manager which I think that base image doesn't have.

# Useful Docker commands
Note: most Docker commands need to be executed as *root* or using `sudo`. You can the username to the Docker group then you should be able to execute docker command without `sudo`.

## List all Docker images on the system
```bash
docker images
```

## List all Docker containers (i.e., built images) on the system
```bash
docker ps
```

## Run a docker container `my-container` that contains a script that needs AWS credentials

We create a Docker volume with the `-v` flag, and set an environment variable using the `-e` flag.
```bash
docker run -v ~/.aws:/root/.aws -e AWS_DEFAULT_PROFILE=my-profile my-container
```
## To start a bash shell to interact with docker container
`sudo docker exec -it boardwalk_dcc-dashboard-service_1 bash`

or 
`docker run --rm -it --entrypoint=/bin/bash 'name_of_docker_image'`

## To tail a log file from a specific Docker container
```bash
sudo docker logs -f name_of_container
```

## Stop containers from running

```bash
docker kill <image_id>
```
(likely you need to run the above command using `sudo`) Instead of `<image_id>` you can use command substitution `$(sudo docker ps -q)` (which prints a list of image IDs only) to loop over all containers images. Besides, this technique can be used for any of the other docker commands in this paragraph.

## Remove (delete) all containers
```bash
sudo docker rm $(sudo docker ps -a -q)
```
Executing this command has been useful in the case that a container (let's say _container1_) is used by another, stopped container (let's say _container2_). In such case, the image of _container1_ can't be delete using the command below. Instead, when I executed it in that case, it returned 
```bash
Error response from daemon: conflict: unable to delete <ID of container2> \
(must be forced) - image is being used by stopped container <ID of container1>
```
But if you run the above `rm` command the below `rmi` command should delete all images.

## Remove (delete) all images from system (i.e., disk)
```bash
sudo docker rmi $(sudo docker images -q)
```

## `docker-compose` commands

### Stop and start Docker compose 
`docker_compose1.yml` and `docker_compose2.yml` are Docker compose files.
```bash
sudo docker-compose -f docker_compose1.yml -f docker_compose2.yml down -v
```

If you get
```bash
sudo docker-compose -f dev.yml down -v
ERROR: build path /home/ubuntu/cgp-deployment/boardwalk/boardwalk either does not exist, is not accessible, or is not a valid URL.
```
it means that you had not brought up the containers using `dev.yml`. 

## Reading the logs for debugging
For development work reading the logs is very useful. For instance, the Docker compose network of containers of CGP's computation platform consists of 9 containers, each of which has a log.
```bash
docker exec -it core-nginx bash
```
Use
```bash
sudo docker-compose logs
```
That command only accepts the file names `docker-compose.yml` or `docker-compose.yaml`. So if your config files are named differently you need to copy them to one of the two names in order to be able to read the logs.

```bash
sudo docker-compose -f docker_compose1.yml -f docker_compose2.yml up -b
```

Also in development mode, if you want to built the Boardwalk containers from the image, then first bring all containters in `boardwalk/` down 
```bash
sudo docker-compose -f dev.yml down -v
```
which also removes all volumes from the network. Then remove the images needed for Boardwalk
```bash
sudo docker rmi <image_id>
```
There should be 3-4 images. Then bring up the network by using 
```bash
sudo docker-compose -f dev.yml --build
```
That will build the containers from the images.


### Delete containers from system
Purge (i.e., delete) all unused or dangling images, containers, volumes, and networks
```bash
docker system prune
```
Use it with the `-a` option to remove any stopped container and all unused images (not just dangling). You'll be surprised how many deleted lines it prints... At the end it reports the amount of reclaimed disk space. 

After the command has finished verify that no image is left by `sudo docker images`. It should show no entries.

### Other Docker commands
(all docker commands need to be prefixed with `sudo`)
```bash
# remove exited containers:
docker ps --filter status=dead --filter status=exited -aq | xargs -r docker rm -v
    
# remove unused images:
docker images --no-trunc | grep '<none>' | awk '{ print $3 }' | 
   \ xargs -r docker rmi <image_id>

# remove any image (run docker images first to get the IMAGE ID, suppose it's 60503d5b81fb):
docker images | grep 60503d5b81fb | awk '{print $1 ":" $2}' | xargs docker rmi

# remove unused volumes
docker volume ls -qf dangling=true | xargs -r docker volume rm
```

# Problem with a native `nginx` service running on my VM
First find out whether a native (i.e., not dockerized) `nginx` is running
```bash
sudo service nginx status
sudo service nginx stop
```
and stop its service. Next find out 
```bash
sudo docker run -i -t 2e92148b6758 "nginx -g 'daemon off;'" 
```

## How to test webpage functionality of WSGIs
If anything goes wrong with a webservice, the first thing to check are the logs. You need to log in as root to read it. On a VM do
```bash
sudo -i
less dcc-dashboard-service/logs/error_uwsgiB.log
```
If you execute `G` (a VI command to get to the last page) you can move around the text page-by-page.

When programming the backend to affect actions controlled by a frontend webpage it is most useful for testing to monitor the logs. For instance, when I worked on the manifest handover I checked what happened to the code on the backhand by monitoring `~/dcc-dashboard-service/logs/error_uwsgiB.log` as root and running it in the foreground (`-f` flag) by using the `tail` command:
```bash
~/dcc-dashboard-service/logs# tail -f error_uwsgiB.log
```
If you operate some functionality on the webpage (i.e., click a button) you see immediately how info messages and error messages fly by on the terminal. Use `Ctrl-C` to exit.

## `pip` installations

### You want to uninstall all packages in the current virtual environment

```bash
pip uninstall -y -r <(pip freeze)
```

### `venv` command in Python 3: the recommended way to create virtual environments

First install the module
```bash
sudo apt install python3-venv
```
To create a virtual environment `newenv` use
```bash
python3 -m venv newvenv
```

I'm not sure how to create a virtual environment using a specific Python point version as is possible to do using `virtualenv` ([here](https://stackoverflow.com/questions/54700307/python3-m-venv-how-to-specify-python-point-release-version) is my question on Stackoverflow).


## Internet speedtest

Download a pip-installable app:
```bash
pip install speedtest-cli
```

Then run
```bash
speedtest-cli
```
and it will report download and upload speed.

## How to secure copy data to and from an EC2 instance
```bash
scp -i "/home/michael/.ssh/sec_key_mk.pem" file_to_scp.txt ubuntu@ec2-35-161-55-66.us-west-2.compute.amazonaws.com:/home/ubuntu
```

## Install `umake`

```bash
sudo add-apt-repository ppa:ubuntu-desktop/ubuntu-make
sudo apt update
sudo apt install ubuntu-make
```
and that's it. Run `umake` on the command line to see the options. I've used it to install Eclipse.

## Java

### IntelliJ
I installed the community edition (CE) by using _snap_, which is nice as it updates automatically to the latest version.

#### Find libraries and install them

When IntelliJ is installed using _snap_, find all libraries in 
```bash
/snap/intellij-idea-community/current/lib
```

### In case you lost the project structure
File -> Projectstructure -> modules -> import module
should solve it.

### Install Eclipse
I've used `umake` (see *umake*) to install the latest version of Eclipse. After `umake` is installed, it's a breeze. The following install Eclipse for use with `Java`.
```bash
umake ide eclipse
```
and done.

## How to `scp` to a AWS AMI

```bash
scp -i "/home/michael/.ssh/sec_key_mk.pem" file_to_scp.txt ubuntu@ec2-35-161-55-66.us-west-2.compute.amazonaws.com:/home/ubuntu
```

## Use a start-up script to stop the avahi daemon
The avahi daemon seems to cause a problem after each reboot. I decided to write a startup script that stops its service right during reboot. This need to work with `systemd`, a new system introduced with Ubuntu 16.

To start a service we need two files:
1. a shell script `stop-avahi` in `~/bin`
2. a service script `stop-avahi.service` in `/etc/systemd/system`

The shell script has the content:
```bash
#!/bin/sh

sudo rm -rf /var/crash/.lock*
sudo rm -rf /var/crash/*
sudo service avahi-daemon stop
```
and the systemd script contains:

```bash
[Unit]
Description=Power-off gpu

[Service]
Type=oneshot
ExecStart=/usr/bin/vgaoff

[Install]
WantedBy=multi-user.target
```
I made both scripts executable with `chmod +x ...`, and then executed

```bash
$ systemctl enable stop-avahi.service
Created symlink from /etc/systemd/system/multi-user.target.wants/stop-avahi.service to /etc/systemd/system/stop-avahi.service.
```
which started the service. It should start automatically after reboot. See also [this link](https://unix.stackexchange.com/questions/47695/how-to-write-startup-script-for-systemd/47715#47715https://unix.stackexchange.com/questions/47695/how-to-write-startup-script-for-systemd/47715#47715)


## Setting the ALPS Dual Trackpoint on Dell Precision 7510 laptop running Linux

Settings for the trackpoint are not set right by default. It is oversensitive such that I was unable to get the pointer into any small spot. In order to set it on Linux (Ubuntu) we first need list the identifier for the mouse and track pointer components:

```bash
xinput list
```
It shows that the AlpsPS2/2 ALPS DualPoint Stick has ID 15. Another interesting list of settings is returned by

```bash
xset q | grep -iA 1 pointer
```
although the interpretation of these settings are not yet clear to me. But moving on, we now want to know what are the properties of the pointer stick, and we find out by querying it:
```bash
xinput --list-props 15
```
The parameters of importance are _Device Accel Constant Deceleration (295)_, _Device Accel Adaptive Deceleration (296)_ and _Device Accel Velocity Scaling (297)_. I've set these values to 5, 5, and 10, respectively, using

```bash
xinput --set-prop 15 295 5
xinput --set-prop 15 296 5
xinput --set-prop 15 297 10
```

Further, to adjust the scrolling of the page using the stick point while holding the middle button I set
```bash
xinput --set-prop 15 348 100
```

Following this [bug-link](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1657553) I create a file named `71-pointingstick-precision.hwdb` with content `evdev:name:*DualPoint Stick:dmi:bvn*:bvr*:bd*:svnDellInc.:pnPrecision7510*:pvr*
 POINTINGSTICK_CONST_ACCEL=0.025`, although I do not understand the impact of it (i.e., this value is very different from the one I input above for the contstant acceleration). After this I executed 
 
```bash
sudo systemd-hwdb update
sudo udevadm trigger 
```

Lastly, I create a file `.xsession`, in which I wrote `xinput set-prop 'AlpsPS/2 ALPS DualPoint Stick' 'libinput Accel Speed' 1`. 

Before I did the last step the step before the last step I noticed no changed when I changed parameters in the `xinput`, and only after these two last steps did such changes take effect.


## GUI

### Top bar of window is off screen after plugging laptop back docking station

`ALT + spacebar`, then choose `move` and recenter it

## Working with Python 3
Install 
```bash
sudo apt-get install -y python3-pip
```
then create a virtual environment `.envblue`
```bash
virtualenv --python=python3 .envblue
```
or like so
```bash
virtualenv --python=$(which python3) .envblue
```
and `source` it, or 
```bash
virtualenv --python=python3.6 .venv
source .venv/bin/activate
ipython3
```
to use Python 3.6 in your IPython (Python 3.6 needs to be installed on your system).

```bash
pip3 install python 
pip3 install ipython
```

## Batch install all modules listed in `requirements.txt`

```bash
pip3 install -r requirements.txt
```

in Python 2.7 that should work (unconfirmed)
```bash
pip install -r requirements.txt --no-index --find-links file:///tmp/packages
```

## Installing nvidia graphics driver on the Dell Precision 7510 laptop
This manual should lead to an easy installation of Nvidia drivers for the graphics card. This is much easier compared to the installation on Debian, which does not have a graphics card PPA (_personal package archive_), and where I need to install the run file I had downloaded from Nvidia.

1. Find out what graphics 

Identify the graphics card(s) using

```bash
$ sudo update-pciids
$ lspci -nn | grep -i --color "2d\|3d\|display\|vga"
```
This will return two graphics cards, one Intel, one Nvidia. The Intel I think comes with the motherboard, the Nvidia GM107GLM Quadro M2000M I ordered with the system, but it isn't used after a fresh Ubuntu installation. You need to get the Nvidia drivers for it and install them. For Ubuntu 16.04 this seems easiest by using apt-get.

For the remainder of the installation I followed [these](http://www.linuxandubuntu.com/home/how-to-install-latest-nvidia-drivers-in-linux) instructions

2. Add the graphics driver PPA as shown [here](https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa)

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
```
I think this is `deadsnakes-ubuntu-ppa-xenial` in `/etc/apt/sources.list.d/`.

3. Now simply use apt-get to install the driver. I did
```bash
$ sudo apt-get install nvidia-352
```
and rebooted the machine. Everything went just find, no black TTY log-in screen, my desktop manager ran just as before. According to Nvidia my card is compatible with driver version 352 (http://www.nvidia.com/Download/driverResults.aspx/95159/en-us). 


4. After reboot I checked the Nvidia GUI. The installed driver version is 384, and everything seems to work. It shows that the card has 4 GB of memory, and can also save configurations.

5. Check information on Nvidia driver
Run `$ nvidia-smi`. This will list all features of your installed Nvidia card. Also, `lsmod | grep nvidia` will now have an output, while `lsmod | grep nouveau` will have no output (it should have had an output before the installation).

6. The only way to prevent the system from upgrading (I think this could become a problem when I use specific GPU settings, which rely on the version), we need to uninstall the graphics driver PPA.
```bash
$ sudo add-apt-repository --remove ppa:PPA_Name/ppa
```
where PPA_name/ppa is `graphics-drivers/ppa`.

7. If you want to uninstall the Nvidia driver use `sudo apt-get purge nvidia*`.


## SSL certificates

The purpose of a certificate is to certify to the browser that if you point it at bankofamerica.com you indeed handshake with the website of the Bank of America, and not some imposter website. This is done by a third-party certificate authority (CA) which has issued BoA a certificate, which works by private / public key encryption.

In development, for example when using letsencrypt, fake certificates can be used instead of real certificates.


## `ssh` issues

### `Offending key for IP in /home/<username>/.ssh/known_hosts:<line_num>`
This can happen, for example, if you have two EC2 instances, say *I1* and *I2*. You logged in to *I1* just find using `ssh`. Then you dissociate its *Elastic IP* and associate it with *I2*. Once you try to `ssh` into *I2* the private / public key handshake doesn't work anymore (because it's not the one from *I1*), but the IP is correct. That's when you see this line.

You can remedy this using brute force by just deleting that line in file *known_hosts* using

```bash
sed -i '<line_num> ~/.ssh/known_hosts
```
assuming the *line_num* is the line number shown in the warning.


## `xclip`

If you have a password saved in a file, and you want to copy it to the X11 clipboard (i.e., without displaying it on the screen), you can use `xclip` like so:
```bash
xclip -sel c < file
```
where `file` is the file that contains the password.


## AWS

### Key for `*.pem` file
It's in the _Secrets Manager_, and there click the button _keys_

### Checking log files in _CloudWatch_

This is an example from repo _azul_, where we want to check the logs of the AWS lambdas and others. In the AWS console go to _CloudWatch_, then in _Logs_. There use the search prompt. To find the logs for the service lambda in the _dev_ deployment search for `/aws/apigateway/azul-service`. To find the logs for the service lambda running my personal deployment, search for `/aws/lambda/azul-service-mjkrause`

Note that time in the _Logs_ page is reported as local time. But if you click on a specific log stream it is reported as UTC (should be 7 h difference). 

In the upper left corner, right above the table of Time/Message, click on the timestamp. A window opens which can be used to set the time interval of log times. Of note is the _Relative_ tab, which can be used to return only logs from the last say 10 min. It also has the option between local and UTC time. It sometimes takes awhile for the log to appear. Click on the refresh button often.


## Perl one-liners

### Use regex to extract specific lines from a text file.
Suppose we want all lines which start with exactly two whitespaces (e.g., this could be some configuration file).
```bash
in_file=$( perl -ne 'print if /^\s{2}(?!\s)/' filename.txt) 
```

## JavaScript, Node.js, and TypeScript

### Developing inside a running Docker container

This set of instructions is specific to developing an app created with Strongloop's Loopback.

Suppose you have a Docker container or a Docker-compose script which launches multiple containers, and you would like to develop the code-base while your code is running inside the container (and the container(s) could run on localhost or remotely). This can be done with VS Code. 

1. Launch VS Code (`code .` inside the directory that contains the code-base opens the IDE). Install the `Remote Explorer` extension.
2. Navigate to the directory containing the code-base, including the `Dockerfile` or the `docker-compose` file. Launch the container(s).
3. In VS Code, click on the Remote Explorer icon (on the left side of the window by default). You should see the container(s) listed. Pick the one that contains the code you want to develop, and connect to it.
4. Hover the container of interest, right mouse button, and click on `Connect to Container`. That action opens another VS Code editor which shows the content of the container. Navigate to the directory holding the code. Also, you should see a green bar on the lower left side of the screen that should something like `Container ...` describing the name of the container.

Now you're connected to the container, and can edit the code inside the running container.

### Debugging Node.js code

Suppose you have a Node.js application running as a backend to some frontend. You would like to exercise some code in the backend that is called by some HTTP method in the front end. 

1. Set a breakpoint at some line where you want the debugger to stop. Next hit `F5` or navigate to menu `Debug` -> `Start Debugging`. In the debugger tab you should see that the debugger has attached to some process.

2. Identify the endpoint that invokes the Node.js backend method and create a HTTP request. In a terminal run `curl <METHOD> http://myendpoint.org`.

3. The debugger should stop at the first breakpoint it passes, i.e., the red dot should be framed with a yellow arrow. Use `F10` and other keys to inspect variables and call stack. Et voila...
