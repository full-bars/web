# URnetwork FAQ

Welcome to the URnetwork FAQ. Here, we address common questions about how the network operates under the hood, how it compares to traditional VPNs, and how it protects both users and providers.

## 1. What makes URnetwork different from a traditional VPN?

Traditional VPNs tunnel all your internet traffic through a single server. This creates a single point of failure and makes it easy for services to block you.

URnetwork utilizes a **Connection Marketplace** via an optimistic multi-IP auction model. For every single connection you make (like loading different images on a webpage), the client evaluates a "window of providers" and strictly uses the one that performs best. This allows for high availability, naturally circumvents blocked IPs, and means your traffic is distributed rather than centralized. 

## 2. How does URnetwork bypass restrictive firewalls?

Users in heavily restricted networks (like corporate firewalls or countries with strict censorship) often face issues with standard VPN protocols getting blocked.

URnetwork addresses this using **Extenders** (the `UREXTENDER1` protocol). The network employs N-TLS encryption combined with **SNI (Server Name Indication) spoofing** to disguise your traffic. It bounces traffic through intermediary IP extenders, masking the true destination and avoiding automated firewall detection.

*Reference: [`net_extender.go`](https://github.com/urnetwork/connect/blob/main/net_extender.go)*

## 3. Can I use my standard WireGuard app with URnetwork?

Yes! While URnetwork has its own native clients, you can connect using any standard WireGuard application (on iOS, Android, macOS, Windows, or Linux) using the **Tether** feature.

Tether creates a bridge on your local network: it acts as a standard WireGuard server that your app can connect to, but under the hood, it routes all that traffic through the decentralized URnetwork marketplace.

*Reference: [`connect/tetherctl`](https://github.com/urnetwork/connect/tree/main/tetherctl)*

## 4. Do I need to configure Port Forwarding to be a Provider?

No. A common pain point for running a node on other networks is dealing with complicated router settings.

While having direct port forwarding (TCP/UDP) can improve performance, URnetwork natively utilizes **WebRTC** to negotiate peer-to-peer connections. This allows provider nodes to automatically traverse NATs and restrictive router setups without any manual configuration.

*Reference: [`transport_p2p_webrtc.go`](https://github.com/urnetwork/connect/blob/main/transport_p2p_webrtc.go)*
```go
func DefaultWebRtcSettings() *WebRtcSettings {
	return &WebRtcSettings{ ... }
}
```

## 5. Does URnetwork prevent DNS leaks?

Yes. URnetwork guarantees that your internet service provider (ISP) or local network administrator cannot see which websites you are visiting.

The client strictly enforces **DNS-over-HTTPS (DoH)** for all DNS lookups. This secures the connection and resolves domains privately before the first data packet is ever routed.

*Reference: [`net_http_doh.go`](https://github.com/urnetwork/connect/blob/main/net_http_doh.go)*
```go
func DefaultDnsResolverSettings() *DnsResolverSettings {
	return &DnsResolverSettings{
		EnableRemoteDoh: true,
		RemoteDohUrlsIpv4: []string{
			"https://1.1.1.1/dns-query",
		},
        // ...
	}
}
```

## 6. Does URnetwork analyze my traffic?

We care deeply about privacy, which is why URnetwork avoids Deep Packet Inspection (DPI) entirely. 

However, users deserve transparency about which third parties are tracking them. We provide this via the **Inspect** feature. Inspect performs **100% on-device metadata clustering**. By analyzing only the timing and packet headers (without ever reading the payload), the app can group applications and identify third-party trackers locally. This data never leaves your device.

*Reference: [`connect/inspect`](https://github.com/urnetwork/connect/tree/main/inspect)*

## 7. Is it safe to be a Provider?

A common fear when running a public node (like a Tor exit node) is being held liable for abuse or malicious traffic.

URnetwork is built with a **symmetric safety model**. The protocol incorporates automated security intelligence directly into the client. By default, it restricts risky behavior (like specific file-sharing ports and unencrypted standards) via `DefaultEgressSecurityPolicy()`. This establishes a safe baseline, protecting the provider's hardware and legal standing without requiring manual intervention.

*Reference: [`ip_security.go`](https://github.com/urnetwork/connect/blob/main/ip_security.go)*
```go
func DefaultEgressSecurityPolicy() SecurityPolicy {
	return DefaultEgressSecurityPolicyWithStats(DefaultSecurityPolicyStatsCollector())
}
```

## 8. Where can I get more support?

For additional help, technical troubleshooting, or simply to connect with other providers and users, join our community on Discord:

[Join the URnetwork Discord](https://discord.gg/urnetwork)
