# libONVIF

[![Conan Remote Recipe](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fapi.github.com%2Frepos%2FPrivatehive%2FlibONVIF%2Fproperties%2Fvalues&query=%24%5B0%5D.value&style=flat&logo=conan&label=conan&color=%232980b9)](https://conan.privatehive.de/ui/repos/tree/General/public-conan/de.privatehive/libonvif)

### Yet another ONVIF library

---

| os        | arch     | CI Status                                                                                                                                                                                                                                                 |
|-----------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Linux`   | `x86_64` | [![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/Privatehive/libONVIF/main.yml?branch=master&style=flat&logo=github&label=create+package)](https://github.com/Privatehive/libONVIF/actions?query=branch%3Amaster) |
| `Windows` | `x86_64` | [![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/Privatehive/libONVIF/main.yml?branch=master&style=flat&logo=github&label=create+package)](https://github.com/Privatehive/libONVIF/actions?query=branch%3Amaster) |
| `Android` | `armv8`  | [![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/Privatehive/libONVIF/main.yml?branch=master&style=flat&logo=github&label=create+package)](https://github.com/Privatehive/libONVIF/actions?query=branch%3Amaster) |

What does ONVIF stand for:

> ONVIF (Open Network Video Interface Forum) is a global and open industry forum with the goal of facilitating the
> development and use of a global open standard for the interface of physical IP-based security products – or, in other
> words, to create a standard for how IP products within video surveillance and other physical security areas can
> communicate with each other. [Wikipedia](https://en.wikipedia.org/wiki/ONVIF)

The idea behind this library is to hide some complexity of gsoap and to provide 'high level' classes including QT
goodness. Currently there are eleven client side service methods implemented:

- ONVIF analytics http://www.onvif.org/ver20/analytics/wsdl
- ONVIF device http://www.onvif.org/ver10/device/wsdl
- ONVIF display http://www.onvif.org/ver10/display/wsdl
- ONVIF event http://www.onvif.org/ver10/events/wsdl
- ONVIF imaging http://www.onvif.org/ver20/imaging/wsdl
- ONVIF media http://www.onvif.org/ver10/media/wsdl
- ONVIF media2 http://www.onvif.org/ver20/media/wsdl
- ONVIF ptz http://www.onvif.org/ver20/ptz/wsdl
- ONVIF receiver http://www.onvif.org/ver10/receiver/wsdl
- ONVIF recording http://www.onvif.org/ver10/recording/wsdl
- ONVIF replay http://www.onvif.org/ver10/replay/wsdl

For every service exists a class following the naming scheme `Onvif*Client` (\* matches service name). These classes
handle the RPCs. For the convenience of WS discovery and ONVIF (pullpoint) event handling there are two more
classes: `OnvifDiscovery` and `OnvifPullPoint`.

Design thoughts:

- The most of this library is thread safe and should work in a multithreaded environment
- gsoap is 'hidden' from the user as much as possible
- RAII classes `Request<>`, `Response<>` wrapping the RPC parameters are responsible for the memory management

### Further reading

- The library comes with a small application (named ovifinfo) that does some device-discovery, inspection. You may want
  to look at the [source code](https://github.com/Tereius/libONVIF/blob/master/src/main.cpp).
- If you want to learn more about how to use libONVIF in a qml app take a look at the following
  project [ONVIFMonitor](https://github.com/Tereius/ONVIFMonitor).
- It is always advisable to look at
  the [ONVIF programmers guide](https://www.onvif.org/wp-content/uploads/2016/12/ONVIF_WG-APG-Application_Programmers_Guide-1.pdf)
  to learn more about ONVIF.
