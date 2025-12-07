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
    GuestVMM->>KVM: virtqueue_kick (VM EXIT I/O Trap)
    
    Note over KVM, QEMU: II. HOST PROCESSING
    KVM->>QEMU: Wakeup (via eventfd or UNIX socket IPC)
    QEMU->>QEMU: BE/vhost_worker_loop (virtqueue_pop(...) / Read Available Ring)
    QEMU->>HostKernel: Native I/O syscall (Access Disk/Network)
    HostKernel-->>QEMU: I/O Completion
    
    Note over QEMU, KVM: III. RESPONSE INJECTION
    QEMU->>QEMU: virtqueue_push (Update Used Ring)
    QEMU->>HostKernel: virtqueue_notify / KVM_IOCTL_IRQFD (Signal Virtual IRQ)
    HostKernel->>KVM: Handle eventfd write
    KVM->>GuestVMM: Inject Virtual Interrupt (The vring_interrupt)
    
    Note over GuestVMM, App: IV. GUEST COMPLETION
    GuestVMM->>GuestVMM: Guest Kernel IRQ Handler
    GuestVMM->>GuestVMM: virtqueue_get_buf (Process Used Ring)
    GuestVMM-->>App: syscall_return (Wake up waiting application)
