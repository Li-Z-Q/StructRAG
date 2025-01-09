# StructRAG
StructRAG: Boosting Knowledge Intensive Reasoning of LLMs via Inference-time Hybrid Information Structurization 

https://arxiv.org/abs/2410.08815

## 0. Environment
```
python 3.8.19
vllm 0.6.3.post1
pip install -r requirement.txt
```

## 1. Data Preparation
```
please follow Loong/README.md
```

## 2. StructRAG Inference
```python
# 1. launch llm api server
model_path = "/mnt/data/lizhuoqun/hf_models/Qwen2-72B-Instruct"
CUDA_VISIBLE_DEVICES=0,1,2,3 && OUTLINES_CACHE_DIR=tmp && nohup python -m vllm.entrypoints.openai.api_server --model ${model_path} --served-model-name Qwen --tensor-parallel-size 4 --port 1225 --disable-custom-all-reduce > vllm.log
# 2. run StructRAG
python main.py --url {url_of_api_server} # output will be in ./eval_results/qwen/loong
# 3. transform model output to Loong results format
python do_merge_each_batch.py # results will be in ./Loong/output/qwen
```

## 3. Results Evaluation
```
cd Loong/src && bash run.sh
```

## 4. Router Training (optional)
Qwen2-72B-Instruct has already achieved good routing performance under the few-shot examples setting. If wish to further improve routing accuracy, we can train the 7B model using the DPO algorithm:
```
bash train_router/train.sh
```

After training, deploy the output model as an API using vllm, and obtain url_of_router. When running StructRAG, use the following command:
```
python main.py --url {url_of_api_server} --router_url {url_of_router}
```