# 进入docker容器



docker容器没起sshd服务的时候，还想进入docker怎么办，我们可以使用nsenter进入docker容器的命名空间namespace来虚拟登入docker容器。



nsenter一般默认自带了，没有的话，安装下。然后把如下这个脚本命名成docker-enter或者别的名字，赋执行权限，放入$PATH下即可，使用方式： docker-enter &lt;container-id&gt;



\#!/bin/sh

if \[ -e $\(dirname "$0"\)/nsenter \]; then

	\# with boot2docker, nsenter is not in the PATH but it is in the same folder

	NSENTER=$\(dirname "$0"\)/nsenter

else

	NSENTER=nsenter

fi

if \[ -z "$1" \]; then

	echo "Usage: \`basename "$0"\` CONTAINER \[COMMAND \[ARG\]...\]"

	echo ""

	echo "Enters the Docker CONTAINER and executes the specified COMMAND."

	echo "If COMMAND is not specified, runs an interactive shell in CONTAINER."

else

	PID=$\(docker inspect --format "{{.State.Pid}}" "$1"\)

	if \[ -z "$PID" \]; then

	  exit 1

	fi

	shift

	OPTS="--target $PID --mount --uts --ipc --net --pid --"

	if \[ -z "$1" \]; then

	  \# No command given.

	  \# Use su to clear all host environment variables except for TERM,

	  \# initialize the environment variables HOME, SHELL, USER, LOGNAME, PATH,

	  \# and start a login shell.

	  "$NSENTER" $OPTS su - root

	else

	  \# Use env to clear all host environment variables.

	  "$NSENTER" $OPTS env --ignore-environment -- "$@"

	fi

fi

