---
title: Getting started with the LHT65 LoRaWAN sensor
description: from the bottom to the top
date: 2021-12-30
tldr: lht65 helium network setup instructions
draft: false
tags: [rf, helium, lora]
---
After setting up my MNTD Helium Hotspot, I wanted to start actually utilizing the network I was helping to build.

Enter the [Dragino LHT65 Temperature/Humidity Sensor](https://www.robotshop.com/en/dragino-lht65-lorawan-temperature-humidity-sensor-915-mhz.html). I had a hard time finding good information on LoRaWAN sensors and the like, but this model stood out to me for the external sensor connector. I was also very happy with the supplier (RobotShop) and will be ordering more from them in the future. 

## Overview
You'll want to make an account on the [Helium Console](https://console.helium.com). You may also want an account on [myDevices Cayenne](https://cayenne.mydevices.com).

1. Wake the sensor from deep sleep mode and connect the external temperature probe
2. Create a new device on the Helium Console with the proper keys
3. Link your new device to an integration

## Unboxing
When you first open the box, you are presented with several long strings and QR codes. Fear not! Typing them in by hand may still be in your future. These keys are necessary to associate your Helium account with your device and provide secure communication. Keep them secret accordingly. There are 3 in particular that we care about.

- **Device EUI** - 64-bit end-device identifier, sometimes called Manufacturer EUI
- **App EUI** - 64-bit application identifier
- **App Key** - 128-bit AES key, used to secure communication between device and network

You will need to depress the button on the bottom left of the device (also called the ACT button) for more than 3 seconds to activate the device. On successful activation, the green LED will blink 5 times indicating that the device has entered 'working' mode and will now attempt to join the network.

## Provisioning
Navigate to the [add new device](https://console.helium.com/devices/new) page on the Helium Console and enter the information from the previous section below. Be sure it is correct, as errors here combined with a long initial block time (up to 20 minutes or so) will only lead to frustration. You will also need to give your device a name. Save your device when you are finished. At this point, you will soon see packets coming in from your device on its page in the dashboard.

## Integration
The first tool I tried is [myDevices Cayenne](https://cayenne.mydevices.com). It is decent, but has some quirks. Setting up this integration is relatively simple.

Ideally, I could dispatch a webhook straight from the Helium Console to one of my own applications. Surely there is already an easy path?

### Google Sheets
Helium already provides an integration for Google Sheets. This is pretty easy to set up and they include some [instructions](https://docs.helium.com/use-the-network/console/integrations/google-sheets/) via their documentation. Essentially, you create a Google Form with the fields you want to capture, then provide that form's ID to the integration page in the console. The console then generates some mapping functions for your form and provides you with a template for a decoder function.

A custom decoder is needed for this method. While this decoder won't give you the nice RF information like the packet SNR, it reports the device's current battery voltage as well as all 3 environmental sensors. (2 onboard and 1 external)
```
function Decoder(bytes, port) {
  var decodedPayload = {
    "bat-v": ((bytes[0]<<8 | bytes[1]) & 0x3FFF)/1000,
    "temp-c-sht": ((bytes[2]<<24>>16 | bytes[3])/100).toFixed(2),
    "hum-sht": ((bytes[4]<<8 | bytes[5])/10).toFixed(1),
    "temp-c-ds": ((bytes[7]<<24>>16 | bytes[8])/100).toFixed(2),
  };

  return Serialize(decodedPayload)
}
```
