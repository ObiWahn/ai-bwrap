# Filtering with xdg-dbus-proxy

`xdg-dbus-proxy` is a tool used to filter and mediate access to D-Bus services, often used in sandboxing environments like Flatpak. It creates a proxy between an application and the D-Bus system/session buses, allowing or denying specific method calls and signals based on filter rules.

## Basic Usage

```sh
xdg-dbus-proxy <bus> <proxy> <policy> [options]
```

- `<bus>`: Path to the original D-Bus socket (e.g., `/run/user/1000/bus`).
- `<proxy>`: Path to the new proxy socket.
- `<policy>`: Path to a policy file defining the filtering rules.

### Example: Filtering a Session Bus
```sh
xdg-dbus-proxy /run/user/1000/bus /tmp/proxied-bus allow-rules.txt &
```
This sets up a filtered proxy bus at `/tmp/proxied-bus`.

> **Note:** `xdg-dbus-proxy` must remain running while your application is executing. If it stops, the proxied D-Bus socket will no longer function.

To keep `xdg-dbus-proxy` running in the background, use `&` at the end of the command. Alternatively, run it in another terminal or use a session manager like `tmux` or `screen`.

## Writing Filter Rules

Rules in the policy file define what services, interfaces, and methods are allowed or denied.

### Example Policy File (`allow-rules.txt`)

```
talk org.freedesktop.Notifications
own org.example.MyApp
```

- `talk <service>` → Allows sending messages to a specific service.
- `own <service>` → Allows owning a specific service on the bus.
- `monitor` → Allows monitoring all D-Bus traffic.
- `none` → Denies everything.

### Example: Filtering Specific Methods
To only allow method calls to `org.freedesktop.Notifications` but block others:
```
talk org.freedesktop.Notifications
none
```

This blocks all other communication except the allowed one.

## Running an App with a Filtered Bus
You can run an application with the filtered bus by setting `DBUS_SESSION_BUS_ADDRESS`:
```sh
DBUS_SESSION_BUS_ADDRESS=unix:path=/tmp/proxied-bus some-app
```

## Debugging D-Bus Calls with xdg-dbus-proxy
You can run your app through `xdg-dbus-proxy` with logging enabled to see what it tries to access:

```sh
xdg-dbus-proxy /run/user/1000/bus /tmp/proxied-bus log-rules.txt --log debug > dbus-log.txt &
```

Then set:
```sh
DBUS_SESSION_BUS_ADDRESS=unix:path=/tmp/proxied-bus your-app
```

Check `dbus-log.txt` for messages.

## Example Bash Script for Managing xdg-dbus-proxy

Below is a script that starts `xdg-dbus-proxy` with a unique bus path, ensures the path is not already taken, and properly cleans up when terminated:

```sh
#!/bin/bash

# Generate a unique 8-digit random suffix
RANDOM_SUFFIX=$(shuf -i 10000000-99999999 -n 1)
PROXY_BUS="/run/user/$(id -u)/dbus-proxy-$RANDOM_SUFFIX"

# Ensure the proxy bus path is not already taken
while [ -e "$PROXY_BUS" ]; do
    RANDOM_SUFFIX=$(shuf -i 10000000-99999999 -n 1)
    PROXY_BUS="/run/user/$(id -u)/dbus-proxy-$RANDOM_SUFFIX"
done

# Start xdg-dbus-proxy
xdg-dbus-proxy /run/user/$(id -u)/bus "$PROXY_BUS" allow-rules.txt &
PROXY_PID=$!

# Set up trap to clean up on exit
cleanup() {
    echo "Stopping xdg-dbus-proxy..."
    kill $PROXY_PID
    rm -f "$PROXY_BUS"
}
trap cleanup EXIT

# Export the proxy bus for the application
export DBUS_SESSION_BUS_ADDRESS="unix:path=$PROXY_BUS"

# Run the application (replace 'your-app' with the actual application)
your-app

# Wait for the application to exit
wait $PROXY_PID
```

This script:
1. Generates a unique 8-digit identifier for the D-Bus proxy path.
2. Ensures the generated path is not already taken.
3. Starts `xdg-dbus-proxy` in the background.
4. Sets up a `trap` to clean up the process on exit.
5. Exports the new D-Bus proxy path for the application.
6. Runs the application and waits for it to exit before stopping `xdg-dbus-proxy`.

Use this script to reliably manage `xdg-dbus-proxy` without manual cleanup.
