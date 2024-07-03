# Setup Nvidia Triton Inferencing Server on GoPiGo 3
## Walkthrough to get example onnx model to work.
### Install and start Triton Server

Create model repository on robot:

```
mkdir -p model_repository/densenet_onnx/1
```
Download example model to model repo:

```
wget -O model_repository/densenet_onnx/1/model.onnx \
     https://contentmamluswest001.blob.core.windows.net/content/14b2744cf8d6418c87ffddc3f3127242/9502630827244d60a1214f250e3bbca7/08aed7327d694b8dbaee2c97b8d0fcba/densenet121-1.2.onnx
```

Triton generates a config.pbtxt file if missing with the model, but this didn't work. Create the two file from 
https://github.com/triton-inference-server/server/blob/main/docs/examples/model_repository/densenet_onnx/
in `model_repository/densenet_onnx/`

Pull and run the Triton Server container image:

```
podman run  --rm -p 8000:8000 -p 8001:8001 -p 8002:8002 -v ${PWD}/model_repository:/models nvcr.io/nvidia/tritonserver:24.06-py3 tritonserver --model-repository=/models
```

This will take a while, it'a around 10GB. The Triton Inference Server versions, releases and tags are listed in the catalog here: https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tritonserver

Replace the tag (24.06 in this case) by the one you want from the catalog.

After the container has been pulled and started properly it should show:

```
I0703 08:38:52.522627 1 grpc_server.cc:2463] "Started GRPCInferenceService at 0.0.0.0:8001"
I0703 08:38:52.523694 1 http_server.cc:4692] "Started HTTPService at 0.0.0.0:8000"
I0703 08:38:52.567659 1 http_server.cc:362] "Started Metrics Service at 0.0.0.0:8002"
```

In addition check a bit higher up in the output the model has been loaded and is ready:

```
+---------------+---------+--------+
| Model         | Version | Status |
+---------------+---------+--------+
| densenet_onnx | 1       | READY  |
+---------------+---------+--------+
```

Verify Triton Is Running Correctly:

From the host system use curl to access the HTTP endpoint that indicates server status, you should get HTTP 200.

```
curl -v localhost:8000/v2/health/ready
...
< HTTP/1.1 200 OK
< Content-Length: 0
< Content-Type: text/plain
```

### Get example client and test
Pull the client image with libraries and examples from NGC.

```
pull nvcr.io/nvidia/tritonserver:<xx.yy>-py3-sdk
```

Run the SDK image in interactive mode:

```
podman run -it --rm --net=host nvcr.io/nvidia/tritonserver:24.06-py3-sdk
```

Now inside the running SDK container run the client:

```
/workspace/install/bin/image_client -m densenet_onnx -c 3 -s INCEPTION /workspace/images/mug.jpg
Request 0, batch size 1
Image '/workspace/images/mug.jpg':
    15.349568 (504) = COFFEE MUG
    13.227468 (968) = CUP
    10.424893 (505) = COFFEEPOT
```

## References
https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/getting_started/quickstart.html
https://github.com/triton-inference-server/tutorials#quick-deploy
https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tritonserver



