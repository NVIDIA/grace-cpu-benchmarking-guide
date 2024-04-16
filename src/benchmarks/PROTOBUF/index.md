# protobuf 

Protocol Buffers (a.k.a., protobuf) are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data.

## Build

The following script will build Protocol Buffers. The script tested on a freshly booted Ubuntu 22.04. 

```bash
#!/bin/bash

set -e

sudo apt update && sudo apt install -y autoconf automake libtool curl make g++ unzip libz-dev git cmake
git clone https://github.com/protocolbuffers/protobuf.git
pushd protobuf
# syncing at a specific commit
git checkout 7cd0b6fbf1643943560d8a9fe553fd206190b27f
git submodule update --init --recursive
./autogen.sh
./configure
make
make check 
sudo make install
sudo ldconfig
pushd benchmarks
make cpp
popd
popd
```

## Running Benchmarks on Grace

```bash
#!/bin/bash
pushd ./protobuf
mkdir -p result
rm -rf result/*
C=$(nproc)
for (( i=0; i < $C-1; i++ ))
do
        filename_result="$i.log"
        filepath_result="result/$filename_result"
        taskset -c $i ./benchmarks/cpp-benchmark --benchmark_min_time=5.0 $(find $(cd . && pwd) -type f -name "dataset.*.pb" -not -path "$(cd . && pwd)/tmp/*") >> $filepath_result &
done
#last core will be sync
filename_result="$i.log"
filepath_result="result/$filename_result"
taskset -c $i ./benchmarks/cpp-benchmark --benchmark_min_time=5.0 $(find $(cd . && pwd) -type f -name "dataset.*.pb" -not -path "$(cd . && pwd)/tmp/*") >> $filepath_result 
sleep 1
popd
echo Done!
```
## Reference Results

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

You need to run n copies of the benchmark as indicated above. Next, you need to take geomean of all the scores. Then, pick the least score across all copies. The total score would be Score * copies = Socket score.

```
Geomean of mins: 1291.26863081306
Total score: 185942.682837081 MB/s
```
