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

```python
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

# ***************************************************************
# Initialize Fifos (first in first out)
# ***************************************************************
fifoIn = mvnc.Fifo("fifoIn0", mvnc.FifoType.HOST_WO)
fifoOut = mvnc.Fifo("fifoOut0", mvnc.FifoType.HOST_RO)

descIn = graph.get_option(mvnc.GraphOption.RO_INPUT_TENSOR_DESCRIPTORS)
descOut = graph.get_option(mvnc.GraphOption.RO_OUTPUT_TENSOR_DESCRIPTORS)

fifoIn.allocate(device, descIn[0], 2)
fifoOut.allocate(device, descOut[0], 2)

# ***************************************************************
# Send the image to the NCS
# ***************************************************************
graph.queue_inference_with_fifo_elem(fifoIn, fifoOut, img, 'user object')

# ***************************************************************
# Get the result from the NCS
# ***************************************************************
output, userobj = fifoOut.read_elem()

# ***************************************************************
# Clean up the graph and the device
# ***************************************************************
fifoIn.destroy()
fifoOut.destroy()
graph.destroy()
device.close()
```
