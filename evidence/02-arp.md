# Evidence 02 — ARP Table
### Incident IR-2026-0847 · QuantiaPay

---

## Collection Details

| Field | Value |
|-------|-------|
| Command | `arp -a` |
| Collected | 2026-07-18 · ~03:12:05 |
| Host | `jump-02` (10.20.9.140) |
| Volatility | 🔴 High — ARP cache entries expire within minutes |
| Collection order | **#2 of 5** |

---

## Raw Output

```
analyst@quantia-ir:~$ arp -a

Interface: 10.20.9.140 --- 0x4

  Internet Address    Physical Address      Type
  10.20.9.1           aa-bb-cc-11-22-33     dynamic
```

✓ ARP table captured.

---

## Analysis

### What this tells us

| Field | Value | Significance |
|-------|-------|--------------|
| Interface | `10.20.9.140` (0x4) | Confirms active network interface on `jump-02` |
| Gateway IP | `10.20.9.1` | Default gateway for the `10.20.9.0/24` admin subnet |
| Gateway MAC | `aa-bb-cc-11-22-33` | Physical address of the gateway — useful for network topology mapping and verifying no ARP spoofing occurred |
| Entry type | `dynamic` | Normal ARP resolution — no static/poisoned entries detected |

### What this rules out

The ARP table shows only the gateway entry. This means:
- No evidence of **ARP poisoning** (a man-in-the-middle technique that would show unexpected MAC-to-IP mappings)
- No other hosts in the `10.20.9.0/24` subnet were recently communicating with `jump-02` at the time of capture
- The attacker's traffic to `198.51.100.23` was routed through the gateway — consistent with the netstat and firewall evidence

### Network segment context

`jump-02` resides in the `10.20.9.0/24` administrative subnet. This is separate from the production build runner segment (`10.20.4.0/24` where `build-runner-03` lives), confirming that the attacker moved between network segments using the stolen credentials.

---

## Forensic Value

While the ARP table alone is not a smoking gun, it:
1. Confirms the host's network configuration at the moment of evidence capture
2. Rules out ARP-based MITM attacks as a factor in the incident
3. Establishes the gateway MAC for network topology documentation
4. Completes the volatile evidence chain — collected while the attacker connection was still live

---

*Evidence 02 of 5 · Incident IR-2026-0847*
