# rpi-ncsdk-docker
_In this tutorial we're going to build Intel [Intel® Movidius™ Neural Compute SDK](https://github.com/movidius/ncsdk) docker image for Raspberry Pi on x86 machine._

1. First, install cross-compiling tool for building arm image on x86 machine. `./qemu-arm-static` is provided for Ubuntu 18.04, but if this does not work, run 

```sh
git clone --recursive git@github.com:duckietown/rpi-ncsdk-docker.git && \
    cd rpi-ncsdk-docker && \
    sudo apt-get install qemu-user-static && \
    cp /usr/bin/qemu-arm-static .
```

2. To build new docker image, run this command bellow on your favorite terminal

```sh
docker build --rm -f "Dockerfile" -t duckietown/rpi-ncsdk-docker .
```

3. Run newly-built docker image via 

```sh
docker run --net=host \
           --privileged \
           -v /dev/bus/usb/:/dev/bus/usb/:ro \
           -v /usr/bin/qemu-arm-static:/usr/bin/qemu-arm-static:ro,rslave \
           --name ncsdk -i -t  \
           duckietown/rpi-ncsdk-docker /bin/bash
```

# How to load and run the graph file

```sh
from mvnc import mvncapi

# Get a list of valid device identifiers
device_list = mvncapi.enumerate_devices()

# Create a Device instance for the first device found
device = mvncapi.Device(device_list[0])

# Open communication with the device
device.open()

# Create a Graph
graph = mvncapi.Graph('graph1')

# Read a compiled network graph from file (set the graph_filepath correctly for your graph file)
graph_filepath = './graph'
with open(graph_filepath, 'rb') as f:
    graph_buffer = f.read()

# Allocate the graph on the device
graph.allocate(device, graph_buffer)

#
# Use the device...
#

# Deallocate and destroy the graph handle, close the device, and destroy the device handle
graph.destroy()
device.close()
device.destroy()
'''
