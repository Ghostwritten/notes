#  shell 挂载目录

```bash
# Notes:
#  - Please install "jq" package before using this driver.
usage() {
	err "Invalid usage. Usage: "
	err "\t$0 init"
	err "\t$0 mount <mount dir> <json params>"
	err "\t$0 unmount <mount dir>"
	exit 1
}

err() {
	echo -ne $* 1>&2
}

log() {
	echo -ne $* >&1
}

ismounted() {
	MOUNT=`findmnt -n ${MNTPATH} 2>/dev/null | cut -d' ' -f1`
	if [ "${MOUNT}" == "${MNTPATH}" ]; then
		echo "1"
	else
		echo "0"
	fi
}

domount() {
	MNTPATH=$1

	local NFS_SERVER=$(echo $2 | jq -r '.server')
	local SHARE=$(echo $2 | jq -r '.share')
	local PROTOCOL=$(echo $2 | jq -r '.protocol')
	local ATIME=$(echo $2 | jq -r '.atime')
	local READONLY=$(echo $2 | jq -r '.readonly')

	if [ -n "${PROTOCOL}" ]; then
		PROTOCOL="tcp"
	fi

	if [ -n "${ATIME}" ]; then
		ATIME="0"
	fi

	if [ -n "${READONLY}" ]; then
		READONLY="0"
	fi

	if [ "${PROTOCOL}" != "tcp" ] && [ "${PROTOCOL}" != "udp" ] ; then
		err "{ \"status\": \"Failure\", \"message\": \"Invalid protocol ${PROTOCOL}\"}"
		exit 1
	fi

	if [ $(ismounted) -eq 1 ] ; then
		log '{"status": "Success"}'
		exit 0
	fi

	mkdir -p ${MNTPATH} &> /dev/null

	local NFSOPTS="${PROTOCOL},_netdev,soft,timeo=10,intr"
	if [ "${ATIME}" == "0" ]; then
		NFSOPTS="${NFSOPTS},noatime"
	fi

	if [ "${READONLY}" != "0" ]; then
		NFSOPTS="${NFSOPTS},ro"
	fi

	mount -t nfs -o${NFSOPTS} ${NFS_SERVER}:/${SHARE} ${MNTPATH} &> /dev/null
	if [ $? -ne 0 ]; then
		err "{ \"status\": \"Failure\", \"message\": \"Failed to mount ${NFS_SERVER}:${SHARE} at ${MNTPATH}\"}"
		exit 1
	fi
	log '{"status": "Success"}'
	exit 0
}

unmount() {
	MNTPATH=$1
	if [ $(ismounted) -eq 0 ] ; then
		log '{"status": "Success"}'
		exit 0
	fi

	umount ${MNTPATH} &> /dev/null
	if [ $? -ne 0 ]; then
		err "{ \"status\": \"Failed\", \"message\": \"Failed to unmount volume at ${MNTPATH}\"}"
		exit 1
	fi

	log '{"status": "Success"}'
	exit 0
}

op=$1

if ! command -v jq >/dev/null 2>&1; then
	err "{ \"status\": \"Failure\", \"message\": \"'jq' binary not found. Please install jq package before using this driver\"}"
	exit 1
fi

if [ "$op" = "init" ]; then
	log '{"status": "Success", "capabilities": {"attach": false}}'
	exit 0
fi

if [ $# -lt 2 ]; then
	usage
fi

shift

case "$op" in
	mount)
		domount $*
		;;
	unmount)
		unmount $*
		;;
	*)
		log '{"status": "Not supported"}'
		exit 0
esac

exit 1
```

