# Sevana Voice Quality MCP Server

**Remote [MCP](https://modelcontextprotocol.io) server for voice and VoIP call‑quality analysis.**
Let AI agents measure speech quality, score MOS, and analyze packet captures — over the
Model Context Protocol, with no local install.

- **Endpoint:** `https://mcp.sevana.biz/mcp` (Streamable HTTP)
- **Free tier, no sign‑up.** An API key raises the limits and unlocks on‑premises deployment.
- Works with any MCP client that supports remote servers: Claude Code, Claude Desktop,
  Cursor, OpenCode, Cline, Continue, and others.

It exposes the production analysis engines behind [Sevana](https://sevana.biz)'s products —
**PVQA**, **AQuA** and **Pcaptix** — so an agent can go from a WAV recording or a `.pcap`
capture to MOS scores, impairments and per‑stream VoIP statistics.

---

## What it does

| Capability | Tool(s) | What you get |
|---|---|---|
| **Non‑intrusive MOS** (no reference needed) | `pvqa_analyze`, `pvqa_detector`, `pvqa_timeline` | Predicted MOS (1–5) and the impairments in a single WAV: low SNR, noise, dead air, clipping, clicks, echo, packet‑loss artifacts. A **P.563 / P.564 alternative.** |
| **Intrusive MOS** (reference vs test) | `aqua_compare` | MOS, percent similarity and a fault analysis explaining the loss of quality between a reference WAV and a degraded one. A **POLQA / PESQ / ViSQOL alternative.** |
| **VoIP capture analysis** | `pcap_analyze`, `pcap_summary`, `pcap_full_report`, `pcap_get_call`, `pcap_get_stream`, `pcap_network_stats`, `pcap_sip_messages`, `pcap_dtmf` | Reconstructs SIP calls and RTP streams from a `.pcap`/`.pcapng` and scores every stream: Sevana MOS from decoded speech, network MOS, loss, jitter, delay, R‑factor, echo, DTMF (RFC 2833/4733), the SIP message ladder, and health warnings. Analyze VoIP/VoLTE quality **without Wireshark.** |
| **File upload** | `create_upload`, `upload_inline`, `delete_upload` | One‑time upload URL for WAV/PCAP (PUT), or small files inline as base64. |
| **Jobs & info** | `job_status`, `job_cancel`, `list_jobs`, `server_info` | Track analysis jobs; check analyzer versions, your tier and remaining quota. |

19 tools in total. Full, self‑describing schemas are returned by the server's `tools/list`.

---

## Connect

### Claude Code
```bash
claude mcp add --transport http sevana https://mcp.sevana.biz/mcp
```

### Cursor / Claude Desktop / generic JSON config
```json
{
  "mcpServers": {
    "sevana": {
      "url": "https://mcp.sevana.biz/mcp"
    }
  }
}
```

### OpenCode (`opencode.json`)
```json
{
  "mcp": {
    "sevana": {
      "type": "remote",
      "url": "https://mcp.sevana.biz/mcp"
    }
  }
}
```

With an API key, add an `Authorization: Bearer YOUR_API_KEY` header. Leave it out to use the
free tier.

---

## Example agent flow

1. `create_upload` with `kind: "pcap"` → returns an `upload_id` and a one‑time `upload_url`.
2. `PUT` the capture file to that URL (e.g. `curl -T call.pcap "<upload_url>"`).
3. `pcap_analyze` with the `upload_id` → returns a `job_id`; it waits up to `wait_seconds`.
4. `pcap_summary` / `pcap_get_stream` → worst‑MOS streams, codecs, loss, jitter, impairments.

For a single recording: `upload_inline` (or `create_upload` + PUT) with `kind: "wav"`, then
`pvqa_analyze` for non‑intrusive MOS, or `aqua_compare` for reference‑vs‑test MOS.

---

## Free tier and API keys

| | Free tier (no key) | With an API key |
|---|---|---|
| Analyses | 10 per hour | up to 1000 per plan |
| WAV length | up to 2 minutes, 100 MB | up to 30 minutes, 200 MB |
| PCAP size | up to 10 MB | up to 200 MB |

For an API key, higher limits or an **on‑premises** installation, see the
[product page](https://sevana.biz/mcp-server-voice-quality-ai-agents/) or
[contact Sevana](https://sevana.biz/contact-us-your-call-quality-partner/).

---

## About

Built by [**Sevana OU**](https://sevana.biz) — voice and audio quality analysis for telecom.
The same engines power Sevana's desktop and server products:

- **[PVQA](https://sevana.biz/pvqa-passive-call-quality-analysis/)** — passive (non‑intrusive) voice quality analyzer.
- **[AQuA](https://sevana.biz/products-aqua-polqa-visqol/)** — intrusive audio quality analyzer (reference vs test).
- **[Pcaptix](https://sevana.biz/our-products-pcaptix/)** — VoIP/VoLTE call‑quality analysis from PCAP files (freeware desktop app).
- **[VQ Monitor](https://sevana.biz/vq-monitor-asterisk-call-quality-monitoring/)** — real‑time call‑quality monitoring (incl. a free Asterisk plugin).

**Keywords:** VoIP, voice quality, call quality, MOS, POLQA, PESQ, ViSQOL, P.563, P.564,
E‑model, R‑factor, PCAP, pcapng, RTP, SIP, VoLTE, WebRTC, jitter, packet loss, echo,
DTMF, Asterisk, Wireshark alternative, MCP, Model Context Protocol, AI agent tools.

## License

MIT — see [LICENSE](LICENSE). The server software is operated by Sevana; this repository
documents how to connect to it.
