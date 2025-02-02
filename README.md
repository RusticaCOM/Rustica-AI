
# RusticaAI: A Spatiotemporal Deep Learning Framework

RusticaAI is a spatiotemporal deep learning framework on top of PyTorch and [Apache Sedona](https://sedona.apache.org/). It enable spatiotemporal machine learning practitioners to easily and efficiently implement deep learning models targeting the applications of raster imagery datasets and spatiotemporal non-imagery datasets. Deep learning applications of raster imagery datasets include satellite imagery classification and satellite image segmentation. Applications of deep learning on spatiotemporal non-imagery datasets are mainly prediction tasks which include but are not limited to traffic volume and traffic flow prediction, taxi/bike flow/volume prediction, precipitation forecasting, and weather forecasting.


## RusticaAI Modules
RusticahAI contains various modules for deep learning and data preprocessing in both raster imagery and spatiotemporal non-imagery categories. Deep learning module offers ready-to-use raster and grid datasets, transforms, and neural network models.

<img src="https://github.com/DataSystemsLab/GeoTorchAI/blob/main/data/architecture.png?raw=true" class="center" width="60%" align="right">

* Datasets: This module conatins processed popular datasets for raster data models and grid based spatio-temporal models. Datasets are available as ready-to-use PyTorch datasets.
* Models: These are PyTorch layers for popular raster data models and grid based spatio-temporal models.
* Transforms: Various tranformations operations that can be applied to dataset samples during model training.
* Preprocessing: Supports preprocessing of raster imagery and spatiotemporal non-imagery datasets in a scalable setting on top of Apache Spark and Apache Sedona. 

## Installation
RusticaAI can be installed by running the following command:
```
pip install geotorchai
```
GeoTorchAI is available on [PyPI](https://pypi.org/project/geotorchai/). For more instructions regrading the required and optional dependencies, please visit the [website](https://kanchanchy.github.io/geotorchai/installation.html).

## Example
End-to-end coding examples for various applications including model training and data preprocessing are available in our [binders](https://github.com/DataSystemsLab/GeoTorchAI/tree/main/binders) and [examples](https://github.com/DataSystemsLab/GeoTorchAI/tree/main/examples) sections.

We show a very short example of satellite imagery classification using GeoTorchAI in a step-by-step manner below. Training a satellite imagery classification model consists of three steps: loading the dataset, initializing the model and parameters, and train the model. We pick the [DeepSatV2](https://arxiv.org/abs/1911.07747) model to classify [EuroSAT](https://github.com/phelber/EuroSAT) satellite images.
#### EuroSAT Image Classes
* Annual Crop
* Forest
* Herbaceous Vegetation
* Highway
* Industrial
* Pasture
* Permanent Crop
* Residential
* River
* SeaLake
#### Spectral Bands of a Highway Image
![Highway Image](https://github.com/DataSystemsLab/GeoTorchAI/blob/main/data/euro-highway.png)
#### Spectral Bands of an Industry Image
![Industry Image](https://github.com/DataSystemsLab/GeoTorchAI/blob/main/data/euro-industry.png)
#### Loading Training Dataset
Load the EuroSAT Dataset. Setting download=True will download the full data in the given directory. If data is already available, set download=False.
```
full_data = geotorchai.datasets.raser.EuroSAT(root="data/eurosat", download=True, include_additional_features=True)
```
#### Split data into 80% train and 20% validation parts
```
dataset_size = len(full_data)
indices = list(range(dataset_size))
split = int(np.floor(0.2 * dataset_size))
np.random.seed(random_seed)
np.random.shuffle(indices)
train_indices, val_indices = indices[split:], indices[:split]

train_sampler = torch.utils.data.sampler.SubsetRandomSampler(train_indices)
valid_sampler = torch.utils.data.sampler.SubsetRandomSampler(val_indices)

train_loader = torch.utils.data.DataLoader(full_data, batch_size=16, sampler=train_sampler)
val_loader = torch.utils.data.DataLoader(full_data, batch_size=16, sampler=valid_sampler)
```
#### Train and Evaluate on GPU
If the device used to train the model has GPUs available, then the model, loss function, and tensors can be loaded on GPU. At first initialize the device with CPU or GPU based on the availability of GPU.
```
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
```
Later, model, loss function, and tensors can be loaded to CPU or GPU by calling .to(device). See the exact examples in the later parts.
#### Initializing Model and Parameters
Model initialization parameters such as in_channel, in_width, in_height, and num_classes are based on the property of SAT6 dataset.
```
model = DeepSatV2(in_channels=13, in_height=64, in_width=64, num_classes=10, num_filtered_features=len(full_data.ADDITIONAL_FEATURES))
loss_fn = torch.nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.0002)
# Load model and loss function to GPU or CPU
model.to(device)
loss_fn.to(device)
```
#### Train the Model for One Epoch
```
for i, sample in enumerate(train_loader):
    inputs, labels, features = sample
    # Load tensors to GPU or CPU
    inputs = inputs.to(device)
    features = features.type(torch.FloatTensor).to(device)
    labels = labels.to(device)
    # Forward pass
    outputs = model(inputs, features)
    loss = loss_fn(outputs, labels)
    # Backward pass and optimize
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```
#### Evaluate the Model on Validation Dataset
```
model.eval()
total_sample = 0
correct = 0
for i, sample in enumerate(val_loader):
    inputs, labels, features = sample
    # Load tensors to GPU or CPU
    inputs = inputs.to(device)
    features = features.type(torch.FloatTensor).to(device)
    labels = labels.to(device)
    # Forward pass
    outputs = model(inputs, features)
    total_sample += len(labels)
    _, predicted = outputs.max(1)
    correct += predicted.eq(labels).sum().item()
val_accuracy = 100 * correct / total_sample
print("Validation Accuracy: ", val_accuracy, "%")
```

## Contributing to this Project
Follow the instructions available [here](https://github.com/DataSystemsLab/GeoTorchAI/blob/main/CONTRIBUTING.md).

## Other Contributions of this Project
We also contributed to [Apache Sedona](https://sedona.apache.org/) to add transformation and write supports for GeoTiff raster images. This contribution is also a part of this project. Contribution reference: [Commits](https://github.com/apache/incubator-sedona/commits?author=kanchanchy)

## Citing the Work:
Kanchan Chowdhury and Mohamed Sarwat. 2022. GeoTorch: a spatiotemporal deep learning framework. In Proceedings of the 30th International Conference on Advances in Geographic Information Systems (SIGSPATIAL '22). Association for Computing Machinery, New York, NY, USA, Article 100, 1–4. https://doi.org/10.1145/3557915.3561036

### BibTex:
```
@inproceedings{10.1145/3557915.3561036,
author = {Chowdhury, Kanchan and Sarwat, Mohamed},
title = {GeoTorch: A Spatiotemporal Deep Learning Framework},
year = {2022},
isbn = {9781450395298},
publisher = {Association for Computing Machinery},
url = {https://doi.org/10.1145/3557915.3561036},
doi = {10.1145/3557915.3561036},
articleno = {100},
numpages = {4},
location = {Seattle, Washington},
series = {SIGSPATIAL '22}
}
```


