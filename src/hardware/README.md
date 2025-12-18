# Hardware Design
Include hardware or accelerator implementation files.

BrainChip Akida v1 Datasheet: https://brainchip.com/wp-content/uploads/2025/04/Akida1-IP-Product-Brochure-V2.1-1.pdf


## Running the Software on the Akida Development Board

This project runs inference and benchmarking on the BrainChip Akida Development Board using a Jupyter notebook. The notebook itself is simple to run, but setting up a compatible hardware and software environment is not trivial due to driver and Linux kernel constraints.

### Hardware Requirements

- BrainChip Akida Development Board (PCIe)
- Intel-based host system (tested on Intel Core i7-8700K)
- System with VT-x and VT-d enabled in BIOS
- Available PCIe slot for the Akida board

Newer systems with recent AMD CPUs and modern Linux kernels were found to be incompatible with the current Akida driver version.

### Host and Virtualization Setup

1. Install Proxmox VE on the host machine.
2. Insert the Akida Development Board into a PCIe slot and verify it is detected in BIOS.
3. Enable VT-x, VT-d, and IOMMU in BIOS.
4. Boot into Proxmox and confirm the Akida board is visible:
   
       lspci

### Virtual Machine Configuration

1. Create a new virtual machine in Proxmox with the following configuration:
   - OS: Ubuntu 20.04 LTS
   - CPU type: Host
   - Cores: 4 or more recommended
   - Memory: 8 GB or more recommended
2. Enable PCI passthrough and attach the Akida PCIe device to the VM.
3. Ensure the VM is running a Linux 5.x kernel, as Akida drivers do not support newer kernels.
4. Start the VM and verify the Akida board is visible:
   
       lspci | grep -i akida

### Software Setup (Inside the VM)

1. Update system packages:
   
       sudo apt update && sudo apt upgrade

2. Install Python and required dependencies:
   
       sudo apt install python3 python3-pip python3-venv build-essential

3. Install the Akida SDK:
   
       pip install akida

4. Verify that the board is detected:
   
       python3
       >>> import akida
       >>> akida.devices()

### Running the Jupyter Notebook

1. Install Jupyter:
   
       pip install notebook jupyterlab

2. Launch Jupyter:
   
       jupyter notebook

3. Open the provided notebook and run all cells to:
   - Load the Akida-compatible model
   - Map the model to the Akida device
   - Run inference and collect performance metrics

Example device mapping:
   
       device = akida.devices()[0]
       model.map(device)

### Notes and Known Issues

- Akida drivers require Linux kernel 5.x (newer kernels will not work).
- PCI passthrough on unsupported platforms may cause device timeouts.
- A virtual machine was required to maintain a supported environment.
- USB-based flashing and newer host systems were unreliable in testing.

### Verified Working Configuration

- Host CPU: Intel Core i7-8700K
- Host OS: Proxmox VE
- VM OS: Ubuntu 20.04 LTS
- Kernel: Linux 5.x
- Interface: PCIe passthrough
- Software: Akida SDK and Jupyter Notebook
