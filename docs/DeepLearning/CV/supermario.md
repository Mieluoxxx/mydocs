---
title: YOLO实战--supermario的检测
---

## 前端页面的搭建

使用的框架为Streamlit，极简实用主义

```python
#app.py
from pathlib import Path
from PIL import Image
import streamlit as st

import config
from utils import (
    load_model,
    infer_uploaded_image,
    infer_uploaded_video,
    infer_uploaded_webcam,
)

# 设置页面布局
st.set_page_config(
    page_title="YOLOv8交互界面",
    page_icon="🤖",
    layout="wide",
    initial_sidebar_state="expanded",
)

# 主页面标题
st.title("YOLOv8交互界面")

# 侧边栏
st.sidebar.header("深度学习模型配置")

# 模型选项
task_type = st.sidebar.selectbox("选择任务", ["检测"])

model_type = None
if task_type == "检测":
    model_type = st.sidebar.selectbox("选择模型", config.DETECTION_MODEL_LIST)
else:
    st.error("目前只实现了 '检测' 功能")

confidence = float(st.sidebar.slider("选择模型置信度", 30, 100, 50)) / 100

model_path = ""
if model_type:
    model_path = Path(config.DETECTION_MODEL_DIR, str(model_type))
else:
    st.error("请在侧边栏中选择模型")

# 加载预训练的深度学习模型
try:
    model = load_model(model_path)
except Exception as e:
    st.error(f"无法加载模型。请检查指定的路径：{model_path}")

# 图像/视频选项
st.sidebar.header("图像/视频配置")
source_selectbox = st.sidebar.selectbox("选择来源", config.SOURCES_LIST)

source_img = None
if source_selectbox == config.SOURCES_LIST[0]:  # 图像
    infer_uploaded_image(confidence, model)
elif source_selectbox == config.SOURCES_LIST[1]:  # 视频
    infer_uploaded_video(confidence, model)
elif source_selectbox == config.SOURCES_LIST[2]:  # 摄像头
    infer_uploaded_webcam(confidence, model)
else:
    st.error("目前只实现了 '图像' 和 '视频' 来源")

```

```python
#config.py
from pathlib import Path
import sys

# 获取当前文件的绝对路径
file_path = Path(__file__).resolve()

# 获取当前文件的父目录
root_path = file_path.parent

# 如果根路径尚未添加到sys.path列表中，则将其添加
if root_path not in sys.path:
    sys.path.append(str(root_path))

# 获取相对于当前工作目录的根目录的相对路径
ROOT = root_path.relative_to(Path.cwd())


# Source
SOURCES_LIST = ["Image", "Video", "Webcam"]


# DL 模型配置
DETECTION_MODEL_DIR = ROOT / "weights" / "detection"
YOLOv8n = DETECTION_MODEL_DIR / "yolov8n.pt"
YOLOv8s = DETECTION_MODEL_DIR / "yolov8s.pt"
YOLOv8m = DETECTION_MODEL_DIR / "yolov8m.pt"
YOLOv8l = DETECTION_MODEL_DIR / "yolov8l.pt"
YOLOv8x = DETECTION_MODEL_DIR / "yolov8x.pt"
mario = DETECTION_MODEL_DIR / "mario.pt"

DETECTION_MODEL_LIST = [
    "yolov8n.pt",
    "yolov8s.pt",
    "yolov8m.pt",
    "yolov8l.pt",
    "yolov8x.pt",
    "mario.pt",
]
```

```python
#utils.py
from ultralytics import YOLO
import streamlit as st
import cv2
from PIL import Image
import tempfile
import torch


def _display_detected_frames(conf, model, st_frame, image):
    """
    使用YOLOv8模型在视频帧上显示检测到的对象。
    :param conf (float): 对象检测的置信度阈值。
    :param model (YOLOv8): 包含YOLOv8模型的`YOLOv8`类的实例。
    :param st_frame (Streamlit对象): 用于显示检测到的视频的Streamlit对象。
    :param image (numpy数组): 表示视频帧的numpy数组。
    :return: 无
    """
    # 将图像调整为标准大小
    image = cv2.resize(image, (720, int(720 * (9 / 16))))

    # 使用YOLOv8模型在图像中预测对象
    res = model.predict(image, conf=conf)

    # 在视频帧上绘制检测到的对象
    res_plotted = res[0].plot()
    st_frame.image(res_plotted, caption="检测到的视频", channels="BGR", use_column_width=True)


@st.cache_resource
def load_model(model_path):
    """
    从指定的model_path加载YOLO目标检测模型。

    参数:
        model_path (str): YOLO模型文件的路径。

    返回:
        YOLO目标检测模型。
    """
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    model = YOLO(model_path).to(device)
    return model


def infer_uploaded_image(conf, model):
    """
    执行上传图像的推断
    :param conf: YOLOv8模型的置信度
    :param model: 包含YOLOv8模型的`YOLOv8`类的实例。
    :return: 无
    """
    source_img = st.sidebar.file_uploader(
        label="选择图像...", type=("jpg", "jpeg", "png", "bmp", "webp")
    )

    col1, col2 = st.columns(2)

    with col1:
        if source_img:
            uploaded_image = Image.open(source_img)
            # 将上传的图像添加到页面并添加标题
            st.image(image=source_img, caption="上传的图像", use_column_width=True)

    if source_img:
        if st.button("执行"):
            with st.spinner("运行中..."):
                res = model.predict(uploaded_image, conf=conf)
                boxes = res[0].boxes
                res_plotted = res[0].plot()[:, :, ::-1]

                with col2:
                    st.image(res_plotted, caption="检测到的图像", use_column_width=True)
                    try:
                        with st.expander("检测结果"):
                            for box in boxes:
                                st.write(box.xywh)
                    except Exception as ex:
                        st.write("尚未上传图像！")
                        st.write(ex)


def infer_uploaded_video(conf, model):
    """
    执行上传视频的推断
    :param conf: YOLOv8模型的置信度
    :param model: 包含YOLOv8模型的`YOLOv8`类的实例。
    :return: 无
    """
    source_video = st.sidebar.file_uploader(label="选择视频...")

    if source_video:
        st.video(source_video)

    if source_video:
        if st.button("执行"):
            with st.spinner("运行中..."):
                try:
                    tfile = tempfile.NamedTemporaryFile()
                    tfile.write(source_video.read())
                    vid_cap = cv2.VideoCapture(tfile.name)
                    st_frame = st.empty()
                    while vid_cap.isOpened():
                        success, image = vid_cap.read()
                        if success:
                            _display_detected_frames(conf, model, st_frame, image)
                        else:
                            vid_cap.release()
                            break
                except Exception as e:
                    st.error(f"加载视频时出错：{e}")


def infer_uploaded_webcam(conf, model):
    """
    执行摄像头推断。
    :param conf: YOLOv8模型的置信度
    :param model: 包含YOLOv8模型的`YOLOv8`类的实例。
    :return: `无
    """
    try:
        flag = st.button(label="停止运行")
        vid_cap = cv2.VideoCapture(0)  # 本地摄像头
        st_frame = st.empty()
        while not flag:
            success, image = vid_cap.read()
            if success:
                _display_detected_frames(conf, model, st_frame, image)
            else:
                vid_cap.release()
                break
    except Exception as e:
        st.error(f"加载视频时出错：{str(e)}")

```




## 后端逻辑（YOLO）

1. 将视频处理成帧

```python
import cv2
import os
def video_to_frames(video_path, output_path, frame_interval=20):
    # 打开视频文件
    cap = cv2.VideoCapture(video_path)

    # 确保视频文件已成功打开
    if not cap.isOpened():
        print("Error: Could not open video.")
        return

    # 获取视频的帧率和帧数
    fps = cap.get(cv2.CAP_PROP_FPS)
    frame_count = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))

    print(f"Video FPS: {fps}")
    print(f"Total frames: {frame_count}")

    # 初始化计数器
    frame_number = 0

    # 逐帧读取视频并保存为图像文件
    while True:
        ret, frame = cap.read()
        if not ret:
            print(f"Error reading frame {frame_number}")
            break

        # 每隔 frame_interval 帧保存一次图像
        if frame_number % frame_interval == 0:
            # 生成图像文件名
            frame_filename = f"{output_path}/frame_{frame_number:04d}.jpg"

            # 保存图像文件
            cv2.imwrite(frame_filename, frame)

        frame_number += 1

    # 释放视频对象
    cap.release()

if __name__ == "__main__":
    # 输入视频文件路径和输出帧图像的目录
    input_video_path = os.getcwd() + "/data/raw/supermario.mp4"
    output_frames_path = os.getcwd() + "/data/processed/"

    # 调用函数进行转换
    video_to_frames(input_video_path, output_frames_path)

```

2. 利用colab进行训练

```python
%%
%pip install ultralytics
%%

%%
from ultralytics import YOLO
from IPython.display import display, Image
%%

%%
# 加载数据集
%pip install roboflow

from roboflow import Roboflow
rf = Roboflow(api_key="AKI-KEY")
project = rf.workspace("morgan-woods-aafya").project("supermario")
dataset = project.version(1).download("yolov8")
%%

%%
# 训练
%yolo task=detect mode=train model=yolov8m.pt data={dataset.location}/data.yaml epochs=20 imgsz=640
%%

%%
# 混淆矩阵
Image(filename=f'/content/runs/detect/train/confusion_matrix.png',width=600)
%%

%%
# 训练验证结果
Image(filename=f'/content/runs/detect/train/results.png',width=600)
%%

%%
# 验证
!yolo task=detect mode=val model=/content/runs/detect/train/weights/best.pt data={dataset.location}/data.yaml
%%

%%
# 测试
!yolo task=detect mode=predict model=/content/runs/detect/train/weights/best.pt conf=0.5 source="/content/SuperMario-1/supermario.mp4"
%%
```

