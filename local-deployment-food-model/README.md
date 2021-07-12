## Demo local deploy
The `train.ipynb` shows fine tuning a classification model which is deployed locally here.

## Local deployment with docker
For locall running on cpu see -> https://github.com/pytorch/serve/tree/master/docker#start-cpu-container

It IS necessary to build the container:
```
docker build -t torchserve:latest deployment/
```

Then run:

```
docker run -p8080:8080 -p8081:8081 -p8082:8082 \
           torchserve:latest \
           torchserve --model-store /home/model-server/model-store/ --models foodnet=foodnet_resnet18.mar
```

Test: `curl http://localhost:8080/ping` then `curl -X POST http://localhost:8080/predictions/foodnet -T sample.jpg`

Tested successfully on Apple M1 on 18/5/2021.

## Native local running of torchserve
Torchserve is written in Java, so unless you plan on using docker you will need Java JDK 11 installed for the tutorial. Install from https://www.oracle.com/uk/java/technologies/javase-jdk11-downloads.html

Install python requirements in a venv as usual. The `torch-model-archiver` is used to convert the `.pth` state dictionary to the `.mar` file required by torchserve - note the `.pth` and `.mar` files are identically sized at around 40 MB. 

NOTE: On Mac M1 native running fails with `Load model failed: foodnet, error: Worker died.`