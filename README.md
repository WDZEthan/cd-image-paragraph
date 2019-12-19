## Training for Diversity in Image Paragraph Captioning

This repository includes a PyTorch implementation of [Exploring Sentence Coherence and Diversity in Image Paragraph Generation](). Our code is based on Luke Melas's implementation of [Training for Diversity in Image Paragraph Captioning(https://www.aclweb.org/anthology/D18-1084/), available [here](https://github.com/lukemelas/image-paragraph-captioning). 

### Requirements
* Python 2.7 (because coco-caption does not support Python 3)
* PyTorch 0.4 (with torchvision)
* cider (already included as a submodule)
* coco-caption (already included as a submodule)

If training from scratch, you also need:
* spacy (to tokenize words)
* h5py (to store features)
* scikit-image (to process images) 


### Train your own network
#### Download and preprocess cations

* Download captions:
  *  Run `download.sh` in `data/captions`
* Preprocess captions for training (part 1): 
  * Download `spacy` English tokenizer with `python -m spacy download en`
  * First, convert the text into tokens: `cd scripts && python prepro_text.py`
  * Next, preprocess the tokens into a vocabulary (and map infrequent words to an `UNK` token) with the following command. Note that image/vocab information is stored in `data/paratalk.json` and caption data is stored in `data/paratalk\_label.h5`
```bash
python scripts/prepro_labels.py --input_json data/captions/para_karpathy_format.json --output_json data/paratalk.json --output_h5 data/paratalk
```
* Preprocess captions into a coco-captions format for calculating CIDER/BLEU/etc: 
  *  Run `scripts/prepro\_captions.py`
  *  There should be 14,575/2487/2489 images and annotations in the train/val/test splits
  *  Uncomment line 44 (`(Spice(), "SPICE")`) in `coco-caption/pycocoevalcap/eval.py` to disable Spice testing
* Preprocess ngrams for self-critical training:
```bash
python scripts/prepro_ngrams.py --input_json data/captions/para_karpathy_format.json --dict_json data/paratalk.json --output_pkl data/para_train --split train
```
* Extract image features using an object detector
  * We make pre-processed features widely available:
    * Download and extract `parabu_fc` and `parabu_att` from [here](https://drive.google.com/drive/folders/1-NRSGJw8JYdEJBJuCLlqbemYiUlbi5Xn) into `data/` 
  * Or generate the features yourself:
    * Download the [Visual Genome Dataset](https://visualgenome.org/api/v0/api_home.html)
    * Apply the bottom-up attention object detector [here](https://github.com/peteanderson80/bottom-up-attention) made by Peter Anderson.
    * Use `scripts/make_bu_data.py` to convert the image features to `.npz` files for faster data loading

#### Train the network

Training hyperparameters may be accessed with `python train.py --help`. 

A reasonable set of hyperparameters is provided in `train_xe.sh` (for cross-entropy) and `train_sc.sh` (for self-critical). 
```bash 
mkdir log_xe
./train_xe.sh 
```

You can then copy the model:
```bash
./scripts/copy_model.sh xe sc
```

And train with self-critical:
```bash
mkdir log_sc
./train_xe.sh
```

<!--### Pretrained Network
You can download a pretrained captioning model [here](https://www.dropbox.com/s/ls9hupvzbg93c8r/topdown_sc_alpha_2.0.zip). -->

### Citation
In case you would like to cite our paper/code (no obligation at all): 
```
@article{melaskyriazi2018paragraph, 
  title={Training for diversity in image paragraph captioning},
  author={Melas-Kyriazi, Luke and Rush, Alexander and Han, George},
  journal={EMNLP},
  year={2018}
}     
```
