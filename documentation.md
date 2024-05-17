This repo is for learning purpose on how to train model using ROSA commonly asked FAQ.

# The InstructLab Framework

[InstructLab](https://github.com/instructlab/instructlab) is an open source framework developed by IBM and Red Hat that is meant to provide a standard, simple and collaborative way to align Large Language Models. The terms Lab means Large Language Models Alignment For Chat-bots.
It provides as single tool to download, serve, test and train LLMs so that anyone can contribute and improve the existing capabilities — whether it’s internally, or externally for the broader community.

In this example we will follow the following steps

1. Download the Granite-7B model
2. Chat with the LLM model
3. Add new knowledge or skill to taxonomy
4. Generate new synthetic training data
5. Retrain the model

## Pre-requiste
Fo thsi I am using local machine (Macbook Pro with 32GB RAM) but you can spin up g5.4xlarge machine on AWS

Instructlab requires ~18GB of GPU memory, which most of the times requires a data-center server — but alternatively it could be a Jupyter Notebook, Google Collab, Kaggle — or any other type of IDE that you’d like to use to fine-tune the model. We are using a very small sample

* [InstructLab](https://github.com/instructlab/instructlab) — A repository that helps you set up the ilab CLI, which is how you interact easily with GGUF LLMs, make sure to follow the instructions to install it
* [Taxonomy](https://github.com/instructlab/taxonomy) — A repository that contains the framework for enriching the base LLM easily, using knowledge/skills that are being added to the LLM based on YAML files that are easy to use

## Tutorial

1. Clone the taxonomy repository:

```
git clone --recurse-submodules https://github.com/instructlab/instructlab.git
```

2. Now init the ilab CLI to and use it to clone the taxonomy repository:
```
source venv/bin/activate
```
```
ilab init
```
##### Output
```
Welcome to InstructLab CLI. This guide will help you set up your environment.
Please provide the following values to initiate the environment [press Enter for defaults]:
Path to taxonomy repo [taxonomy]: <ENTER>
```

3. Downloading and Serving The LLM. we’ll be using the Granite-7B LLM

```
ilab download --repository instructlab/granite-7b-lab-GGUF --filename granite-7b-lab-Q4_K_M.gguf
```

4. The LLM is basically being downloaded from HuggingFace into your local directory, it should be found in the models directory under your current directory context. Let’s serve it locally so that we’ll be able to interact with it.

```
 ilab serve --model-path models/granite-7b-lab-Q4_K_M.gguf --num-threads 14
```

When you serve the model, there’s a Python wrapper that takes your current LLM and serves it using a local endpoint, for example — http://127.0.0.1:8000 by default.

5. Adding Skills and Knowledge to the LLM

Now, let’s assume that we want to teach the model how to answer questions and if the LLM doesn’t have that knowledge. We can use this to add FAQ for **HCP or ROSA clusters*** then we’ll add skills in the form of questions and answers (Q&A) that can be presented in the form of a YAML file.

First, let’s create a directory under the taxonomy repository, that represnts the topic that we want the model to be taught on:

```
mkdir -p taxonomy/compositional_skills/writing/freeform/rosa
```

Add the qna.yaml file that has basic questions and answers for ROSA classic and other is a metadata file called attribution.txt — both will be used in the training phase later on:

Example yaml file (Note each line is limited to 120 characters)

```
task_description: 'Teach the model about deployment of ROSA classic STS cluster'
created_by: Nerav Doshi
seed_examples:
  - question: What is Red Hat OpenShift Service on AWS (ROSA)?
    answer: |
      Red Hat OpenShift Service on AWS (ROSA) is a fully-managed and jointly supported Red Hat OpenShift |
      offering that combines the power of Red Hat OpenShift, the industry’s most comprehensive enterprise |
      Kubernetes platform, and the AWS public cloud.
  - question: Where can I go to get more information/details?
    answer: |
      ROSA_url: https://www.openshift.com/products/amazon-openshift
      ROSA_link: [ROSA Webpage, ROSA_url]
      ROSA_doc_url: https://docs.openshift.com/rosa/welcome/index.html
      ROSA_doc_link: [ROSA Documenation, ROSA_doc_url]
  - question: When will ROSA be GA?
    answer: |
      Red Hat OpenShift Service on AWS is GA as of March 2021.
```

6. Generate a Dataset Based on Q&A

```
ilab generate --model granite-7b-lab-Q4_K_M
INFO 2024–05–10 23:07:05,280 generate_data.py:468 Selected taxonomy path compositional_skills->writing->freeform->rosa
```

##### Output

```
Q> What is the purpose of the `rosaclassic-setup` command in the ROSA Classic deployment?
I> 
A> The `rosaclassic-setup` command is used to configure and initialize the ROSA Classic environment, enabling users to deploy and manage the ROSA Classic platform on their infrastructure. This command simplifies the deployment process and ensures that the ROSA Classic environment is properly configured for optimal performance.
```

Now that we’re done and the dataset is generated, notice that you have a few files created under your current context that are being used is the training and testing phases in generated folder

```
(venv) nerav@nedoshi-mac generated % ls
discarded_granite-7b-lab-Q4_K_M_2024-05-17T14_00_27.log
discarded_granite-7b-lab-Q4_K_M_2024-05-17T15_17_03.log
generated_granite-7b-lab-Q4_K_M_2024-05-17T14_00_27.json
generated_granite-7b-lab-Q4_K_M_2024-05-17T15_17_03.json
test_granite-7b-lab-Q4_K_M_2024-05-17T14_00_27.jsonl
test_granite-7b-lab-Q4_K_M_2024-05-17T15_17_03.jsonl
train_granite-7b-lab-Q4_K_M_2024-05-17T14_00_27.jsonl
train_granite-7b-lab-Q4_K_M_2024-05-17T15_17_03.jsonl
```

7. Train the LLM with generated Dataset.
Now that we’re done and the dataset is generated, notice that you have a few files created under your current context that are being used is the training and testing phases. If you have NVIDIA GPU then you can specify device.

```
ilab train --gguf-model-path models/granite-7b-lab-Q4_K_M.gguf --device=cuda
```

OR on Macbook (takes a long time and it can fail if not enough resources)

```
ilab train --gguf-model-path models/granite-7b-lab-Q4_K_M.gguf
```
##### Output
```
(venv) nerav@nedoshi-mac instructlab % ilab train --gguf-model-path models/granite-7b-lab-Q4_K_M.gguf
Found multiple files from `ilab generate`. Using the most recent generation.
[INFO] Loading
config.json: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 644/644 [00:00<00:00, 3.68MB/s]
added_tokens.json: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 119/119 [00:00<00:00, 117kB/s]
generation_config.json: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 136/136 [00:00<00:00, 1.14MB/s]
model.safetensors.index.json: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 23.9k/23.9k [00:00<00:00, 24.4MB/s]
special_tokens_map.json: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 655/655 [00:00<00:00, 8.38MB/s]
tokenizer_config.json: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2.33k/2.33k [00:00<00:00, 23.3MB/s]
tokenizer.model: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 493k/493k [00:00<00:00, 1.03MB/s]
tokenizer.json: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1.80M/1.80M [00:01<00:00, 984kB/s]
model-00002-of-00003.safetensors:   0%|                                                                                                                                               | 0.00/5.00G [00:00<?, ?B/s]
model-00001-of-00003.safetensors:   1%|█▉                                                                                                                                    | 73.4M/4.94G [00:33<36:31, 2.22MB/s]
model-00003-of-00003.safetensors:   2%|██▏                                                                                                                                   | 73.4M/4.54G [00:33<33:43, 2.21MB/s]
model-00002-of-00003.safetensors:   1%|█▉                                                                                                                                    | 73.4M/5.00G [00:34<38:30, 2.13MB/s]
```
8. Once the training was finished, you should expect to have the new GGUF model under that model path (which in our case is the models directory).

  Note: You can Quantize the Fine-Tuned LLM model.(Optional step). Quantization is a technique used to reduce the size of neural networks, including LLMs, by modifying the precision of their weights. 
  Instead of using full precision (e.g., 32-bit floating point), quantization reduces the weights to lower bit precision (e.g., 8-bit, 4-bit, or even lower).
  The goal is to make the model more memory-efficient and suitable for deployment on consumer hardware. Example [github](https://github.com/ggerganov/llama.cpp/tree/master/examples/quantize) for 
  reference

9. Test The Fine-Tuned LLM

```
ilab serve --model-path models/ggml-model-Q4_K_M.gguf --num-threads 14
```

10. Open a new terminal and use ilab chat

```
ilab chat --model ggml-model-Q4_K_M
```
Now you can ask ROSA specific questions that was included in qna.yaml file to make sure model is indeed accurate.

