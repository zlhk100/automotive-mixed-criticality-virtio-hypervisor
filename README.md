# automotive-mixed-criticality-virtio-hypervisor
Discuss automotive mixed criticality and virtualization technologies

```mermaid
sequenceDiagram
    autonumber
    participant App
    participant GuestVMM as Guest Kernel (Frontend)
    participant KVM
    participant QEMU as Host VMM (Backend)
    participant HostKernel
    
    Note over App, GuestVMM: I. COMMAND SUBMISSION
    App->>GuestVMM: syscall: read/ioctl (I/O Request)
    GuestVMM->>GuestVMM: virtqueue_add_buf (Populate Descriptor)
    GuestVMM->>GuestVMM: vring_avail_update (Update Available Ring)
    GuestVMM->>KVM: VM EXIT (I/O Trap/Kick)
    
    Note over KVM, QEMU: II. HOST PROCESSING
    KVM->>QEMU: Wakeup (via eventfd or UNIX socket IPC)
    QEMU->>QEMU: vhost_worker_loop (Read Available Ring)
    QEMU->>HostKernel: Native I/O syscall (Access Disk/Network)
    HostKernel-->>QEMU: I/O Completion
    
    Note over QEMU, KVM: III. RESPONSE INJECTION
    QEMU->>QEMU: vring_used_update (Update Used Ring)
    QEMU->>HostKernel: KVM_IOCTL_IRQFD (Signal Virtual IRQ)
    HostKernel->>KVM: Handle eventfd write
    KVM->>GuestVMM: Inject Virtual Interrupt (The vring_interrupt)
    
    Note over GuestVMM, App: IV. GUEST COMPLETION
    GuestVMM->>GuestVMM: Guest Kernel IRQ Handler
    GuestVMM->>GuestVMM: vring_get_buf (Process Used Ring)
    GuestVMM-->>App: syscall_return (Wake up waiting application)
