FROM python:3.8.12-alpine3.14

RUN apk add --update --no-cache libstdc++ \
    && pip install --no-cache-dir --extra-index-url https://alpine-wheels.github.io/index \
        numpy==1.19.2 \
        opencv-python==4.4.0.44
