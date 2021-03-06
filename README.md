# CartoonGAN-TensorFlow2
Generate your own cartoon-style images with [CartoonGAN (CVPR 2018)](http://openaccess.thecvf.com/content_cvpr_2018/papers/Chen_CartoonGAN_Generative_Adversarial_CVPR_2018_paper.pdf)

## Train your own CartoonGAN

In this section, we will explain how to train a CartoonGAN using the script we provide.

### Setup Environment


### Prepare Dataset


```text
datasets
└── YourDataset [your dataset name]
    ├── testA [(must) 8 real-world images for evaluation]
    ├── trainA [(must) (source) real-world images]
    ├── trainB [(must) (target) cartoon images]
    └── trainB_smooth [(must, but can be generated by running scripts/smooth.py) cartoon images with smooth edges]
```    

`trainA` and `testA` folders contain real-world images, while `trainB` contain images with desired cartoon style. Notice that 8 images in `testA` folder will be evaluated after each epoch, so they should not appear in `trainA`. 

In order to generate `trainB_smooth`, you can run `scripts/smooth.py`:

```
python path/to/smooth.py --path path/to/datasets/YourDataset  # YourDataset should contain trainB for executing this script

```

[smooth.py credit to taki0112 https://github.com/taki0112/CartoonGAN-Tensorflow/blob/master/edge_smooth.py](https://github.com/taki0112/CartoonGAN-Tensorflow/blob/master/edge_smooth.py)

### Start training



```bash
python train.py \
    --batch_size 8 \
    --pretrain_epochs 1 \
    --content_lambda .4 \
    --pretrain_learning_rate 2e-4 \
    --g_adv_lambda 8. \
    --generator_lr 8e-5 \
    --discriminator_lr 3e-5 \
    --style_lambda 25. \
    --light \
    --dataset_name {your dataset name}
```


```bash
python train.py \
    --batch_size 4 \
    --pretrain_epochs 1 \
    --content_lambda .4 \
    --pretrain_learning_rate 1e-4 \
    --g_adv_lambda 8. \
    --generator_lr 4e-5 \
    --discriminator_lr 1.5e-5 \
    --style_lambda 25. \
    --light \
    --dataset_name {your dataset name}
```


### Choose model architecture

Notice that we specified `--light` in our previous example:

```bash
python train.py \
    ...
    --light \
    ...
```


![generator](images/generator.png)

Generator proposed by the original CartoonGAN authors 


To train a CartoonGAN with the original generator/discriminator architecture proposed by the [CartoonGAN](http://openaccess.thecvf.com/content_cvpr_2018/papers/Chen_CartoonGAN_Generative_Adversarial_CVPR_2018_paper.pdf) authors, simply remove `--light` option:

```bash
python train.py \
    --batch_size 8 \
    --pretrain_epochs 1 \
    --content_lambda .4 \
    --pretrain_learning_rate 2e-4 \
    --g_adv_lambda 8. \
    --generator_lr 8e-5 \
    --discriminator_lr 3e-5 \
    --style_lambda 25. \
    --dataset_name {your dataset name}
```

### Inference with trained checkpoint

Once your generator is well-trained, you can try cartoonizing with your trained checkpoint:

```
# You should specify --light if your model is trained with --light
# If you didn't specify --light on your training, you should remove --light
# default of --out_dir is out
python inference_with_ckpt.py \
    --m_path path/to/model/folder \
    --img_path path/to/your/img.jpg \
    --out_dir path/to/your/desired/output/folder \
    --light
```
And generated image will be saved to `path/to/your/desired/output/folder/img.jpg`.

### Export checkpoint to SavedModel and tfjs

Once your generator is well-trained, you can export your model to tfjs model and SavedModel:

```
# You should specify --light if your model is trained with --light
# If you didn't specify --light on your training, you should remove --light
# default of --out_dir is exported_models
python export.py \
    --m_path path/to/model/folder \
    --out_dir path/to/your/desired/export/folder \
    --light
```
And exported tfjs model and SavedModel will be saved to `path/to/your/desired/export/folder`.

Note that the whole model architecture is saved to SavedModel and tfjs model, so you don't need to specify `--light` anymore.

You can try cartoonizing with your exported SavedModel:

```
# default of --out_dir is out
python inference_with_saved_model.py \
    --m_path path/to/your/exported/SavedModelFolder \
    --img_path path/to/your/img.jpg \
    --out_dir path/to/your/desired/output/folder
```

And generated image will be saved to `path/to/your/desired/output/folder/img.jpg`.

We trained 2 model checkpoints and put them in the repo: `light_paprika_ckpt` and `light_shinkai_ckpt` (~7MB Total). You can play around the ckpts with `inference_with_ckpt.py` and `export.py`.

Also, we put our exported shinkai and paprika SavedModels in `exported_models` (~11MB Total). You can play around the SavedModels with `inference_with_saved_model.py`.

Image generated using our `exported_models/light_paprika_SavedModel` (left: original; right: generated):
![origami_demo.jpg](images/origami_demo.jpg)


### (Will be deprecated) Export checkpoint to frozen_pb

This script is just a demontration of backward compatibility.

```
# You should specify --light if your model is trained with --light
# If you didn't specify --light on your training, you should remove --light
# default of --out_dir is optimized_pbs
python to_pb.py \
    --m_path path/to/your/exported/SavedModelFolder \
    --out_dir path/to/your/desired/export/folder \
    --light
```


### [Cartoonize using Colab Notebook](https://colab.research.google.com/drive/1WIZBHix_cYIGsBKa4phIwCq5qXwO8fRX) 

Currently, there are 4 styles available:
- `shinkai`
- `hayao`
- `hosoda`
- `paprika`

For demo purpose, let's assume we want to transform [input_images/temple.jpg](input_images/temple.jpg):

<img src="input_images/temple.jpg" alt="temple" width="33%"/>

To cartoonize this image with `shinkai` and `hayao` styles, you can run:

```commandline
python cartoonize.py \
    --input_dir input_images \
    --output_dir output_images \
    --styles shinkai hayao \
    --comparison_view horizontal
```




## Acknowledgement
- Thanks to the author `[Chen et al., CVPR18]` who published this great work
- [CartoonGAN-Test-Pytorch-Torch](https://github.com/Yijunmaverick/CartoonGAN-Test-Pytorch-Torch) where we extracted pretrained Pytorch model weights for TensorFlow usage
- [TensorFlow](https://www.tensorflow.org/) which provide many useful tutorials for learning TensorFlow 2.0:
    - [Deep Convolutional Generative Adversarial Network](https://www.tensorflow.org/alpha/tutorials/generative/dcgan)
    - [Build a Image Input Pipeline](https://www.tensorflow.org/alpha/tutorials/load_data/images)
    - [Get started with TensorBoard](https://www.tensorflow.org/tensorboard/r2/get_started)
    - [Custom layers](https://www.tensorflow.org/tutorials/eager/custom_layers)
- [Google Colaboratory](https://colab.research.google.com/) which allow us to train the models and [cartoonize images](#cartoonize-using-colab-notebook) using free GPUs
- [TensorFlow.js](https://www.tensorflow.org/js) team which help us a lot when building the [online demo](https://leemeng.tw/generate-anime-using-cartoongan-and-tensorflow2-en.html) for CartoonGAN
- [taki0112/CartoonGAN-Tensorflow](https://github.com/taki0112/CartoonGAN-Tensorflow) where we modify [edge_smooth.py](https://github.com/taki0112/CartoonGAN-Tensorflow/blob/master/edge_smooth.py) to fit our needs 
