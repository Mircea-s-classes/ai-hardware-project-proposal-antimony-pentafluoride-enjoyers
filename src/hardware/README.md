# Hardware Design
Include hardware or accelerator implementation files.

Brainchip Akida v1 Datasheet: https://brainchip.com/wp-content/uploads/2025/04/Akida1-IP-Product-Brochure-V2.1-1.pdf
Brainchip Akida PCIe Board User Guide: https://brainchip.com/wp-content/uploads/2022/06/Akida-PCIe_Board_UG_200c.pdf

## Running the Software on the Akida Development Board

This project runs inference and benchmarking on the Brainchip PCIe Board using a Jupyter notebook. The notebook itself is simple to run, but setting up a compatible hardware and software environment is not trivial due to driver and Linux kernel constraints.

### Hardware Requirements

- Brainchip Akida PCIe Board 
- x86-64-based host system (tested on Intel Core i7-8700K)
- System with support for Ubuntu with Linux Kernel 5.x
- Available PCIe slot for the Akida board

Newer systems with recent AMD CPUs and modern Linux kernels were found to be incompatible with the current Akida driver version.

### Failed Host and Virtualization Setup

1. Install Ubuntu 24.04.3 LTS via Dual Booting on an AMD 9950x3D system
   - First USB drive failed, had to buy a new one to install the OS
   - OS was not compatible with the Akida PCIe Board
   - Accidentally wiped original OS from the machine and lost all the files on it
3. Install Proxmox VE on the host machine.
4. Insert the Akida Development Board into a PCIe slot and verify it is detected in BIOS.
5. Enable VT-x, VT-d, and IOMMU in BIOS.
6. Boot into Proxmox and setup Ubunutu VM

### Failed Virtual Machine Configuration

1. Create a new virtual machine in Proxmox with the following configuration:
   - OS: Ubuntu 20.04 LTS
   - CPU type: Host (AMD 9950x3D)
   - Cores: 16
   - Memory: 32 GB
2. Enable PCI passthrough and attach the Akida PCIe device to the VM.
3. Ensure the VM is running a Linux 5.x kernel, as Akida drivers do not support newer kernels.
4. Start the VM and verify the Akida board is visible:
   
       lspci | grep -i Co-processor
   
       Output: Co-processor: Device 1e7c:bca1 (rev 01)
   
This setup was able to locate the Akida board, but DMA connections kept on timing out, so we switched to a different desktop computer.

### Working Hardware Setup
1. Intel i8700k Processor.
2. Inserted a 2TB Samsung 990 SSD in the motherboard's M.2 Slot and put the Akida PCIe board in a PCIe slot.
3. Installed Ubuntu 20.04 LTS on the SSD via bootable flash drive,

### Software Setup

1. Update system packages:
   
       sudo apt update && sudo apt upgrade

2. Install Python and required dependencies:
   
       sudo apt install python3 python3-pip python3-venv build-essential

3. Install the Akida PCIe Board Drivers

       lspci | grep Co-processor
       git clone https://github.com/Brainchip-Inc/akida_dw_edma
       sudo apt install build-essential linux-headers-$(uname -r)
       cd akida_dw_edma
       sudo ./install.sh
     
4. Install the Akida SDK:
   
       pip install akida

5. Verify that the board is detected in Python:
   
       python3
       >>> import akida
       >>> akida.devices()

6. Install Extra Python Modules

         pip install numpy tf_keras quantizeml tensorflow cnn2snn matplotlib pandas

### Running the Jupyter Notebook

1. Install Jupyter:
   
       pip install notebook

2. Launch Jupyter:
   
       jupyter notebook

3. Open the provided notebook and run all cells to:
   - Load and train the custom Akida-compatible CNN
   - Convert the CNN to an SNN
   - Map the SNN to the Akida device
   - Run inference and collect performance metrics (CNN ==> CPU, SNN ==> Akida PCIe Board)

### Final Hardware Configuration

- Host CPU: Intel Core i7-8700K
- OS: Ubuntu 20.04 LTS
- Kernel: Linux 5.x
- Interface: PCIe
- Software: Akida SDK, Jupyter Notebook, and Python
