FROM infotrend/ctpo-cuda_tensorflow_pytorch_opencv:11.8.0_2.14.1_2.1.1_4.8.0-20231201

RUN mkdir /ferret
WORKDIR /ferret

RUN git clone https://github.com/apple/ml-ferret
WORKDIR /ferret/ml-ferret


RUN pip3 install --upgrade pip  \
    && pip3 install -e . \
    && pip3 install pycocotools \
    && pip3 install protobuf==3.20.0 \
    && rm -rf /root/.cache/pip

RUN pip3 install ninja \
    && pip3 install flash-attn --no-build-isolation

RUN apt update && apt install -y tmux

CMD /bin/bash
