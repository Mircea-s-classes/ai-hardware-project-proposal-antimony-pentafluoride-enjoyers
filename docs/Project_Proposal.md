# University of Virginia
## Department of Electrical and Computer Engineering

**Course:** ECE 4332 / ECE 6332 — AI Hardware Design and Implementation  
**Semester:** Fall 2025  
**Proposal Deadline:** November 5, 2025 — 11:59 PM  
**Submission:** Upload to Canvas (PDF) and to GitHub (`/docs` folder)

---

# AI Hardware Project Proposal

## 1. Comparison of CNNs and SNNs on the MNIST Dataset Using the BrainChip Akida AI Processor

**Team Name:** Antimony Pentafluoride Enjoyers  
**Students:** Alex Clunan, Bobby Downey  

---

## 2. Platform Selection
**Selected Platform:** BrainChip Akida (Neuromorphic Computing Platform)

**Justification:**  
The BrainChip Akida processor is a hardware environment optimized for spiking neural networks. Its neuromorphic architecture allows inference that is energy-efficient specifically for tasks in on-chip learning.  
Because of its optimized architecture, the BrainChip Akida can natively run SNNs which enables optimized performance. We will compare this with the same CNN running on a CPU/GPU.

---

## 3. Problem Definition
As discussed in class, convolutional neural networks (CNNs) perform well on image classification tasks such as MNIST, but due to their size, they often result in substantial energy consumption.  
Spiking neural networks (SNNs) emulate biological neural systems using event-driven computation. Since computation is only triggered by discrete events that are often sparse in time, there is most often much less parameters than for CNNs, potentially offering significant improvements in energy efficiency.

The goal of this project is to compare CNN and SNN performance on the MNIST dataset using the a CPU/GPU and Akida processor respectively, focusing specifically on metrics of accuracy, inference latency, and energy efficiency.  

---

## 4. Technical Objectives
1. Implement SNN using the Akida Development Kit and PyTorch and the CNN using PyTorch for MNIST classification.  
2. Measure differences in accuracy, energy consumption, and inference latency between CNN and SNN models.    
3. Perform these comparisons with both rate encoding and temporal encoding for SNN
4. Visualize these realtive metrics with plots and analyze.  
5. If time permits, compare the performance of the same SNN that uses Akida Edge Learning to learn the same dataset.
6. Provide code and documentation for benchmarking CNNs vs. SNNs on Akida hardware.

---

## 5. Methodology
1. **Dataset:**  
   - Use the standard MNIST dataset (60,000 training, 10,000 testing images).  
   - Convert static images into spike trains for the SNN model using rate and temporal encoding.

2. **Model Development:**  
   - Implement a baseline CNN for MNIST using the Akida Python SDK or some other compatible PyTorch model.  
   - Convert the CNN into an equivalent SNN using Akida’s conversion tools.  
   - Deploy both models onto the Akida Development Board for benchmarking of inferences.

3. **Hardware and Software Tools:**  
   - **Hardware:** BrainChip Akida Development Board, a PC compatable with the dev board  
   - **Software:** Akida Python SDK, PyTorch, NumPy, Matplotlib, MNIST dataset

4. **Evaluation Metrics:**  
   - Classification accuracy (%)  
   - Energy consumption (mJ/inference)  
   - Inference latency (ms/inference)  


## 6. Expected Deliverables
- Fully implemented CNN and SNN MNIST classifiers  
- Comparative report analyzing accuracy, latency, and energy efficiency  
- GitHub repository with reproducible code and setup instructions  
- Final report and presentation slides summarizing findings
- If time, compare the a normal SNN with another that uses Akida Edge Learning 

---

## 7. Team Responsibilities
| Name | Role | Responsibilities |
|------|------|------------------|
| Alex Clunan | Hardware and Deployment Lead, Anitmony Pentafluoride Merchant | Hardware Setup, SNN Deployment |
| Bobby Downey | Software and Analysis Lead | CNN/SNN training, data preprocessing, results analysis |


---

## 8. Timeline and Milestones
| Week | Milestone | Deliverable |
|------|------------|-------------|
| 2 | Proposal Submission | Project proposal (PDF and GitHub upload) |
| 4 | CNN and SNN Implementation | Baseline models tested in simulation |
| 6 | Hardware Deployment | CNN and SNN running | Evaluation & Comparison | Performance and energy benchmarking |
| Dec. 18 | Final Presentation | Report, slides, and GitHub repository | 

---

## 9. Resources Required
- BrainChip Akida Development Kit
- GPU
- MNIST Dataset  
- Akida Python SDK and documentation  
- Python measurement tools 

---

## 10. References
1. BrainChip Akida Documentation: https://brainchip.com/developer  
2. Diehl, P.U. & Cook, M., “Unsupervised Learning of Digit Recognition Using Spike-Timing-Dependent Plasticity,” *Frontiers in Computational Neuroscience*, 2015.  
3. Akida User Guide: https://doc.brainchipinc.com/user_guide/user_guide.html 
