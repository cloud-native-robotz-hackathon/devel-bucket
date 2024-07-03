# Setup Nvidia Triton Inferencing Server on GoPiGo 3
Walkthrough to get example onnx model to work.

Create model repository on robot:

```
mkdir -p model_repository/densenet_onnx/1
```
Download example model to model repo:

```
wget -O model_repository/densenet_onnx/1/model.onnx \
     https://contentmamluswest001.blob.core.windows.net/content/14b2744cf8d6418c87ffddc3f3127242/9502630827244d60a1214f250e3bbca7/08aed7327d694b8dbaee2c97b8d0fcba/densenet121-1.2.onnx
```

Pull and run the Triton Server container image:

```
podman run  --rm -p 8000:8000 -p 8001:8001 -p 8002:8002 -v ${PWD}/model_repository:/models nvcr.io/nvidia/tritonserver:24.06-py3 tritonserver --model-repository=/models
```

This will take a while, it'a around 10GB. The Triton Inference Server versions, releases and tags are listed in the catalog here: https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tritonserver

Replace the tag (24.06 in this case) by the one you want from the catalog.

