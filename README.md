# quic-disable-encryption

This describes a method for negotiating the disablement of encryption on QUIC
1-RTT packets, allowing for reduced CPU load and improved performance. This
extension is only meant to be used in environments where both endpoints
completely trust the path between themselves; not, for instance, on the open
internet.
