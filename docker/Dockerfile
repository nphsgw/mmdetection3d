FROM pytorch/pytorch:1.11.0-cuda11.3-cudnn8-runtime
# ベースイメージによってはdocker build時にGPGエラーが発生して途中で止まる。
# その場合、<https://zenn.dev/takakurasato/scraps/d3e0ee6132a5c5>を参考にして解決する。
RUN apt-get update && apt-get install -y \
    ffmpeg libsm6 libxext6 git ninja-build libglib2.0-0 libxrender-dev \
    libgl1 libgomp1 build-essential

RUN pip install open3d
RUN pip install openmim
RUN mim install mmcv-full mmdet mmsegmentation
# docker buildしたときのカレントディレクトリをコンテナー/mmdetection3dにコピーする
COPY . /mmdetection3d
WORKDIR /mmdetection3d
RUN pip install -e .