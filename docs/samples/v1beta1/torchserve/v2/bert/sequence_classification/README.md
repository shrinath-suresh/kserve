# TorchServe example with Huggingface bert model

In this example we will show how to serve [Huggingface Transformers with TorchServe](https://github.com/pytorch/serve/tree/master/examples/Huggingface_Transformers)
on KServe.

## Model archive file creation

Clone [pytorch/serve](https://github.com/pytorch/serve) repository,
navigate to `examples/Huggingface_Transformers` and follow the steps for creating the MAR file including serialized model and other dependent files.
TorchServe supports both eager model and torchscript and here we save as the pretrained model. 
  
```bash
torch-model-archiver --model-name BERTSeqClassification --version 1.0 \
--serialized-file Transformer_model/pytorch_model.bin \
--handler ./Transformer_handler_generalized.py \
--extra-files "Transformer_model/config.json,./setup_config.json,./Seq_classification_artifacts/index_to_name.json"
```

Note: In case of bytes input, the [custom handler](https://github.com/pytorch/serve/blob/master/examples/Huggingface_Transformers/Transformer_handler_generalized.py) can be used without any change.
if the bert input is of type tensor, use the [custom handler](tensor/Transformer_handler_generalized.py) placed inside tensor directory.

The command will create `BERTSeqClassification.mar` file in current directory

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
model_snapshot={"name":"startup.cfg","modelCount":1,"models":{"bert":{"1.0":{"defaultVersion":true,"marName":"BERTSeqClassification.mar","minWorkers":1,"maxWorkers":5,"batchSize":1,"maxBatchDelay":5000,"responseTimeout":120}}}}
```

## Preparing input

Sample bytes input for this example is provided in [bert_v2.json](bytes/bert_v2.json)
Update the input text under inputs --> data tag


## Deploying the model in local machine

Start TorchServe

```
torchserve --start --ts-config /mnt/models/config/config.properties --ncs
```

To test locally, clone TorchServe and move to the following folder `kubernetes/kserve/kserve_wrapper`

Start Kserve

```
python __main__.py
```

## Sample request and response for bytes input

Run the following command in local machine

```
curl -v -H "ContentType: application/json" http://localhost:8080/v2/models/bert/infer -d @./bytes/bert_v2.json
```

and the sample response is as below

```
{"id": "d3b15cad-50a2-4eaf-80ce-8b0a428bd298", "model_name": "BERTSeqClassification", "model_version": "1.0", "outputs": [{"name": "predict", "shape": [], "datatype": "BYTES", "data": ["Not Accepted"]}]}
```


## Sample request and response for tensor input

Run the following command in local machine

```
curl -v -H "ContentType: application/json" http://localhost:8080/v2/models/bert/infer -d @./tensor/bert_v2.json
```

and the sample response is as below

```
{"id": "d3b15cad-50a2-4eaf-80ce-8b0a428bd298", "model_name": "BERTSeqClassification", "model_version": "1.0", "outputs": [{"name": "predict", "shape": [1], "datatype": "INT64", "data": [0]}]}
```


