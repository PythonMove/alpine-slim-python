FROM alpine:3.14

LABEL maintainer="PythonMove <pythonmove@gmail.com>"

ENV TENSORFLOW_VERSION="2.6.0" \
    BAZEL_VERSION="3.7.2" \
    JAVA_HOME="/usr/lib/jvm/java-11-openjdk"

RUN apk add --no-cache --update python3 py3-pip py3-numpy py3-numpy-f2py libpng libjpeg-turbo libstdc++ libexecinfo-dev \
    && apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community hdf5-dev \
    && rm -rf /usr/bin/python \
    && ln -s /usr/bin/python3 /usr/bin/python \
    && pip3 install -U --user --no-cache-dir pip setuptools wheel \
    && apk add --no-cache --virtual=.build-deps curl bash openjdk11 build-base gcc g++ linux-headers zip musl-dev patch python3-dev py3-numpy-dev git perl \
    && cd /tmp \
    && curl -qSLO https://github.com/bazelbuild/bazel/releases/download/${BAZEL_VERSION}/bazel-${BAZEL_VERSION}-dist.zip \
    && mkdir bazel-${BAZEL_VERSION} \
    && unzip -qd bazel-${BAZEL_VERSION} bazel-${BAZEL_VERSION}-dist.zip \
    && cd bazel-${BAZEL_VERSION} \
    && wget -q https://github.com/PythonMove/alpine-python3-tensorflow/raw/main/bazel_src/linux-sandbox-pid1.cc -O src/main/tools/linux-sandbox-pid1.cc \
    && env EXTRA_BAZEL_ARGS="--host_javabase=@local_jdk//:jdk" bash ./compile.sh \
    && cp -p output/bazel /usr/bin/ \
    && cd /tmp \
    && wget -q -O - https://github.com/tensorflow/tensorflow/archive/refs/tags/v${TENSORFLOW_VERSION}.tar.gz | tar -xzf - -C /tmp \
    && cd /tmp/tensorflow-${TENSORFLOW_VERSION} \
    && sed -i -e '/define TF_GENERATE_BACKTRACE/d' tensorflow/core/platform/default/stacktrace.h \
    && PYTHON_BIN_PATH=/usr/bin/python \
        PYTHON_LIB_PATH=/usr/lib/python3.9/site-packages \
        CC_OPT_FLAGS="-march=native" \
        TF_NEED_GCP=0 \
        TF_NEED_HDFS=0 \
        TF_NEED_S3=0 \
        TF_ENABLE_XLA=0 \
        TF_NEED_GDR=0 \
        TF_NEED_VERBS=0 \
        TF_NEED_OPENCL=0 \
        TF_NEED_CUDA=0 \
        TF_NEED_MPI=0 \
        TF_NEED_KAFKA=0 \
        TF_NEED_AWS=0 \
        TF_NEED_OPENCL_SYCL=0 \
        TF_DOWNLOAD_CLANG=0 \
        TF_SET_ANDROID_WORKSPACE=0 \
        bash configure \
    && bazel sync \
    && export PROCESS_INC_PATH=$(find /root/.cache/bazel/_bazel_root/ -name "Process.inc*"|grep Unix) \
    && export LLVM_CONFIG_PATH=$(find /root/.cache/bazel/_bazel_root/ -name "config.h.cmake*"|grep llvm/include/llvm) \
    && wget -q https://github.com/PythonMove/alpine-python3-tensorflow/raw/main/tensorflow_src/Process.inc -O $PROCESS_INC_PATH \
    && wget -q https://github.com/PythonMove/alpine-python3-tensorflow/raw/main/tensorflow_src/config.h.cmake -O $LLVM_CONFIG_PATH \
    && bazel build \
        --jobs 6 \
        --local_cpu_resources 6 \
        --local_ram_resources 8192 \
        --config=v2 \
        --config=noaws \
        --config=nogcp \
        --config=nonccl \
        --config=mkl \
        --cxxopt="-DSTACK_TRACE:BOOL=OFF" \
        //tensorflow/tools/pip_package:build_pip_package \
    && ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg \
    && cd \
    && pip3 install --no-cache-dir /tmp/tensorflow_pkg/tensorflow-${TENSORFLOW_VERSION}-cp38-cp38m-linux_x86_64.whl \
    && apk del .build-deps \
    && rm -f /usr/bin/bazel \
    && rm -rf /tmp/* /root/.cache

# --copt="-DSTACK_TRACE:BOOL=OFF"
