# Dreambooth- Personalized Diffusion Model
This repo aims at using [Dreambooth](https://dreambooth.github.io/) to teach a Diffusion model to learn my pictures and generate images of me from text prompts. I fine-tune stable-diffusion-xl model from Huggingface (over 10GB in size) on a single Turing T4 GPU (16GB) on Google Colab using LoRA and Accelerate from Huggingface. The repo also looks at merging different LoRA adapters in order to merge styles.

## Motivation 
- Can Low-Rank Adapters work well for training Dreambooth?
- Dreambooth works well on pictures of objects. Can it learn to represent human faces well?
- How many images do I need to teach the model about myself?
- What is Prior Preservation?
- How will a model recognize me from the text prompt?
- Can we merge different Adapters to learn different styles?
- What are some difficulties when it comes to training on human faces, and how can we offset them?
- On what text prompts does the model do well, and when does it mess up?


## Jump To
* <a id="jumpto"></a> [Project Structure](#project-structure-)
* <a id="jumpto"></a> [Dataset](#dataset-)
* <a id="jumpto"></a> [Prior-Preservation](#prior-preservation-)
* <a id="jumpto"></a> [Training](#training-)
* <a id="jumpto"></a> [Results](#results-)
* <a id="jumpto"></a> [Merging Adapters](#merging-adapters-)
* <a id="jumpto"></a> [Limitations](#limitations-)
* <a id="jumpto"></a> [References](#references-)

# Project Structure [`↩`](#jumpto)
- The **data** directory contains 7 high resolution images of me. This folder also contains **prior.zip**, which contains 197 images of human faces (excluding my own). These images are used to train the model with prior preservation.
- The **personalized-diffusion.ipynb** contains a notebook to train with and without Prior Preservation. 
- The **train_dreambooth_lora_sdxl.py** notebook contains the model and the script to train it.
- The **dreambooth-inference.ipynb** contains a comprehensive and structured inference of the models trained with and without Prior Preservation. This contains all the images generated from text prompts post-training.


# Dataset [`↩`](#jumpto)
The data contains 7 high-resolution images of me. For Dreambooth, it is important that these images cover different angles and clearly display the face. **According to the experiments, 5-6 images are enough to train stable-diffusion-xl (SDXL) with LoRA.**  For prior preservation, we also use 197 images of other human faces to increase diversity and reduce language drift. These images are generated by **the same** Diffusion model itself.

# Prior-Preservation [`↩`](#jumpto)
Fine-tuning layers that are conditioned on the text embeddings gives rise to the problem of language drift, where a model that is pre-trained on a large text corpus and later fine-tuned for a specific task progressively loses syntactic and semantic knowledge of the language. This phenomenon also affects diffusion models, where the model slowly forgets how to generate subjects of the same class as the target subject.

Another problem is the possibility of reduced output diversity. Text-to-image diffusion models naturally possess high amounts of output diversity. When fine-tuning a small set of images, we would like to be able to generate the subject in novel viewpoints, poses, and articulations. Yet, there is a risk of reducing the amount of variability in the output poses and views of the subject.
To mitigate the two aforementioned issues, the paper proposes an autogenous class-specific prior preservation loss that encourages diversity and counters language drift. The method is to supervise the model with its own generated samples, in order for it to retain the prior once the few-shot fine-tuning begins. This allows it to generate diverse images of the class prior, as well as retain knowledge about the class prior that it can use in conjunction with knowledge about the subject instance.

<img width="432" alt="image" src="https://github.com/satvikkk/dreambooth/assets/57042606/63784b44-dc48-4c8f-b1fd-9fd03698dd83">


# Training [`↩`](#jumpto)
To accommodate such a large model on a 16GB Turing T4 GPU, I make use of gradient accumulation, gradient checkpointing, and 8-bit fused Adam (instead of the regular Adam). Training on 7 images for 1000 steps is conducted with and without the prior preservation loss to verify that the prior preservation actually helps. 

## Training Prompt 
In order to teach the model a mapping between text and a subject, Dreambooth proposes using a rare token from the model's vocabulary and combining it with the subject prior. For instance, to train on my face, I use the prompt

```
A photo of Satvik person
```

Here, Satvik is the rare vocabulary token, and person is the class prior to the subject.


# Results [`↩`](#jumpto)
 
It turns out that LoRA + Dreambooth with 1000 steps works decently well on human faces as well. Prior-Preservation definitely improves the model (as seen from images below). For me, the PNDM Scheduler works well with just 50 timesteps and DDIM with 80 timesteps. 

```
prompt = "a painting of rraj person at Oktoberfest"
```

![wedding]((https://github.com/satvikkk/dreambooth/assets/58619255/473bddd5-7951-4e94-847d-096cc1307b4d))

## Art Renditions
```
prompt = "a painting of rraj person in the style of Van Gogh"
```
![image](https://github.com/rajlm10/dreambooth_lora_merging/assets/57042606/9ac1efc8-0a80-4d20-ad46-a3e40cc48bec)

## Property Modification
```
prompt = "a painting of rraj person with blonde hair"
```
![image](https://github.com/rajlm10/dreambooth_lora_merging/assets/57042606/90ce1e46-c89c-40fd-81d5-dbe886264c23)

## Novel-View Synthesis
```
prompt = "a side view photo of rraj person"
```
![image](https://github.com/rajlm10/dreambooth_lora_merging/assets/57042606/9d077d23-1800-42fb-a2ea-edf0c4fa8d6c)

## Acccesorization
```
prompt = "a of rraj person with sunglasses"
```

![image](https://github.com/rajlm10/dreambooth_lora_merging/assets/57042606/d56dd447-902a-44a1-99c6-3ef7d3f4e720)

# Merging Adapters [`↩`](#jumpto)
I experiment with generating my images in pixel-art style using two merged adapters. Particularly, I experiment with generating my pictures merged with the [Pixel Art](https://huggingface.co/nerijs/pixel-art-xl) style.

```
prompt = "pixel, a photo of rraj person wearing sunglasses"
```
![image](https://github.com/rajlm10/dreambooth_lora_merging/assets/57042606/72e3039d-1c60-46ef-9108-38447397473a)

 


# Limitations [`↩`](#jumpto)
Generating faces is **tough**, sometimes eyes and teeth are not rendered properly or could be mismatches. For instance, below, my eyes are rendered in green but are black in the training images.

![image](https://github.com/rajlm10/dreambooth_lora_merging/assets/57042606/a50fc682-5f7f-4183-9680-5fd28d610915)



## Compute Limits
The GPU did not allow me to fine-tune the text encoder (2 text encoders in the case of SDXL). Fine-tuning text encoders certainly improves image generation quality.


# References [`↩`](#jumpto)
[[1] Huggingface Blog](https://huggingface.co/blog/dreambooth) 

[[2] "DreamBooth: Fine Tuning Text-to-Image Diffusion Models for Subject-Driven Generation" ](https://arxiv.org/abs/2208.12242)

[[3] Dreambooth Diffusers Training Script](https://github.com/huggingface/diffusers/blob/main/examples/dreambooth/train_dreambooth_lora_sdxl.py)
