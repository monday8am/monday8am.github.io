---
layout: post
title:  "Fidesmo Kiosk app"
date:   2019-03-30 19:00:00 +0100
categories: project
---

[Fidesmo](https://www.fidesmo.com/) is a company that created a [whole platform](https://developer.fidesmo.com/) to digitize and connect payment cards and other contactless services to smart [wearables](https://www.fidesmo.com/fidesmo/devices/), and to fill these wearables with digital services. One important part of the company is the [Fidesmo Pay](https://fidesmo.com/pay) technology, that let's final users to install their payment card in Fidesmo enabled devices.

As part of Fidesmo Pay, Fidesmo provides a *Kiosk* in order to let retailers to tokenise the products their sell. The Kiosk is configured by an iPad and NFC/BLE reader and let final users to put their payment card into the product they bought, for example, a strap, a watch or a ring.

### Architecture

Although in principle this project looks simple, the architecture is affected by an initial requirement: it has to run on iOS devices. 

Because iOS system doesn't support enough NFC features as needed by the Fidesmo Platform, an extra peripheral is needed in order to stablish a secure channel between the Fidesmo device and the servers. This peripheral is the NFC/BLE reader [ACS ACR1255](https://www.acs.com.hk/en/products/403/acr1255u-j1-secure-bluetooth%C2%AE-nfc-reader/)

The app establishes a BLE connection with the reader and the reader establishes an NFC connection with the Fidesmo device. Two connections are managed at the same time, together with the connection with the API.

As in the Android app, a [_Redux_ like framework](https://github.com/ReSwift/ReSwift) is used to managed the state in a predictable and testable way.

### UX/UI

The Kiosk presents a very simple and intuitive interface to let the user follow the whole steps in a clear way.

![Screenshot]({{ "/assets/projects/fidesmo-ui.png" | absolute_url }})

### Ecosystem

The app is integrated into a continuos integration pipeline built over _Travis_, _Firebase_ and _fastlane_ that provides a fast release pace of new features and bugs fixed.

### Services

- Frontend development and maintenance
- Content creation
- UX/UI design collaboration

### Related links

- [Fidesmo Kiosk at Apple Store](https://itunes.apple.com/us/app/fidesmo-kiosk-app/id1454792665)
- [Fidesmo Android App](http://monday8am.com/project/2018/11/10/fidesmo-app.html)
- [Fidesmo Pay](https://fidesmo.com/pay)
- [Unidirectional flow to tame the state](http://monday8am.com/blog/2018/03/30/unidirectional.html)
