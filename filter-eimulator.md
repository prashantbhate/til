
I needed to list and filter the iOS emulators so that I could use it in the github actions

credits to jq and -j (json) option

````
#!/bin/bash

get_ios_devices() {
    local ios_version="$1"
    local device_model="$2"

    xcrun simctl list devices -j | jq -r --arg version "$ios_version" --arg model "$device_model" '
        .devices | 
        to_entries[] | 
        select(.key | contains("SimRuntime.iOS-\($version)"))
        |. as $parent |
         .value[]|select(
            (.name | contains($model)) and 
            (.isAvailable == true)
        )| "\(.udid),\(.name),(iOS \($parent.key|split("SimRuntime.iOS-")[1] ))"
    '
}

if [ $# -ne 2 ]; then
    echo "Usage: $0 <ios_version> <device_model>"
    echo "Example: $0 18-1 '16 Pro'"
    exit 1
fi
get_ios_devices "$1" "$2"

````
