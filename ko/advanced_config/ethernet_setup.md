# PX4 Ethernet Setup

Ethernet connectivity provides a fast, reliable, and flexible communication alternative to using USB or other serial connections.

It can be used to connect to ground stations, companion computers, and other MAVLink systems. It is particularly recommended when connecting to systems that "natively" use Ethernet - for example IP radios.

This topic covers:
- [PX4 Ethernet Setup](#px4-ethernet-setup)
  - [Supported Flight Controllers](#supported-flight-controllers)
  - [Setting up the Ethernet Network](#setting-up-the-ethernet-network)
    - [PX4 Ethernet Network Setup](#px4-ethernet-network-setup)
    - [Ubuntu Ethernet Network Setup](#ubuntu-ethernet-network-setup)
    - [Companion Computer Ethernet Network Setup](#companion-computer-ethernet-network-setup)
  - [PX4 MAVLink Serial Port Configuration](#px4-mavlink-serial-port-configuration)
  - [QGroundControl Setup Example](#qgroundcontrol-setup-example)
  - [MAVSDK-Python Setup Example](#mavsdk-python-setup-example)
  - [ROS2 Setup Example](#ros2-setup-example)

## Supported Flight Controllers

PX4 supports Ethernet connectivity on [Pixhawk 5X-standard](https://github.com/pixhawk/Pixhawk-Standards/blob/master/DS-011%20Pixhawk%20Autopilot%20v5X%20Standard.pdf) flight controllers (and later) that have an Ethernet port. It may also be supported on other boards.

Supported flight controllers include:

 - [CUAV Pixhawk V6X](../flight_controller/cuav_pixhawk_v6x.md)
 - [Holybro Pixhawk 5X](../flight_controller/pixhawk5x.md)
 - [Holybro Pixhawk 6X](../flight_controller/pixhawk6x.md)

## Setting up the Ethernet Network

To connect systems over Ethernet you need to configure them to run on the same IP network, so that each system has a unique IP address and can find the other systems. This might be done using a DHCP server to allocate addresses, or by manually configuring the addresses of each system on the network.

There is no single "out of the box configuration" that we can provide that will necessarily work in your local network. Therefore as an example of the kind of configuration you might do, below we show how to set up the systems on an IP network with static addresses in the range `192.168.0.Xxx`, where PX4 has a statically allocated address `192.168.0.4` and the computer has address `192.168.0.1`. If you wanted to connect a companion computer or other system to the network you could use a similar approach to allocate a static address.

:::note
There is nothing "special" about the network configuration (other than perhaps the tools used to modify the network settings); it works much the same as any home or business network.
Which is to say that a knowledge of how IP networks work is highly desirable!
:::

### PX4 Ethernet Network Setup

<!-- Information about NuttX network manager: https://github.com/PX4/PX4-Autopilot/pull/16330 -->

PX4 uses the [netman](../modules/modules_system.md#netman) module to apply and update network settings.

Network settings are defined in the configuration file `/fs/microsd/net.cfg` on the SD card. This is a text file, that defines each setting on a new line as a `name=value` pair. A configuration file might look like this:
```
DEVICE=eth0
BOOTPROTO=fallback
IPADDR=192.168.0.4
NETMASK=255.255.255.0
ROUTER=192.168.0.254
DNS=192.168.0.254
```

Where the values are:
- `DEVICE`: Interface name. Default is `eth0`.
- `BOOTPROTO`: Protocol for getting PX4 IP address. Valid values for proto are: `dhcp`, `static`, `fallback` (use DHCP but fall back to static address after time, if that fails)
- `IPADDR`: Static IP address (used if BOOTPROTO is `static` or `fallback`)
- `NETMASK`: Network mask
- `ROUTER`: The address of the default route.
- `DNS`: The address of the DNS server.

To set the above "example" configuration using the *QGroundControl*:

1. Connect the flight controller to the computer with the USB cable.
1. Open **QGroundcontrol > Analyze Tools > MAVLink Console**
1. Enter commands "like" the ones below into the *MAVLink Console* (to write the values to the configuration file):
   ```
   echo DEVICE=eth0 > /fs/microsd/net.cfg
   echo BOOTPROTO=fallback >> /fs/microsd/net.cfg
   echo IPADDR=192.168.0.4 >> /fs/microsd/net.cfg
   echo NETMASK=255.255.255.0 >>/fs/microsd/net.cfg
   echo ROUTER=192.168.0.254 >>/fs/microsd/net.cfg
   echo DNS=192.168.0.254 >>/fs/microsd/net.cfg
   ```
1. Once the network configuration has been set you can disconnect the USB cable.
1. Reboot the flight controller to apply the settings.

Note that the above setup gives the flight controller an address on the Ethernet network. You also need to \[configure the Ethernet port\]((#px4-mavlink-serial-port-configuration) to use MAVLink.


### Ubuntu Ethernet Network Setup

If you're using Ubuntu for your ground station (or companion computer) then you can use [netplan](https://netplan.io/) to configure the network.

Below we show how you write a setup to the netplan configuration file "`/etc/netplan/01-network-manager-all.yaml`", which would run on the same network as used by the PX4 setup above. Note that there are many more [examples](https://netplan.io/examples/) and instructions in the [netplan](https://netplan.io/) documentation.

To setup the Ubuntu Computer:

1. In a terminal, create and open a `netplan` configuration file: `/etc/netplan/01-network-manager-all.yaml` Below we do this using the *nano* text editor.

   ```
   sudo nano /etc/netplan/01-network-manager-all.yaml 
   ```

1. Copy and paste the following configuration information into the file (note: the indentations are important!):
   ```
   network:
     version: 2
     renderer: NetworkManager
     ethernets:
         enp2s0:
             addresses:
                 - 192.168.0.1/24
             nameservers:
                 addresses: [192.168.0.1]
             routes:
                 - to: 192.168.0.1
                   via: 192.168.0.1
   ```

   Save and exit the editor.
1. Apply the *netplan* configuration by entering the following command into the Ubuntu terminal.
   ```
   sudo netplan apply
   ```


### Companion Computer Ethernet Network Setup

The setup for a companion computer will depend on the companion computer's operating system.

A Linux operating system may support `netplan`, in which case the instructions would be the same as above, but using a unique IP address.


## PX4 MAVLink Serial Port Configuration

The Ethernet port configuration sets the properties of the _serial link_ (which is how PX4 views the Ethernet connection). This includes the set of MAVLink messages that are streamed, the data rate, the UDP ports that a remote system can connect listen to, etc.

:::note
You must separately configure the PX4 IP address and other _network settings_ ([as shown previously](#px4-ethernet-network-setup)).
:::

PX4 configures the serial port to connect to a GCS via MAVLink, using the parameters shown below:

| Parameter                                                                        | Value  | Description                                                  |
| -------------------------------------------------------------------------------- | ------ | ------------------------------------------------------------ |
| [MAV_2_CONFIG](../advanced_config/parameter_reference.md#MAV_2_CONFIG)         | 1000   | Configure Ethernet port                                      |
| [MAV_2_BROADCAST](../advanced_config/parameter_reference.md#MAV_2_BROADCAST)   | 1      | Broadcast `HEARTBEAT` messages                               |
| [MAV_2_MODE](../advanced_config/parameter_reference.md#MAV_2_MODE)             | 0      | Send the "normal" set of MAVLink messages (i.e. the GCS set) |
| [MAV_2_RADIO_CTL](../advanced_config/parameter_reference.md#MAV_2_RADIO_CTL)   | 0      | Disable software throttling of MAVLink traffic               |
| [MAV_2_RATE](../advanced_config/parameter_reference.md#MAV_2_RATE)             | 100000 | Maximum sending rate                                         |
| [MAV_2_REMOTE_PRT](../advanced_config/parameter_reference.md#MAV_2_REMOTE_PRT) | 14550  | MAVLink Remote Port of 14550 (GCS)                           |
| [MAV_2_UDP_PRT](../advanced_config/parameter_reference.md#MAV_2_UDP_PRT)       | 14550  | MAVLink Network Port of 14550 (GCS)                          |

Normally a companion computer would use port `14540` (rather than `14550`) and stream the set of MAVLink messages specified in the `Onboard` profile. You can configure this setup by changing [MAV_2_REMOTE_PRT](../advanced_config/parameter_reference.md#MAV_2_REMOTE_PRT) and [MAV_2_UDP_PRT](../advanced_config/parameter_reference.md#MAV_2_UDP_PRT) to `14540` and [MAV_2_MODE](../advanced_config/parameter_reference.md#MAV_2_MODE) to `2` (Onboard). Note however that this will still work using the GCS profile.

For more information on MAVLink serial port configuration see [MAVLink Peripherals (GCS/OSD/Companion)](../peripherals/mavlink_peripherals.md)


## QGroundControl Setup Example

Assuming you have already [Set up the Ethernet Network](#setting-up-the-ethernet-network) so your ground station computer and PX4 run on the same network, and

To connect QGroundControl to PX4 over Ethernet:
1. [Set up the Ethernet Network](#setting-up-the-ethernet-network) so your ground station computer and PX4 run on the same network.
1. Connect the ground station computer and PX4 using an Ethernet cable.
1. Start QGroundControl and [define a comm link](https://docs.qgroundcontrol.com/master/en/SettingsView/SettingsView.html) (**Application Settings > Comm Links**) specifying the _server address_ and port as the IP address and port assigned in PX4, respectively.

   Assuming that the values are set as described in the rest of this topic the setup will look like this:

   ![QGC comm link for ethernet setup](../../assets/qgc/settings/comm_link/px4_ethernet_link_config.png)

1. QGroundControl should then connect if you select this link.

:::note
[PX4 Ethernet Port Configuration](#px4-ethernet-network-setup) should not be needed (the default are appropriate for a GCS).
:::

## MAVSDK-Python Setup Example

To setup MAVSDK-Python running on a companion computer:

1. [Set up the Ethernet Network](#setting-up-the-ethernet-network) so your companion computer and PX4 run on the same network.
1. Modify the [PX4 Ethernet Port Configuration](#px4-ethernet-network-setup) to connect to a companion computer. You might change the parameters [MAV_2_REMOTE_PRT](../advanced_config/parameter_reference.md#MAV_2_REMOTE_PRT) and [MAV_2_UDP_PRT](../advanced_config/parameter_reference.md#MAV_2_UDP_PRT) to `14540`, and [MAV_2_MODE](../advanced_config/parameter_reference.md#MAV_2_MODE) to `2` (Onboard).
1. Follow the instructions in [MAVSDK-python](https://github.com/mavlink/MAVSDK-Python) to install and use MAVSDK.

   For example, your code will connect to the PX4 using:

   ```python
   await drone.connect(system_address="udp://192.168.0.4:14540")
   ```

:::note MAVSDK
can connect to the PX4 on port `14550` if you don't modify the PX4 Ethernet port configuration. However this is not recommended because the default configuration is optimised for communicating with a GCS (not a companion computer).
:::


## ROS2 Setup Example

Prerequisites:

- You have a supported autopilot hardware with RTPS feature enabled firmware on it by using [this guide](../middleware/micrortps.md#client-px4-px4-autopilot).
- [ROS2](../ros/ros2_comm.md#sanity-check-the-installation) has been set up correctly and [sanity check](../ros/ros2_comm.md#sanity-check-the-installation) has been confirmed.
- You have followed the Ethernet network and port setup as discussed at the top of this page.

In this example it is assumed that you have followed the example to set your IP addresses.

1. Connect your Flight controller via Ethernet
2. Open **QGroundControl > Analyze Tools > MAVLink Console**
3. Enter the command below to start the micro_rtps client on your flight controller. Note that the remote IP here is your companion computer IP. This by default starts the micrortps_client connected to UDP ports 2019 and 2020 To make changes you can take a look at [RTPS guide](../middleware/micrortps.md#client-px4-px4-autopilot)
   ```
   micrortps_client start -t UDP -i <remote IP>
   ```
   An output like below is expected in the console:
   ```
   INFO  [micrortps_client] UDP transport: ip address: 192.168.0.1; recv port: 2019; send port: 2020
   INFO  [micrortps_client] UDP transport: Trying to connect...
   INFO  [micrortps_client] UDP transport: Connected to server!
   ```
5. Then we need to run the agent by typing the below commands in a new terminal on either our Linux computer. This will start the agent on `localhost` which is `127.0.0.1`.
   ```
   $ source ~/px4_ros_com_ros2/install/setup.bash
   $ micrortps_agent start -t UDP
   ```
6. In a new terminal you can run a listener node to confirm the connection is established:
   ```
   $ source ~/px4_ros_com_ros2/install/setup.bash
   $ ros2 launch px4_ros_com sensor_combined_listener.launch.py
   ```

If everything goes ok and there is an established connection you can see the output below in your terminal:
```
RECEIVED SENSOR COMBINED DATA
=============================
ts: 855801598
gyro_rad[0]: -0.00339938
gyro_rad[1]: 0.00440091
gyro_rad[2]: 0.00513893
gyro_integral_dt: 4997
accelerometer_timestamp_relative: 0
accelerometer_m_s2[0]: -0.0324082
accelerometer_m_s2[1]: 0.0392213
accelerometer_m_s2[2]: -9.77914
accelerometer_integral_dt: 4997
```
