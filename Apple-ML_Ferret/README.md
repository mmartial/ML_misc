# Building and testing Apple's ML Ferret

**Requirements:** an Nvidia GPU (tested on an RTX 3090) and a host running Linux (tested on an Ubuntu 22.04), as well as some familiarity with the `docker` and `tmux`.

- Obtain the source code to check its instructions in case of issues.
``` bash
git clone https://github.com/apple/ml-ferret.git
```

- Using the provided [`Dockerfile`](Dockerfile), build a `ml_ferret` container
``` bash
docker build --tag ml_ferret:test .
```

- Download `vicuna-7b-v1.3` from HuggingFace.co 
``` bash
git lfs install
git clone https://huggingface.co/lmsys/vicuna-7b-v1.3
```

- Obtain Apple's Vicuna to Ferret Delta zip file and `unzip` it
``` bash
wget https://docs-assets.developer.apple.com/ml-research/models/ferret/ferret-7b/ferret-7b-delta.zip
unzip ferret-7b-delta.zip
```

- Run the built container to apply the delta onto the file and obtain the final weights. We will get a `ferret-7b-v1.3` directory owned by `root` (since the command was run within `docker`)
``` bash
docker run --rm -it --gpus all -v `pwd`:/base ml_ferret:test python3 -m ferret.model.apply_delta --base /base/vicuna-7b-v1.3 --target /base/ferret-7b-v1-3 --delta /base/ferret-7b-delta
 ```

- The next step prepares the container to start its controller, its UI and load the model:
 ``` bash
 docker run --rm -it --gpus all -v `pwd`:/base -p 10000:10000 -p 7860:7860 ml_ferret:test
```
- Within the running container, start a `tmux` and in three terminals, we will run the controller, the gradio UI and the model_worker:
    - tmux1:
    ```bash
    python -m ferret.serve.controller --host 0.0.0.0 --port 10000
    ```
    - tmux2:
    ```bash
    python -m ferret.serve.gradio_web_server --controller http://localhost:10000 --model-list-mode reload --add_region_feature
    ```
    - tmux3:
    ```bash
    python -m ferret.serve.model_worker --host 0.0.0.0 --controller http://localhost:10000 --port 40000 --worker http://localhost:40000 --model-path /base/ferret-7b-v1-3 --add_region_feature
    ```

- Make sure to wait until the model is loaded (in `tmux3`, you will see `Uvicorn running on http://0.0.0.0:40000`)
In a web browser, access the IP of the host on port 7860, you will access the gradio UI and should at this point will see the `ferret-7b-v1.3` model listed in the top left of the UI and can now use the examples in the bottom left.

