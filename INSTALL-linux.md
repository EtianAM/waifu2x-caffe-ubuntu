# Build on Ubuntu 16.04

## Install deps
```
apt install libboost-system-dev libboost-filesystem-dev libboost-thread-dev libopenblas-dev libboost-iostreams-dev libopenblas-dev libhdf5-dev \
git build-essential cmake pkg-config libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev \
protobuf-compiler libgflags-dev libgoogle-glog-dev liblmdb-dev nkf
```

## Build caffe for waifu2x-caffe

```
git clone -b waifu2x-caffe-ubuntu https://github.com/nagadomi/caffe.git lltcggie-caffe
cd lltcggie-caffe
cp Makefile.config.example-ubuntu Makefile.config
# edit Makefile.config
make
```

When using cuDNN, set `USE_CUDNN := 1` in `Makefile.config`.

## Build waifu2x-caffe

(I tested on Ubuntu18.04 + CUDA10.1 + CuDNN 7.5 + GTX 1080)

```sh
git clone -b ubuntu https://github.com/nagadomi/waifu2x-caffe.git
cd waifu2x-caffe
git submodule update --init --recursive

# create symlink to ltcggie-caffe
ln -s ../lltcggie-caffe ./caffe
ln -s ../lltcggie-caffe ./libcaffe

# build
rm -fr build # clean
mkdir build
cd build
cmake .. -DCUDA_NVCC_FLAGS="-D_FORCE_INLINES  -gencode arch=compute_61,code=sm_61 " # sm_61 is for GTX1080
make
ln -s `realpath ./waifu2x-caffe` ../bin
```

When you got `cuDNN Not Found` issue,
```
cuDNN             :   Not Found
```
re-run cmake command. (Not sure, but it can be solved with re-run cmake command).

## Run
```
cd bin
./waifu2x-caffe -p cudnn -m scale -i input.png -o out.png | nkf
./waifu2x-caffe -p cuda -m scale -i input.png -o out.png | nkf
./waifu2x-caffe -p cpu -m scale -i input.png -o out.png | nkf
```

# Docker

build
```
docker build -t waifu2x-caffe-ubuntu .
```

test gpu
```
$ docker run --gpus 0 waifu2x-caffe-ubuntu nvidia-smi
```

help
```
$ docker run --gpus 0 waifu2x-caffe-ubuntu waifu2x-caffe -h
```

run (2x ./query/test.png -> ./query/2x.png )
```
$ docker run --gpus 0 -v `pwd`/query:/images waifu2x-caffe-ubuntu waifu2x-caffe -p cudnn -m scale -i /images/test.png -o /images/2x.png | nkf
```

run with photo model
```
docker run --gpus 0 -v `pwd`/query:/images waifu2x-caffe-ubuntu waifu2x-caffe -p cudnn --model_dir ./models/upconv_7_photo  -m scale -i /images/test.png -o /images/2x.png | nkf
```
