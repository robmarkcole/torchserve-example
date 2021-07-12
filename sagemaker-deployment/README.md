## Sagemaker deploy torchserve
Torchserve can be hosted as a sagemaker endpoint, and for our super-res model we require a custom endpoint. The tutorial at https://github.com/aws-samples/amazon-sagemaker-endpoint-deployment-of-fastai-model-with-torchserve works through exactly this scenario.

Train: clone the above repo onto a sagemaker notebook instance with GPU, I selected `ml.p2.xlarge` instance in Ireland. Note: if you stop and restart the sagemaker instance your files/work will be lost.

Key points:
- Fastai model files (`.pkl`) must be converted to pytorch `.pth` format, then the `.mar` file is generated
- Dont use one-line python lambda functions to pass to fastai as these will introduce issue on serialization when we want to export a FastAI model
- calling `learn.export` places the file at `learn.path`, not the local directory, also sagemaker UI does not allow browsing to the `.fastai` directory where the files are placed!
- the file `fastai_unet.pkl` is 1.3 Gb, as is the converted `.pth` file. The `.mar` file is slightly smaller at 1.2 Gb, as is the `.tar.gz` file put to S3

Was able to get a response performing inference on torchserve on the sagemaker instance

## Sagemaker
- Activate an conda env from the terminal: `source activate pytorch_latest_p36`
- I have better luck installing with pip than conda
- Update fastai: `pip install -Uqq fastai`
- Alternative ports can be accessed at https://gpu-notebook-qtqu.notebook.eu-west-1.sagemaker.aws/proxy/8082
- by default, notebook instances have 5GB of storage, no matter what type they are. I hit this limit quickly and had to delete some large files
- Models are deployed as containers hosted on ECR
- You can skip S3 permissions if the bucket has "sagemaker" in its name

At model deployment hit https://stackoverflow.com/questions/51044657/validationerror-when-creating-a-sagemaker-model but got around this by creating bucket `sagemaker-models-scratchpad` in Ireland. Then hit https://github.com/shashankprasanna/torchserve-examples/issues/3

## Torchserve tips
- Check health: `curl http://localhost:8080/ping`
- List models: `curl http://localhost:8081/models`
- Stop server: `torchserve --start`
- Example request: `curl http://localhost:8080/predictions/fastunet -T street_view_of_a_small_neighborhood.png`

## Local deployment with docker
It IS necessary to build the container (due to ARM mac?):
```
docker build -t torchserve:latest deployment/
```

```
docker run -p8080:8080 -p8081:8081 -p8082:8082 \
           torchserve:latest \
           torchserve --model-store /home/model-server/model-store/ --models fastunet=fastunet.mar
```

Test: `curl http://localhost:8081/models` then `curl -X POST http://localhost:8080/predictions/fastunet -T street_view_of_a_small_neighborhood.png`

Tested failed on Apple M1 with `Load model failed: fastunet, error: Worker died.`. See https://github.com/pytorch/serve/issues/1010: `The underlying issue is the docker can't handle any custom handler`?