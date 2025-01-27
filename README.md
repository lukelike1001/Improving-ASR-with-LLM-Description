# Improving-ASR-with-LLM-Description

# Fork Details

The forked version of the [Improving-ASR-with-LLM-Description](https://github.com/nickjw0205/Improving-ASR-with-LLM-Description) features air combat transcription tests, which can be accessed with the `--dataset aerial` tag. The `.wav` and `.json` files can be found under the `aerial` folder, with the predicted transcripted found under the `results` folder.

Furthermore, testing was predominantly done on the `--basic` tag, as LLM-generated descriptions were not used for the air combat tests. An example script can be shown below.

```
## Use Collected Description
python whisper_fine.py --dataset aerial --batch 32 --freeze --basic
```

The main changes to `whisper_fine.py` are as shown below. Lines 57-60 implement the new `aerial` dataset:

```
    elif args.dataset == 'aerial':
        data_train = PromptWhisperDataset(base_path=os.path.join(data_root,"aerial/"), phase='train', feature_extractor=feature_extractor, audio_type=".wav", tokenizer=tokenizer, prompt=args.prompt, basic=args.basic, random=args.random)
        data_eval = PromptWhisperDataset(base_path=os.path.join(data_root,"aerial/"), phase='dev', feature_extractor=feature_extractor, audio_type=".wav", tokenizer=tokenizer, prompt=args.prompt, basic=args.basic)
        data_test = PromptWhisperDataset(base_path=os.path.join(data_root,"aerial/"), phase='test', feature_extractor=feature_extractor, audio_type=".wav", tokenizer=tokenizer, prompt=args.prompt, basic=args.basic)
```

Lines 99-101 ensure that the `eval_step` and `log_step` are not set to 0.

```
   # Hard coded for evaluation
    eval_step = max(1, int((len(data_train) // 2) // args.batch))
    log_step = max(1, int((len(data_train) // 50) // args.batch))
```

Lines 154-157 are **commented out** to ensure that only `evaluation` tests are run, since we are evaluating model performance without fine-tuning or training a model beforehand.

```
    # if not args.eval:
    #     print("Start Training!")
    #     # trainer.train(resume_from_checkpoint = True) # if needed
    #     trainer.train()
```

--------------------------------------
[[Paper]](https://www.isca-archive.org/interspeech_2024/suh24_interspeech.html)
Accepted to INTERSPEECH 2024
<!-- 
## Table of Contents
1. [Abstract](#abstract)
2. [Overview of our method](#overview-of-our-method)
3. [Setup](#setup)
    - [Conda Environment](#conda-environment)
    - [Requirements](#requirements)
4. [Run](#run)
5. [Results](#results)
    - [Base.en](#baseen)
    - [Medium.en](#mediumen) -->

## Abstract
End-to-end automatic speech recognition (E2E ASR) systems have significantly improved speech recognition through training on extensive datasets. Despite these advancements, they still struggle to accurately recognize domain specific words, such as proper nouns and technical terminologies. To address this problem, we propose a method to utilize the state-of-the-art Whisper without modifying its architecture, preserving its generalization performance while enabling it to leverage descriptions effectively. Moreover, we propose two additional training techniques to improve the domain specific ASR: decoder fine-tuning, and context perturbation. We also propose a method to use a Large Language Model (LLM) to generate descriptions with simple metadata, when descriptions are unavailable. Our experiments demonstrate that proposed methods notably enhance domain-specific ASR accuracy on real-life datasets, with LLM-generated descriptions outperforming human-crafted ones in effectiveness.

-------------

### Overview of our method.
![Model Structure](./images/model_architecture.jpg)

-------------

### Dataset
Earnings Call Dataset : [[link]](https://drive.google.com/file/d/13R44k-u5yoJ06dlg4LJsQM3bKWXBGvZG/view?usp=sharing)
(original dataset: [[link]](https://github.com/GeminiLn/EarningsCall_Dataset/blob/master/README.md))

OCW Dataset : [[link]](https://drive.google.com/file/d/17rZoXrldUkqI1GdeYtkIUr8hreAijl2f/view?usp=sharing)

-------------
### Setup
##### Conda Environment
```
conda create -n llm-description python=3.9 -y
conda activate llm-description
```
##### Requirements
```
sudo apt update && sudo apt install ffmpeg
pip install -r requirements.txt
```
-------------
### Run
Set dataset path(data_root) and save path(root_path) in whisper_fine.py
```
...
>>> data_root = "/data/jwsuh/whisper-datasets/main"
...
>>> root_path = "results/"
...
```
#### Script
```
# OCW
## Use LLM Generated Description
CUDA_VISIBLE_DEVICES=0 python whisper_fine.py  --dataset ocw --batch 32 --freeze

## Use Collected Description
CUDA_VISIBLE_DEVICES=0 python whisper_fine.py  --dataset ocw --batch 32 --freeze --basic

# Earnings Call
## Use LLM Generated Description
CUDA_VISIBLE_DEVICES=0 python whisper_fine.py  --dataset earning --batch 32 --freeze

## Use Collected Description
CUDA_VISIBLE_DEVICES=0 python whisper_fine.py  --dataset earning --batch 32 --freeze --basic
```
-------------
### Results
##### Base.en
| Models                     | Earnings Call (20 h) | Earnings Call (40 h) | OCW (20 h) | OCW (40 h) |
|----------------------------|----------------------|----------------------|------------|------------|
| Whisper (Frozen)           | 16.39%               | 16.39%               | 11.98%     | 11.98%     |
| + Full Fine-tuning         | 17.38%               | 16.64%               | 10.41%     | 9.94%      |
| + Description              | 20.63%               | 17.70%               | 9.81%      | 9.72%      |
| + Decoder Fine-tuning      | 16.61%               | 15.70%               | **9.79%**  | **9.67%**  |
| + Context Perturbation     | **16.24%**           | **15.15%**           | **9.79%**  | 9.68%      |

##### Medium.en
| Models                     | Earnings Call (20 h) | Earnings Call (40 h) | OCW (20 h) | OCW (40 h) |
|----------------------------|----------------------|----------------------|------------|------------|
| Whisper (Frozen)           | 13.39%               | 13.39%               | 8.71%      | 8.71%      |
| + Full Fine-tuning         | 10.53%               | 10.15%               | 7.94%      | 7.69%      |
| + Description              | 10.47%               | 10.05%               | 8.46%      | 7.66%      |
| + Decoder Fine-tuning      | 10.29%               | 9.87%                | 7.89%      | 7.36%      |
| + Context Perturbation     | **10.18%**           | **9.71%**            | **7.68%**  | **7.33%**  |
