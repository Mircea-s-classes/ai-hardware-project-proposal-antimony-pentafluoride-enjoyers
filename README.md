# Comparison of CNNs and SNNs on the MNIST Dataset Using the BrainChip Akida AI Processor

**Team:** Antimony Pentafluoride Enjoyers  
**Students:** Alex Clunan, Bobby Downey  
**Course:** ECE 4380 / ECE 6380 — AI Hardware Design and Implementation (Fall 2025)  

---

## Introduction
This project investigates the performance and efficiency of **Convolutional Neural Networks (CNNs)** and **Spiking Neural Networks (SNNs)** on the MNIST dataset using both CPU and BrainChip Akida hardware. Our goal is to compare classification accuracy, inference latency, and energy efficiency while highlighting the benefits and trade-offs of neuromorphic computation. We use the Akida neuromorphic platform for running SNNs and compare them against conventional CNNs running on CPU.

---

## Background
CNNs are widely used for image classification due to their high accuracy. However, their computations are continuous and energy-intensive. SNNs emulate biological neurons using event-driven computation, which triggers processing only when spikes occur, potentially reducing energy usage. This architecture allows neuromorphic systems like Akida to perform low-power inference suitable for edge devices.

---

## Method
The dataset used is MNIST, containing 60,000 training images and 10,000 testing images. For the CNN, images are normalized and processed conventionally. For the SNN, images are converted into spike trains using either rate or temporal encoding to simulate neuron firing.

The baseline CNN was implemented in PyTorch and trained for five epochs. We quantized the model to 4-bit weights and activations to prepare it for conversion to an SNN using Akida's tools. The SNN was then deployed on the Akida Development Kit, and benchmarks were conducted to measure throughput, latency, and energy consumption. The general pipeline for preparing the CNN and SNN models is shown below, followed by a call to our benchmarking function.


**CNN to SNN Flow Diagram**  
Flow Diagram from CNN to Quantized CNN to SNN using Akida SDK.  

| ![CNN2SNN_Flow](CNN2SNN_Flow.png) |
|-----------------------|

```python
import numpy as np
import tf_keras as keras
from tf_keras.datasets import mnist

# load and preprocess dataset
(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train = np.expand_dims(x_train, -1).astype(np.uint8)
x_test  = np.expand_dims(x_test,  -1).astype(np.uint8)
y_train = y_train.astype(np.int32)
y_test  = y_test.astype(np.int32)

# define the CNN <odel
def make_mnist_cnn_v1_safe():
    return keras.Sequential([
        keras.layers.Input(shape=(28,28,1)),
        keras.layers.Rescaling(1./255),
        keras.layers.Conv2D(16, 3, padding="same", use_bias=True),
        keras.layers.ReLU(max_value=6.0),
        keras.layers.Conv2D(32, 3, strides=2, padding="same", use_bias=True),
        keras.layers.ReLU(max_value=6.0),
        keras.layers.Conv2D(64, 3, strides=2, padding="same", use_bias=True),
        keras.layers.ReLU(max_value=6.0),
        keras.layers.Flatten(),
        keras.layers.Dense(10),
    ], name="mnist_akida_v1_safe")

model_fp32 = make_mnist_cnn_v1_safe()
model_fp32.compile(
    optimizer=keras.optimizers.Adam(1e-3),
    loss=keras.losses.SparseCategoricalCrossentropy(from_logits=True),
    metrics=["accuracy"],
)
model_fp32.fit(x_train, y_train, epochs=5, batch_size=256, validation_split=0.1)

# quantize the model
from quantizeml.models import quantize, QuantizationParams
qparams = QuantizationParams(input_weight_bits=4, weight_bits=4, activation_bits=4)
model_q = quantize(model_fp32, qparams=qparams, samples=x_train, num_samples=2048, batch_size=256, epochs=1)

# convert the CNN model to Akida SNN for deployment on the BrainChip Akida
from cnn2snn import convert, set_akida_version, AkidaVersion
import akida

with set_akida_version(AkidaVersion.v1):
    model_ak_v1 = convert(model_q)

dev = akida.devices()[0]
model_ak_v1.map(dev)
dev.soc.power_measurement_enabled = True

# our benchmarking function usage
stats_ak = benchmark_akida_with_statistics(model_ak_v1, dev, x_test, y_test, batch_size=256, n_warmup=20, n_iters=3000, verbose=True)
```


## Experiments and Results

Our experiments evaluated both the CNN running on CPU and the SNN deployed on the Akida Development Board. We tested with batch sizes of 1 and 256, recording classification accuracy, throughput, inference latency, and energy per image. Overall, the CNN achieved slightly higher accuracy, while the SNN on Akida demonstrated a substantial reduction in energy consumption per inference, highlighting the advantages of neuromorphic hardware.

| Platform | Batch | Accuracy | Throughput (img/s) | Energy per image |
|----------|-------|---------|------------------|----------------|
| CPU      | 256   | 98.42% | 56,824           | 0.00176 J      |
| CPU      | 1     | 98.42% | 2,651            | 0.01398 J      |
| Akida    | 256   | 96.01% | 4,455            | 0.24 mJ        |
| Akida    | 1     | 96.01% | 1,489            | 0.44 mJ        |

### CPU vs Akida Performance Ratios
**Batch = 256**  
- Accuracy: CPU 98.42%, Akida 96.01%, Δ = 2.41 percentage points  
- Throughput: CPU / Akida speedup = 12.75×  
- Energy per image: CPU / Akida energy ratio = 7.34×  
- Average power: CPU / Akida power ratio = 91.42×  
- Per-image latency: CPU / Akida latency ratio = 0.08×

**Batch = 1**  
- Accuracy: CPU 98.42%, Akida 96.01%, Δ = 2.41 percentage points  
- Throughput: CPU / Akida speedup = 1.78×  
- Energy per image: CPU / Akida energy ratio = 31.77×  
- Average power: CPU / Akida power ratio = 37.18×  
- Per-image latency: CPU / Akida latency ratio = 0.56×

### Hardware Setup Challenges
Setting up the Akida Development Board proved to be time-consuming and complex. Initially, we tried using a new computer with an AMD 9950X3D CPU and RTX 5070 GPU. During Linux installation, we encountered a corrupted flash drive that required replacement. The Akida drivers only supported Linux kernel version 5, which the new computer could not run. We attempted to use a virtual machine via Proxmox running Ubuntu 20.04 (the supported Linux version), which allowed PCI passthrough, but frequent connection timeouts made this setup unusable. Eventually, we switched to an older computer in the Rice 240 CPE lab which uses an Intel Core i7-8700K CPU. After failed dual-boot attempts, we successfully used Proxmox with a VM running Ubuntu 20.04, allowing the environment to finally function for deployment and benchmarking.

**Hardware Used**  
AMD 9950x3d Desktop and Akida PCIe Board.  

| ![desktop](desktop.png) | ![board](board.png) |
|------------------------------------------------------|----------------------------------------------------------------------|

### Graphs and Interpretation

To make the report more compact and visually organized, we grouped related graphs side by side. Each group shows complementary performance metrics comparing CPU vs Akida behavior.

**Batch Latency (P99, P90, P50)**  
Shows latency percentiles for each batch size, highlighting worst-case, near-worst, and median latencies. Arranging these together allows easy comparison of latency distribution.  

| ![LATP99](LATP99.png) | ![LATP90](LATP90.png) | ![LATP50](LATP50.png) |
|------------------------|------------------------|------------------------|

**Power and Energy Metrics**  
Average power consumption and energy per image across batch sizes. Viewing these together emphasizes Akida's energy efficiency advantage.  

| ![power_versus_batchsize](power_versus_batchsize.png) | ![energy_per_image_versus_batchsize](energy_per_image_versus_batchsize.png) |
|------------------------------------------------------|----------------------------------------------------------------------|

**Latency and Throughput per Image**  
Per-image latency versus batch size alongside throughput shows how batch processing affects performance on CPU and Akida.  

| ![latency_versus_batchsize](latency_versus_batchsize.png) | ![throughput_versus_batchsize](throughput_versus_batchsize.png) |
|-----------------------------------------------------------|-----------------------------------------------------------------|

**Accuracy vs Batch Size**  
Compares classification accuracy across batch sizes for CPU vs Akida, highlighting any degradation in SNN performance.  

| ![accuracy_versus_batchsize](accuracy_versus_batchsize.png) |
|-------------------------------------------------------------|

**Pareto: Throughput vs Energy per Image**  
Plots the trade-off between throughput and energy, showing Akida’s efficiency advantage relative to CPU.  

| ![pareto](pareto.png) |
|-----------------------|

---

## Conclusion
The Akida SNN provides energy-efficient inference while maintaining reasonable accuracy on the MNIST dataset. CPU-based CNNs offer higher throughput and marginally better accuracy, but their energy consumption is orders of magnitude higher. The setup experience illustrates that while the hardware offers substantial benefits, integrating it with modern systems can be challenging due to compatibility issues and driver limitations. Using older, compatible lab computers in combination with a virtualized Ubuntu 20.04 environment ultimately allowed successful deployment. 

These results highlight that neuromorphic hardware is promising for low-power, edge AI applications, but careful attention to hardware and software compatibility is required. Future work could explore temporal encoding on a different platform which better supports it, Akida Edge Learning, and scaling to more complex datasets to further leverage the benefits of SNNs.

---

## References
- BrainChip Akida Documentation: https://brainchip.com/developer  
- Diehl, P.U. & Cook, M., “Unsupervised Learning of Digit Recognition Using Spike-Timing-Dependent Plasticity,” Frontiers in Computational Neuroscience, 2015  
- Akida User Guide: https://doc.brainchipinc.com  
- BrainChip Developer Docs: https://developer.brainchip.com/docs/  
- AKD1500 Product Brief V2.5 (Nov 25, 2025): https://brainchip.com/wp-content/uploads/2025/11/AKD1500-Product-Brief-V2.5-Nov.25.pdf  
- BrainChip Documentation Index: https://doc.brainchipinc.com/index.html

