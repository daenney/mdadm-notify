#!/usr/bin/env sh
#
# Author: Daniele Sluijters
# Source: https://github.com/daenney/mdadm-notify
# License: GPLv3
# Version: 0.1.0

required_system_utils="cut id logger basename sudo notify-send"
for util in $required_system_utils; do
    command -v "$util" >/dev/null 2>&1 || {
        echo >&2 "Could not find $util. Please use your system package manager to install it."; exit 1;
    }
done


send_message() {
    urgency=$1
    summary=$2
    body=$3
    category='device'
    timeout=3000

    if test "$urgency" = 'critical'; then
        category='device.error'
        timeout=0
    fi

    # When we don't have a DBUS_SESSION_BUS_ADDRESS it's likely we were called by mdadm-monitor
    # that was started as part of the system startup session and not part of the login session.
    # Find non-system users, check if there's a socket named bus and use that instead.
    if test -z "$DBUS_SESSION_BUS_ADDRESS"; then
        for uid in /run/user/*; do
            uid=$(basename "$uid")

            if test "$uid" -ge 1000; then
                dbus_socket="/run/user/$uid/bus"
                if test -S "$dbus_socket"; then
                    username=$(id -un "$uid")
                    response=$(sudo -u "$username" DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$uid/bus" -- notify-send -u "$urgency" -a 'mdadm' -i 'drive-harddisk' -c $category -t $timeout "$summary" "$body" 2>&1)
                    if test $? -ne 0; then
                        logger -t 'mdadm-notify' "Could not notify because: $response"
                    fi
                fi
            fi
        done
    else
        notify-send -u "$urgency" -a 'mdadm' -i 'drive-harddisk' -c $category -t $timeout "$summary" "$body"
    fi
}

EVENT=$1
ARRAY=$2
DEVICE=$3

case "$EVENT" in
    'DeviceDisappeared')
        send_message 'critical' "Array $ARRAY has disappeared"
        ;;
    'RebuildStarted')
        send_message 'normal' "Rebuild was started for $ARRAY"
        ;;
    'RebuildFinished')
        send_message 'normal' "Rebuild for $ARRAY has completed" 'Please note that it <b>may have succeeded</b> or <b>have failed</b>. You need to inspect the status of the array by hand'
        ;;
    'Rebuild'[0-9][0-9])
        percent=$(echo "$EVENT" | cut -c8,9)
        send_message 'normal' "Rebuild of $ARRAY at ${percent}%"
        ;;
    'Fail')
        send_message 'critical' "$ARRAY has marked device $DEVICE as faulty"
        ;;
    'FailSpare')
        send_message 'critical' "Rebuild failed for $DEVICE" "Failed to replace a faulty device in $ARRAY"
        ;;
    'SpareActive')
        send_message 'normal' "Rebuild succesfull for $DEVICE" "Replaced a faulty device in $ARRAY"
        ;;
    'NewArray')
        send_message 'normal' "New array $ARRAY has been detected"
        ;;
    'DegradedArray')
        send_message 'critical' "Array $ARRAY is degraded"
        ;;
    'MoveSpare')
        send_message 'normal' "Spare from $DEVICE was moved to $ARRAY"
        ;;
    'SpareMissing')
        send_message 'critical' "Spare(s) missing from $ARRAY" 'There are less spares available than required by its configuration'
        ;;
    'TestMessage')
        send_message 'low' "Found array $ARRAY at startup (test)"
        ;;
    *)
        logger -t 'mdadm-notify' "Unhandled message of type: $EVENT with arguments $ARRAY $DEVICE"
esac

exit 0
