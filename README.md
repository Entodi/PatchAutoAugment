# Patch AutoAugment
Learning the optimal augmentation policies for different regions of an image and achieving the joint optimal on the whole image.

## Introduction
Patch AutoAugment implementation in PyTorch.
The paper can be found [here](https://arxiv.org/abs/2103.11099).

<img src=https://github.com/LinShiqi047/PatchAutoAugment/blob/main/figure/imagelevel_v.s_patchlevel.jpg width=600 height=350 />
<img src=https://github.com/LinShiqi047/PatchAutoAugment/blob/main/figure/framework.jpg />

## Preparation
### Install
This project is run on GPU (NVIDIA 2TX 2080Ti).
We conduct experiments under python 3.8, pytorch 1.6.0, cuda 10.1 and cudnn7. 
You can download [dockerfile](https://github.com/LinShiqi047/PatchAutoAugment/blob/main/Dockerfile).

We use [Kornia](https://github.com/kornia/kornia), a differentiable computer vision library that can be used to accelerate augmentation operations on tensors.

We are working with the official [Kornia](https://github.com/kornia/kornia) team to integrate PAA into Kornia package. The Kornia-build-in PAA is coming soon.

### Data preparation
Here, we take CIFAR and fine-grained datasets [CUB-200-2011](http://www.vision.caltech.edu/visipedia/CUB-200-2011.html), [Stanford Dogs](http://vision.stanford.edu/aditya86/ImageNetDogs/) to illustrate how to use our PatchAutoAugment.

## Example
First, import PAA module.
```
from augs.PAA import *   
from augs.A2Cmodel import ActorCritic
```
Then, 
```
if args.aug == 'PAA':
        model_a2c = ActorCritic(len(ops)).cuda()
        optimizer_a2c = torch.optim.SGD(model_a2c.parameters(), hyper_params['lr_a2c'], 
                                    momentum=0.9, nesterov=True,
                                    weight_decay=1e-4)
        scheduler_a2c = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer_a2c, len(train_loader)* hyper_params['epochs'])
```
Thirdly, 
```
if args.aug == 'PAA':
    losses_a2c = AverageMeter()
    model_a2c.train()

    
    for i, (input, target) in enumerate(train_loader):
        target = target.cuda(non_blocking=True)
        input = input.cuda(non_blocking=True)

        if args.aug == 'PAA':
            # patch data augmentation
            operations ,operation_logprob , operation_entropy , state_value = \
                rl_choose_action(input, model_a2c)
            input_aug = patch_auto_augment(input, operations, args.batch_size, epochs=hyper_params['epochs'], epoch=epoch)

            output = model(input_aug.detach())
            loss = criterion(output, target)

            # PAA loss
            reward = loss.detach()
            loss_a2c = a2closs(operation_logprob, operation_entropy, state_value, reward)

            # update PAA model
            optimizer_a2c.zero_grad()
            loss_a2c.backward()
            optimizer_a2c.step()
            scheduler_a2c.step()
```
### Example - CIFAR10
For example, train Wide-ResNet-28-10, PyramidNet+ShakeDrop on CIFAR-10 
```
python /code/CIFAR/train.py --dataset cifar10 --model WRN --aug PAA
python /code/CIFAR/train.py --dataset cifar10 --model SD --aug PAA --lr_a2c 0.0001
```
Some available options:
- ```--dataset```: Training and testing dataset, support cifar10 | cifar100
- ```--batch-size```: Batch size
- ```--model```: Target training network, support Wide-ResNet-28-10 (WRN) | ShakeShake (SS) | PyramidNet+ShakeDrop (SD)
- ```--SS_w_base```: ShakeShake (26 2x32d) | SS (26 2x96d) | SS (26 2x112d)
- ```--aug```: Augmentation method, support baseline (base), Cutout (cutout), AutoAugment+Cutout (AA), AutoAugment (onlyAA), PatchAutoAugment (PAA)
- ```--lr_a2c```: PAA learning rate, default = 1e-3

### Example - Fine-grained dataset
For example, train ResNet-50 (pretrained) on Stanford Dogs
```
python /code/Fine_grained_dataset/tools/train.py --cfg /code/DOG/experiments/cls_res50.yaml --AUG PAA --N_GRID 4 --DATASET dog --IMAGE_SIZE 224 --EPOCHS 50 --BATCH_SIZE 32
```
Some available options:
- ```--AUG```: Augmentation method, support baseline (base), AutoAugment (AA), PatchAutoAugment (PAA)
- ```--N_GRID```: Number of patches, support 1 | 2 | 4 | 7 | 14.
- ```--DATASET```: Training and testing dataset, CUB-200-2011 (cub) | Stanford Dogs (dog).
- ```--IMAGE_SIZE```: Image size, support 224 | 448.
- ```--EPOCHS```: Training epochs.
- ```--BATCH_SIZE```: Batch size.

### Example - Kornia PatchSequential
Further, we use the patchSequential module of the container in Kornia to implement PAA.
```
python /code/Kornia_PAA/tools/train.py --cfg /code/DOG/experiments/cls_res50.yaml --AUG PAA --N_GRID 4 --DATASET dog --IMAGE_SIZE 224 --EPOCHS 50 --BATCH_SIZE 32
```

The Kornia-build-in PAA is coming soon.

## Cite Us
Please cite us if you find this work helps.
```
@article{lin2021patch,
  title={Patch AutoAugment},
  author={Lin, Shiqi and Yu, Tao and Feng, Ruoyu and Chen, Zhibo},
  journal={arXiv preprint arXiv:2103.11099},
  year={2021}
}
```
