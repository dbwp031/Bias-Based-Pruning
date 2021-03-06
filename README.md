# Bias-Based-Pruning
###  배치 정규화의 편향 매개변수를 고려한 딥러닝 모델 가지치기 기법 
[2021 IPIU](https://drive.google.com/drive/folders/1UZ6mj-I0F1_5FYEDisoeUlVoldTOl7nm?usp=sharing)


You can train or test ResNet/MobileNet on CIFAR10/CIFAR100/ImageNet.  
Specially, you can train or test on any device (CPU/sinlge GPU/multi GPU) and different device environment available.

You can prune above architectures with our CoBaL method.

----------

## Requirements

- `python 3.5+`
- `pytorch 1.4+`
- `torchvision 0.4+`

----------

## Files

- `config.py`: set configuration
- `data.py`: data loading
- `main.py`: main python file for training or testing
- `models`
  - `__init__.py`
  - `mobilenet.py`
  - `resnet.py`
- `pruning`
  - `dpf`
  - `models`
    - `__init__.py`
    - `mobilenet_mask.py`
    - `resnet_mask.py`
  - `__init__.py`
  - `utils.py`
- `get_flops`: to calculate the number of parameters and FLOPs in model
- `utils.py`

----------

## How to train / test networks

``` text
usage: main.py [-h] [-a ARCH] [--layers N] [--width-mult WM] [--datapath PATH]
               [-j N] [--epochs N] [-b N] [--lr LR] [--acceleration AC][--momentum M]
               [--wd WEIGHT_DECAY] [--nest] [--sched TYPE] [--step-size STEP]
               [--milestones EPOCH [EPOCH ...]] [--gamma GAMMA] [-p N]
               [--ckpt PATH] [-E] [-C] [-g GPU [GPU ...]]
               DATA

positional arguments:
  DATA                  dataset: cifar10 | cifar100 | imagenet (default:
                        cifar10)

optional arguments:
  -h, --help            show this help message and exit
  -a ARCH, --arch ARCH  model architecture: mobilenet | mobilenetv2 | resnet
                        (default: resnet)
  --layers N            number of layers in ResNet (default: 56)
  --width-mult WM       width multiplier to thin a network uniformly at each
                        layer (default: 1.0)
  --datapath PATH       where you want to load/save your dataset? (default:
                        ../data)
  -j N, --workers N     number of data loading workers (default: 8)
  --epochs N            number of total epochs to run (default: 200)
  -b N, --batch-size N  mini-batch size (default: 256), this is the total
                        batch size of all GPUs on the current node when using
                        Data Parallel
  --lr LR, --learning-rate LR
                        initial learning rate (default: 0.1)
  --acceleration        warmup scheme (default: 0)
  --momentum M          momentum (default: 0.9)
  --wd WEIGHT_DECAY, --weight-decay WEIGHT_DECAY
                        weight decay (default: 5e-4)
  --nest, --nesterov    use nesterov momentum?
  --sched TYPE, --scheduler TYPE
                        scheduler: step | multistep | exp | cosine (default:
                        step)
  --step-size STEP      period of learning rate decay / maximum number of
                        iterations for cosine annealing scheduler (default:
                        30)
  --milestones EPOCH [EPOCH ...]
                        list of epoch indices for multi step scheduler (must
                        be increasing) (default: 30 80)
  --gamma GAMMA         multiplicative factor of learning rate decay (default:
                        0.1)
  -p N, --print-freq N  print frequency (default: 100)
  --ckpt PATH           Path of checkpoint for testing model (default: none)
  -E, --evaluate        Test model?
  -C, --cuda            Use cuda?
  -g GPU [GPU ...], --gpuids GPU [GPU ...]
                        GPU IDs for using (default: 0)
                        
pruning argument:
  -P --prune			Use pruning?
  --pruner				method of pruning to apply (default: dpf)
  --prune-type			structure | unstructured (default: unstructured)
  --prune-freq			mask update frequency (default: 16)
  --prune-rate			pruning ratio (default: 0.3)
  --prune-imp			weight importance method: group(optional)_ + conv | bn | coba
  --prune-imptype		weight importance type: L1 | L2 | grad | syn
  --shrink				Exclude zero weights for structured pruning?
```

### Training

#### Train a network using default scheduler (stepLR) with multi-GPU

``` shell
$ python main.py cifar10 -a resnet --layers 56 -C -g 0 1 2 3
```

or

``` shell
$ python main.py cifar10 -a mobilenet -C -g 0 1 2 3
```

#### Train a network using multi-step scheduler with multi-GPU

``` shell
$ python main.py cifar10 -a resnet --layers 56 -C -g 0 1 2 3 --scheduler multistep --milestones 50 100 150 --gamma 0.1
```

### Test

``` shell
$ python main.py cifar10 -a resnet --layers 56 -C -g 0 1 2 3 -E --ckpt ckpt_best.pth
```

### Pruning

#### Prune a network using CoBaL with multi-GPU

```shell
$ python main.py cifar10 -a resnet --layers 56 -C -g 0 1 2 3 -P --pruner dpf --prune-type structured --prune-freq 16 --prune-rate 0.5 --prune-imp coba --prune-imptype L1 --batch-size 128 --epochs 300 --lr 0.2 --wd 1e-4 --nesterov --scheduler multistep --milestones 150 225 --gamma 0.1
```

----------

## References

- [torchvision models github codes](https://github.com/pytorch/vision/tree/master/torchvision/models)
- [MobileNet Cifar GitHub (unofficial)](https://github.com/kuangliu/pytorch-cifar)

