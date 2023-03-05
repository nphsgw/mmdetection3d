FROM pytorch/pytorch:1.11.0-cuda11.3-cudnn8-runtime
# ベースイメージによってはdocker build時にGPGエラーが発生して途中で止まる。
# その場合、<https://zenn.dev/takakurasato/scraps/d3e0ee6132a5c5>を参考にして解決する。

RUN apt-get update && apt-get install -y \
    ffmpeg libsm6 libxext6 git ninja-build libglib2.0-0 libxrender-dev \
    libgl1 libgomp1 build-essential wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN pip install open3d
RUN pip install openmim
RUN mim install mmcv-full mmdet mmsegmentation
WORKDIR /workspace
RUN git clone https://github.com/open-mmlab/mmdetection3d.git
WORKDIR /workspace/mmdetection3d
RUN mkdir checkpoints
RUN wget -P ./checkpoints/ https://download.openmmlab.com/mmdetection3d/v1.0.0_models/centerpoint/centerpoint_01voxel_second_secfpn_circlenms_4x8_cyclic_20e_nus/centerpoint_01voxel_second_secfpn_circlenms_4x8_cyclic_20e_nus_20220810_030004-9061688e.pth

# -eオプションはインストールしたパッケージを編集可能な状態にするという意味。
# コードを変更しても再度インストールしなくても反映される。
RUN pip install -e .
# docker build -t mmdet3d -f docker/Dockerfile .
# rockerを使う場合
# rocker --nvidia --x11 --privileged --home mmdet3d
# docker runで起動する場合
# xhost local:root
# docker run -it --rm --gpus 'all,"capabilities=compute,utility,graphics"' -v /tmp/.X11-unix:/tmp/.X11-unix -v/home:/home -e DISPLAY mmdet3d bash
# コンテナー内ででもプログラムを起動する
# python demo/pcd_demo.py demo/data/kitti/kitti_000008.bin configs/second/hv_second_secfpn_6x8_80e_kitti-3d-car.py checkpoints/hv_second_secfpn_6x8_80e_kitti-3d-car_20200620_230238-393f000c.pth --show
# cd data
# mkdir nuscenes
# cd nuscenes
# ln -s /home/yuki/clone/mmdetection3d/data/nuscenes/maps/ maps && ln -s /home/yuki/clone/mmdetection3d/data/nuscenes/samples/ samples && ln -s /home/yuki/clone/mmdetection3d/data/nuscenes/sweeps/ sweeps && ln -s /home/yuki/clone/mmdetection3d/data/nuscenes/v1.0-mini/ v1.0-mini
# cd ../..
# python tools/create_data.py nuscenes --root-path ./data/nuscenes --version v1.0-mini --out-dir ./data/nuscenes --extra-tag nuscenes
# wget -P ./checkpoints/ https://download.openmmlab.com/mmdetection3d/v1.0.0_models/centerpoint/centerpoint_01voxel_second_secfpn_circlenms_4x8_cyclic_20e_nus/centerpoint_01voxel_second_secfpn_circlenms_4x8_cyclic_20e_nus_20220810_030004-9061688e.pth

# トレーニング済みモデルの予測結果を可視化して指定フォルダに保存する。
# @note このコマンドだとエラーが発生。mmdet3d/api/test.pyの52、53行目のshow=showと,score_thr=show_score_thrを削除すると実行できる。
#TypeError: show_results() got an unexpected keyword argument 'show' 
#TypeError: show_results() got an unexpected keyword argument 'score_thr'
# python tools/test.py configs/centerpoint/centerpoint_01voxel_second_secfpn_circlenms_4x8_cyclic_20e_nus.py checkpoints/centerpoint_01voxel_second_secfpn_circlenms_4x8_cyclic_20e_nus_20220810_030004-9061688e.pth --show --show-dir ./show_results

# 上記のコマンドを使うぐらいなら以下をコマンドが修正がない分便利。
# 評価処理中に同時に可視化する場合。out_dirを指定することで同時に保存できる。
# "show=True"を削除すれば可視化はなくなる。
# python tools/test.py configs/centerpoint/centerpoint_01voxel_second_secfpn_circlenms_4x8_cyclic_20e_nus.py checkpoints/centerpoint_01voxel_second_secfpn_circlenms_4x8_cyclic_20e_nus_20220810_030004-9061688e.pth --out ./data/nuscenes/show_results/result.pkl --eval mAP --eval-options "show=True" "out_dir=./data/nuscenes/show_results"

# バックエンドで可視化するコマンド
# pklファイルをもとにデータセットに対して予測処理を行い、それをファイルに保存する。
# python tools/misc/visualize_results.py configs/centerpoint/centerpoint_01voxel_second_secfpn_circlenms_4x8_cyclic_20e_nus.py --result ./data/nuscenes/show_results/result.pkl --show-dir ./data/nuscenes/show_results

# データセットの可視化
# output-dirを指定することでフォルダに保存される。
#python tools/misc/browse_dataset.py configs/_base_/datasets/nus-3d.py --task det --output-dir ./dataset_shows --online