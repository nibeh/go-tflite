# go-tflite

Go binding for TensorFlow Lite

![](https://raw.githubusercontent.com/mattn/go-tflite/master/screenshots/screenshot.png)

## Usage

```go
model := tflite.NewModelFromFile("sin_model.tflite")
if model == nil {
	log.Fatal("cannot load model")
}
defer model.Delete()

options := tflite.NewInterpreterOptions()
defer options.Delete()

interpreter := tflite.NewInterpreter(model, options)
defer interpreter.Delete()

interpreter.AllocateTensors()

v := float64(1.2) * math.Pi / 180.0
input := interpreter.GetInputTensor(0)
input.Float32s()[0] = float32(v)
interpreter.Invoke()
got := float64(interpreter.GetOutputTensor(0).Float32s()[0])
```

See `_example` for more examples

## Requirements

* TensorFlow Lite

## Installation

At the first, you must install Tensorflow Lite C API. The repository of tensorflow must be located on `$GOPATH/src/github.com/tensorflow/tensorflow`.

```
$ cd /path/to/gopath/src/github.com/tensorflow/tensorflow
$ bazel build --config opt --config monolithic tensorflow:libtensorflow_c.so
```

Then go build on go-tflite. If you don't love bazel, you can use this Makefile. Put this on the `tensorflow/lite/experimental/c`, and `make`. Sorry, I don't make sure this works on Linux or Mac.

```make
SRCS = \
        c_api.cc \
        c_api_experimental.cc

OBJS = $(subst .cc,.o,$(subst .cxx,.o,$(subst .cpp,.o,$(SRCS))))

TENSORFLOW_ROOT = $(shell go env GOPATH)/src/github.com/tensorflow/tensorflow
CXXFLAGS = -DTF_COMPILE_LIBRARY -I$(TENSORFLOW_ROOT) -I$(TENSORFLOW_ROOT)/tensorflow/lite/tools/make/downloads/flatbuffers/include
TARGET = libtensorflowlite_c
ifeq ($(OS),Windows_NT)
OS_ARCH = windows_x86_64
TARGET_SHARED := $(TARGET).dll
else
ifeq ($(shell uname -m),x86_64)
OS_ARCH = linux_x86_64
else
ifeq ($(shell uname -m),armv6l)
OS_ARCH = linux_armv6l
else
OS_ARCH = rpi_armv7l
endif
TARGET_SHARED := $(TARGET).so
endif
endif
LDFLAGS += -L$(TENSORFLOW_ROOT)/tensorflow/lite/tools/make/gen/$(OS_ARCH)/lib
LIBS = -ltensorflow-lite

.SUFFIXES: .cpp .cxx .o

all : $(TARGET_SHARED)

$(TARGET_SHARED) : $(OBJS)
        g++ -shared -o $@ $(OBJS) $(LDFLAGS) $(LIBS)

.cxx.o :
        g++ -std=c++14 -c $(CXXFLAGS) -I. $< -o $@

.cpp.o :
        g++ -std=c++14 -c $(CXXFLAGS) -I. $< -o $@

clean :
        rm -f *.o $(TARGET_SHARED)
```

## License

MIT

## Author

Yasuhrio Matsumoto (a.k.a. mattn)
