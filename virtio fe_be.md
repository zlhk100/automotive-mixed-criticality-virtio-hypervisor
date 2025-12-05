```mermaid
sequenceDiagram
    autonumber
    participant App as App (Guest)
    participant VMD as Virtio Frontend Driver
    participant KVM as Hypervisor (KVM)
    participant QEMU as Host Backend (QEMU)
    participant HostOS as Host Kernel
    
    %% I. Command Submission
    App->>VMD: 1. I/O Request (read/write)
    VMD->>VMD: 2. Add Descriptors to Virtqueue
    VMD->>KVM: 3. VM EXIT / Kick Notification
    
    %% II. Host Processing
    KVM->>QEMU: 4. Signal QEMU (via eventfd/IPC)
    QEMU->>HostOS: 5. Execute Native I/O
    HostOS-->>QEMU: 6. I/O Completion
    
    %% III. Response Injection
    QEMU->>VMD: 7. Update Used Ring
    QEMU->>KVM: 8. Signal Virtual IRQ (irqfd)
    KVM->>VMD: 9. Inject Virtual Interrupt
    
    %% IV. Guest Completion
    VMD->>App: 10. Wake up waiting thread
