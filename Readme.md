### ML Training Techniques
This project summarizes different building blocks of ML training techniques to stabilize and speed up training of deep neural networks. The focus is on the training process itself, not on the model architecture. The goal is to provide a comprehensive overview of the most important techniques and to give a guideline for the practical use of these techniques.
Not considered are model changes, like changing the model architecture or the type of loss function.
The techniques are shown using the MNIST dataset and a simple CNN model. The code is written in Pytorch.
Every Technique is showcased using its own training script
Planned is:
- [Base Case](#base-case)
- [Weighted Data Sampling](#weighted-data-sampling)
- [Progressive Curriculum Learning: Progressive Dropout](#progressive-curriculum-learning-progressive-dropout)
- [Progressive Curriculum Learning: Semi Supervised Learning](#progressive-curriculum-learning-semi-supervised-learning)
- Learning Rate Scheduling
- Random Weight Initialization in Pytorch
- Data Augmentation
- Layerwise Learning Rate & Weight Decay
- Supervised Pretraining (Transfer Learning)
- Self-Supervised Pretraining Masked Image Modeling
- Self-Supervised Contrastive Learning like MoCo
- Distillation for Model Regularization comparison of intermediate layer outputs
- Semi-Supervised Learning

#### Base Case
The base case is a simple `ConvNet` model. The model consists of two convolutional layers followed a linear layer to downproject onto the number of classes. The model is trained for 10 epochs with a batch size of 32 and a learning rate of 0.01. The loss function is the cross-entropy loss. The optimizer is the AdamW optimizer. The model is trained on 10 percent of the training set to be able to easily extend the dataset later. The accuracy is tracked for training and validation.
The according training script is `tr_baseline.py`[🔗](training/tr_baseline.py).

#### Weighted Data Sampling
This technique is used to balance the dataset. The MNIST dataset is already balanced, but the technique is shown for demonstration purposes. The training script is `tr_weighted_sampling.py`[🔗](training/tr_weighted_sampling.py). The script uses the `WeightedRandomSampler` from Pytorch to sample the data. The weights are calculated based on the class distribution of the dataset.  The weights are defined by class and input into the DataModule as 
```python
class_weights=[0.1,0,1,1,1,1,1,1,1,]
```
The above code would mean that the class 0 is sampled with a probability of 0.1, class 1 is not sampled, and all other classes are sampled with a probability of 1. The weights are normalized to sum up to 1. 
#### Progressive Curriculum Learning (CL): Progressive Dropout
As a proxy for changing simple model, regularization or other properties changing the dropout percentage is shown. 
The idea is to start easy with low dropout rates and increase the dropout rate over time. The training script is `tr_progressive_dropout.py`[🔗](training/tr_progressive_dropout.py). A broader study was conducted in the paper [Curriculum Dropout](https://arxiv.org/abs/1703.06229). 
In this implementation the dropout change can be implemented in the training script using a callback function to change the dropout rate of a storage variable `dropout_mem` of the model. 
```python
def linear_dropout(
    epoch: int,
    dropout_mem: dict[int, list[int]],
    start_dropout=0.05,
    end_dropout=1,
    end_epoch=2,
):

    for dropout_layer_key in dropout_mem:
        dropout_mem[dropout_layer_key][0] = min(
            start_dropout + epoch / end_epoch * (end_dropout - start_dropout),
            end_dropout,
        )
```
#### Semi Supervised Self-Learning (Curriculum Learning)
https://arxiv.org/pdf/2101.10382
https://cdn.aaai.org/ojs/16852/16852-13-20346-1-2-20210518.pdf
