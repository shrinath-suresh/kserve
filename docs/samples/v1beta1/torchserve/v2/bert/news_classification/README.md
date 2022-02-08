## Training the model

This example finetunes HuggingFace Bert model to classify AG news dataset. The example is adapted from [here](https://github.com/mlflow/mlflow-torchserve/tree/master/examples/E2EBert)

Copy the example files from [here](https://github.com/mlflow/mlflow-torchserve/tree/master/examples/E2EBert)

To train the model run the following command

```
python news_classifier.py --max_epochs <num-epochs> --num_samples <num-samples>
```

For example:

```
python news_classifier.py --max_epochs 5 --num_samples 50000
```

Once the training is completed `state_dict.pth` file will be created.

## Packaging the model

Choose the handler from the bytes/tensor based on the input data type (bytes or tensor)

Run the following command to create mar file

```
torch-model-archiver --model-name bert --serialized-file state_dict.pth --model-file news_classifier.py --handler news_classifier_handler.py -v 1 --extra-files "class_mapping.json,bert_base_uncased_vocab.txt,wrapper.py"
```

The command will create `bert.mar` file in current directory

Move the mar file to model-store folder 

and use the following config properties

```
inference_address=http://0.0.0.0:8085
management_address=http://0.0.0.0:8085
metrics_address=http://0.0.0.0:8082
enable_envvars_config=true
install_py_dep_per_model=true
enable_metrics_api=true
service_envelope=kservev2
metrics_format=prometheus
NUM_WORKERS=1
number_of_netty_threads=4
job_queue_size=10
model_store=/mnt/models/model-store
model_snapshot={"name":"startup.cfg","modelCount":1,"models":{"bert":{"1.0":{"defaultVersion":true,"marName":"bert.mar","minWorkers":1,"maxWorkers":5,"batchSize":1,"maxBatchDelay":5000,"responseTimeout":120}}}}
```

## Preparing input

Sample bytes input for this example is provided in [bert_v2.json](bytes/bert_v2.json)
Update the input text under inputs --> data tag

Sample tensor input for this example is provided in [bert_v2.json](tensor/bert_v2.json)

## Deploying the model

Start TorchServe

```
torchserve --start --ts-config /mnt/models/config/config.properties --ncs
```

To test locally, clone TorchServe and move to the following folder `kubernetes/kserve/kserve_wrapper`

Start Kserve

```
python __main__.py
```

## Sample request and response

Run the following command in local machine

```
curl -v -H "ContentType: application/json" http://localhost:8080/v2/models/bert/infer -d @./bert_v2.json
```

and the sample response is as below

```
{"id": "d3b15cad-50a2-4eaf-80ce-8b0a428bd298", "model_name": "bert", "model_version": "1", "outputs": [{"name": "predict", "shape": [], "datatype": "BYTES", "data": ["\"Sci/Tech\""]}]}
```


