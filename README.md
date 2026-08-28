# FFNet Realtime Semantic Segmentation on Qualcomm® Hexagon™

Experience ultra-fast semantic segmentation on Qualcomm platforms using FFNet. This project leverages **ONNX Runtime (QNN)** to achieve state-of-the-art performance for urban scene understanding.

This document describes how to validate the Qualcomm NPU-enabled ONNX Runtime container on the Qualcomm Hexagon platform and perform real-time semantic segmentation using FFNet.

- **Real-time Performance**: Optimized for QNN HTP backend.
- **Visual Legend**: Automatically identifies 19 Cityscapes categories.
- **Boundary Enhancement**: High-contrast contour detection for better segmentation visibility.
- **Seamless Integration**: Simple CLI for batch video processing.


## 1. Hardware Specifications
> [!NOTE]
> This container image is compatible with Advantech AOM-2721, Advantech AIR-055 and Advantech AFE-A503.

| Component       | Specification      |
|-----------------|--------------------|
| Target Hardware | [Advantech AOM-2721](https://www.advantech.com/en-us/products/risc_evaluation_kit/aom-dk2721/mod_0e561ece-295c-4039-a545-68f8ded469a8) |
| SoC             | Qualcomm® QCS6490   |
| GPU             | Qualcomm® Adreno™ 643        |
| DSP             | Qualcomm® Hexagon™ 770 (12 TOPs) |
| Memory          | 8GB LPDDR5         |

| Component       | Specification      |
|-----------------|--------------------|
| Target Hardware | [Advantech AIR-055](https://www.advantech.com/en-us/products/932c8818-07cc-4917-89e9-7a678ddc029c/air-055/mod_4e23ea2a-d196-4884-8c62-c31780fbafb0) |
| SoC             | Qualcomm® Dragonwing™ IQ-9075   |
| GPU             | Qualcomm® Adreno™ 663        |
| DSP             | Qualcomm® Hexagon™ (100 TOPs) |
| Memory          | 32GB LPDDR5         |

| Component       | Specification      |
|-----------------|--------------------|
| Target Hardware | [Advantech AFE-A503](https://www.advantech.com/zh-tw/products/8d5aadd0-1ef5-4704-a9a1-504718fb3b41/afe-a503/mod_12fdad30-7018-42b3-8d55-4b463f90166b) |
| SoC             | Qualcomm® Dragonwing™ IQ-9075M   |
| GPU             | Qualcomm® Adreno™ 663        |
| DSP             | Qualcomm® Hexagon™ (100 TOPs) |
| Memory          | 32GB LPDDR5         |

## 2. Software Components

| Base Container        | Version |
| --------------------- | ------- |
| [onnxruntime-on-Qualcomm-Hexagon](https://github.com/Advantech-Containers/onnxruntime-on-Qualcomm-Hexagon)| v1   |

| Component             | Version | Description                                                        |
| --------------------- | ------- | ------------------------------------------------------------------ |
| Ubuntu                | 22.04   | Guest OS                                                           |
| Python                | 3.10    | Runtime environment                                                |
| ONNX Runtime (QNN EP) | 1.24.1  | Custom build with QNN Execution Provider (Built with QAIRT 2.43.0) |
| QAIRT (QNN SDK)       | 2.43.0  | Qualcomm AI Runtime backend library                                |
| LiteRT                | 2.1.4   | Provides QNN TFLite Delegate support for GPU/NPU acceleration      |
| Gstreamer             | 1.20.3  | Multimedia framework for building flexible audio/video pipelines   |

**Note**: The custom build of `onnxruntime-qnn` currently only works within this container environment.

## 3. Run Container

### Develop on device
---
### Option 1 : Deploy Container with WEDA API

1. Select the Docker Compose file that matches your device and platform directly
   - `docker-compose-qcs6490-yocto.yml`: Advantech AOM-2721 (Advantech QCS6490 platforms)
   - `docker-compose-qcs9075-ubuntu.yml`: Advantech AIR-055, Advantech AFE-A503 (Advantech IQ9 platforms)
     
2. [Use WEDA API to deploy container to the edge device](https://learn.advantech.com/weda/docs/Getting_Started/Deploy_Container_to_Device)


#### Option 2: Auto Judge Platform Script
1. Download repo and copy the project files to device

   ![alt text](image.png)
3. Unzip files and setup permission with following commands:
    ```bash
    unzip FFNet-Real-time-Semantic-Segmentation-on-Qualcomm-Hexagon-main.zip
    chmod +x -R FFNet-Real-time-Semantic-Segmentation-on-Qualcomm-Hexagon-main
    cd FFNet-Real-time-Semantic-Segmentation-on-Qualcomm-Hexagon-main
    ```
4. launch the container
    ```bash
    ./run-container
    ```
---
#### Option 3: Docker Command Setup (manual judge platform)
1. Copy the Docker Compose file that matches your device and platform directly
- `docker-compose-qcs6490-yocto.yml`: Advantech AOM-2721
- `docker-compose-qcs9075-ubuntu.yml`: Advantech AIR-055, Advantech AFE-A503

2. Launch the container
    For QCS6490 Yocto devices, such as Advantech AOM-2721:
    ```bash
    docker compose -f docker-compose-qcs6490-yocto.yml up -d
    ```

    For IQ9075 Ubuntu devices, such as Advantech AIR-055 or Advantech AFE-A503:
    ```bash
    docker compose -f docker-compose-qcs9075-ubuntu.yml up -d
    ```


## 4. Usage (FFNet Inference)

You can perform inference on video files using the provided script. The script automatically adds a visual legend and enhances class boundaries for better clarity.

### Basic Inference
```bash
python ffnet-inference.py -i <input_video> -o /workspace/<output_video> --gst-sink <gst-sink>
```

For example, to run inference on a video file and save the segmentation result:

- QCS6490
```bash
python ffnet-inference.py -i example.mp4 -o /workspace/output.mp4 --gst-sink glimagesink
```

- IQ9
```bash
python ffnet-inference.py -i example.mp4 -o /workspace/output.mp4 --gst-sink waylandsink
```
You can view the output video in the host project directory `FFNet-Real-time-Semantic-Segmentation-on-Qualcomm-Hexagon-main/workspace`.



### Advanced Options
| Argument | Description | Default |
| :--- | :--- | :--- |
| `-i, --input` | Path to the input video or camera path | (Required) |
| `-o, --output` | Path to the output video | `output.mp4` |
| `-m, --model` | Path to the ONNX model | `models/.../model.onnx` |
| `--target-width` | Model and display processing width. | `1024` |
| `--target-height` | Model and display processing height. | `512` |
| `--cam-width` | Camera capture width for `/dev/video*` input. | `1280` |
| `--cam-height` | Camera capture height for `/dev/video*` input. | `720` |
| `--cam-fps` | Camera capture FPS for `/dev/video*` input. | `30` |
| `--cam-format` | Camera FourCC format. Supported values are `MJPG`, `YUYV`, and `YUY2`. | `MJPG` |
| `--no-display` | Disable GStreamer UI display. | Disabled by default |
| `--no-save` | Disable output video writing. | Disabled by default |
| `--gst-sink` | GStreamer video sink used for display. For QCS6490 select `glimagesink`, for IQ9 select `waylandsink` | `glimagesink` |
| `--gst-sync` | Enable GStreamer sink synchronization. By default, sync is disabled to reduce display latency. | Disabled |

---

## 5. Exit container
Inside the container, type:
```bash
exit
```

Expected output:
```text
Exited container. Cleaning up...
[+] Running 2/2
 ✔ Container ffnet-real-time-semantic-segmentation-on-qualcomm-hexagon       Removed                                                                                 10.4s 
 ✔ Network ffnet-real-time-semantic-segmentation-on-qualcomm-hexagon_default  Removed  
```


## 6. Model Performance

Performance was measured on the Qualcomm HTP (Hexagon Tensor Processor). Quantized models (W8A8) provide a significant boost in throughput.

> Device: AOM-2721 (Qualcomm QCS6490)

| Model Name                | Quantization  | Inference/s (FPS) | Acceleration |
| :---                      | :---:         | :---:             | :---: |
| **ffnet_122ns_lowres**    | W8A8          | **71.76**         | NPU |
| ffnet_122ns_lowres        | W8A8          | 9.17              | CPU |


> Device: AIR-055 (Qualcomm IQ9050)

| Model Name                | Quantization  | Inference/s (FPS) | Acceleration |
| :---                      | :---:         | :---:             | :---: |
| **ffnet_122ns_lowres**    | W8A8          | **166.51**         | NPU |
| ffnet_122ns_lowres        | W8A8          | 18.30              | CPU |


> Device: AFE-A503 (Qualcomm IQ9050M)

| Model Name                | Quantization  | Inference/s (FPS) | Acceleration |
| :---                      | :---:         | :---:             | :---: |
| **ffnet_122ns_lowres**    | W8A8          | **150.36**         | NPU |
| ffnet_122ns_lowres        | W8A8          | 22.82              | CPU |

---

## 7. Result

Below is a demonstration of the semantic segmentation results generated by the FFNet model.

![FFNet Inference Result](output.gif)

---

## 8. Project Structure

- `ffnet_inference.py`: Core inference script with legend generation and visual enhancements.
- `performance-comparison.py`: Benchmarking script for throughput analysis.
- `models/`: Contains the ONNX model files (W8A8 quantized and Float32 versions).
- `example.mp4`: Sample input video for testing.

---

