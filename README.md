# Text-To-Video-Finetuning
## Finetune ModelScope's Text To Video model using Diffusers 🧨 
***(This is a WIP)***

## Getting Started
### Requirements

```bash
git clone https://github.com/ExponentialML/Text-To-Video-Finetuning.git
```

```bash
pip install torch torchvision torchaudio
pip install git+https://github.com/huggingface/diffusers.git
pip install transformers
pip install einops
pip install decord
pip install tqdm
```

### Models
The models were downloaded from here https://huggingface.co/damo-vilab/text-to-video-ms-1.7b/tree/main.

This repository was only tested with **FP16** safetensors. Other files (bin, FP32) should work fine, but if you have any trouble, refer to this.

## Hardware
Minimum RTX 3090. You're free to open a PR for optimization (please do!), but this is heavy without gradient checkpointing support.

## Usage
```python
python train.py --config ./configs/my_config.yaml
```

## Preprocessing your data
All videos were preprocessed using the script [here](https://github.com/ExponentialML/Video-BLIP2-Preprocessor) using automatic BLIP2 captions. Please follow the instructions there.

## Configuration
The configuration uses a YAML config borrowed from [Tune-A-Video](https://github.com/showlab/Tune-A-Video) reposotories. Here's the gist of how it works.

<details>
  
```yaml

# The path to your diffusers folder. The structure should look exactly like the huggingface one with folders and json configs
pretrained_model_path: "diffusers_path"

# The directory where your training runs (and samples) will be saved.
output_dir: "./outputs"

# Enable training the text encoder or not.
train_text_encoder: False

# The basis of where your training data is store.
train_data:
  
  # The path to your JSON file using the steps above.
  json_path: "json/train.json"
  
  # Leave this as true for now. Custom configurations are currently not supported.
  preprocessed: True
  
  # Number of frames to sample from the videos. The higher this number, the more VRAM is required (usage is similar to batchsize)
  n_sample_frames: 4
  
  # Choose whether or not to ignore the frame data from the preprocessing step, and shuffle them.
  shuffle_frames: False
  
  # The height and width of training data.
  width: 256      
  height: 256
  
  # At what frame to start the video sampling. Ignores preprocessing frames.
  sample_start_idx: 0
  
  # The rate of sampling frames. This effectively "skips" frames making it appear faster or slower.
  sample_frame_rate: 1
  
  # The key of the video data name. This is to align with any preprocess script changes.
  vid_data_key: "video_path"

# Only sample_preview is used. This is used for validation prompts, but is read by the training json by default.
validation_data:
  sample_preview: True
  video_length: 16
  width: 256
  height: 256
  num_inference_steps: 50
  guidance_scale: 12.5

# Training parameters
learning_rate: 5e-6
adam_weight_decay: 0
train_batch_size: 1
max_train_steps: 50000

# Allow checkpointing during training (save once every X amount of steps)
checkpointing_steps: 10000

# How many steps during training before we create a sample
validation_steps: 100

# The parameters to unfreeze. As it is now, all attention layers are unfrozen. 
# Unfreezing resnet layers would lead to better quality, but consumes a very large amount of VRAM.
trainable_modules:
  - "attentions"

# Seed for sampling validation
seed: 64

# Use mixed precision for better memory allocation
mixed_precision: "fp16"

# This seems to be incompatible at the moment in my testing.
use_8bit_adam: False

# Currently has no effect.
enable_xformers_memory_efficient_attention: True

```
  </details>
