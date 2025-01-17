---
title: "Network Device Selector"
description: "How to configure network devices by selecting them using hardware information"
aliases:
  - ../../guides/device-selector
---

## Configuring Network Device Using Device Selector

`deviceSelector` is an alternative method of configuring a network device:

```yaml
machine:
  ...
  network:
    interfaces:
      - deviceSelector:
          driver: virtio
          hardwareAddr: "00:00:*"
        address: 192.168.88.21
```

Selector has the following traits:

- qualifiers match a device by reading the hardware information in `/sys/class/net/...`
- qualifiers are applied using logical `AND`
- `machine.network.interfaces.deviceConfig` option is mutually exclusive with `machine.network.interfaces.interface`
- the selector is invalid when it matches multiple devices, the controller will fail and won't create any devices for the malformed selector

The available hardware information used in the selector can be observed in the `LinkStatus` resource (works in maintenance mode):

```yaml
# talosctl get links eth0 -o yaml
spec:
  ...
  hardwareAddr: 4e:95:8e:8f:e4:47
  busPath: 0000:06:00.0
  driver: alx
  pciID: 1969:E0B1
```

## Using Device Selector for Bonding

Device selectors can be used to configure bonded interfaces:

```yaml
machine:
  ...
  network:
    intefaces:
      - interface: bond0
        bond:
          mode: balance-rr
          deviceSelectors:
            - hardwareAddr: '00:50:56:8e:8f:e4'
            - hardwareAddr: '00:50:57:9c:2c:2d'
```

In this example, the `bond0` interface will be created and bonded using two devices with the specified hardware addresses.

## Use Case

`machine.network.interfaces.interface` name is generated by the Linux kernel and can be changed after a reboot.
Device names can change when the system has several interfaces of the same kind, e.g: `eth0`, `eth1`.

In that case pinning it to `hardwareAddress` will make Talos reliably configure the device even when interface name changes.
