# TOSCA Continuum Governance
# TOSCA-continuum-governance
Run this on your TOSCA topology and get a full architectural risk report in 30 seconds

 
```bash
pip install pyyaml pyvis
python tools/tosca_audit.py topologies/grid5000/toulouse.yaml --format html
```

A lightweight Python toolchain that treats a **TOSCA YAML file as a queryable knowledge base** about your computing continuum — detecting scheduling risks, memory constraints, accelerator single points of failure, and ARM 32/64-bit incompatibilities *before* you deploy a single workload.

---

## Why this exists

Managing heterogeneous computing continuums — cloud servers, edge nodes, FPGA boards, NPU accelerators — is hard. Container images fail silently on wrong architectures. K3s schedulers place pods on 512 MiB nodes that immediately OOM. A single NPU becomes a SPOF for your entire inference pipeline.

**TOSCA** (Topology and Orchestration Specification for Cloud Applications) can describe all of this in a single YAML file. This toolchain makes that file *actionable*.
 

### Quick start

```bash
git clone https://github.com/alebagnato/tosca-continuum-governance
cd tosca-continuum-governance
pip install -r requirements.txt

# Audit your topology
python tools/tosca_audit.py topologies/continuum/clusters_topology.yaml

# Generate an HTML report
python tools/tosca_to_html.py topologies/grid5000/luxembourg.yaml
 
```

 
---

## Audit rules

18 rules across 8 categories. Each rule has a trigger condition, detection logic, remediation guidance, and a real-world testbed example.

| Category | Rules | Example findings |
|---|---|---|
| **ARCH** | 4 | ARMv7 32-bit in K3s cluster, mixed ISA without taints |
| **MEM** | 3 | RAM < 1 GiB (OOM risk), 512× server/agent imbalance |
| **ACCEL** | 3 | Single NPU = SPOF, unique FPGA fabrics, PS/PL contention |
| **ORCH** | 4 | K3s cluster without server node, partial Liqo federation |
| **REDUND** | 2 | Single compute node per cluster, single ISA in continuum |
| **SCALE** | 1 | Scaling policy default equals maximum |
| **CPU** | 2 | Sub-1 GHz frequency, single-core nodes |
| **COMPAT** | 1 | ARM32 declared as K3s server (unsupported since v1.24) |

```
🔴 [CRITICAL] ARCH-001 · upm_node_pynq_z1
   ARMv7 32-bit — no 64-bit Docker image support
   → Build images with: docker buildx --platform linux/arm/v7

🔴 [CRITICAL] MEM-001  · unis_node_zedboard
   RAM = 0.5 GiB — high OOM risk under container workloads
   → Reserve for bare-metal FPGA tasks only

⚠️  [WARNING]  ACCEL-001 · Continuum
   Single NPU (abi_node_imx8mp) — SPOF for inference workloads
   → Implement CPU-based fallback inference path
```

---

## ISA Taxonomy

All rules operate on a three-tier ISA taxonomy applicable to any cloud-fog-edge continuum:

- **Tier 1 — 64-bit server-class** (`linux/amd64`, `linux/arm64`): full K3s support, 64-bit address space. Architectures: x86_64, AArch64 (ARMv8-A).
- **Tier 2 — 32-bit embedded-class** (`linux/arm/v7`, `linux/riscv32`): dropped K3s v1.24+ support, 4 GB address space ceiling, in-order pipelines. Architectures: ARMv7-A, MIPS32.
- **Tier 3 — Accelerator-attached** (no OS): FPGA, NPU, GPU — sub-nodes to Tier 1/2 hosts.

---

## Topology examples

 
### Grid'5000 sites (open, reproducible data)

Four real HPC sites modelled from the [Grid'5000 public hardware pages](https://www.grid5000.fr/w/Hardware):

```
topologies/grid5000/
├── nantes.yaml        ← 3 clusters, 74 nodes, 6x Nvidia A100, all x86_64
├── luxembourg.yaml    ← 3 clusters, 56 nodes, 36x AMD MI210/MI300X, 100% SSD
├── louvain.yaml       ← 1 cluster, 8 nodes, 2×100 Gbps SR-IOV, no GPU
└── toulouse.yaml      ← 2 clusters, x86_64 + AArch64 (Jetson AGX Xavier)
```

Topologies that use publicly verifiable data  

| Site | Highlight | TOSCA finding |
|---|---|---|
| Nantes | ecotaxe: 3× A100 80GB (exotic) | ACCEL-002: 5 unique FPGA fabrics |
| Luxembourg | vianden: 8× MI300X, 1.5 TiB GPU RAM | ACCEL-001: GPU-only via exotic access |
| Louvain | spirou: 2×100 Gbps SR-IOV per node | REDUND-002: single cluster, 8 nodes |
| Toulouse | estats: only AArch64 site in survey | ARCH-003: mixed ISA (x86_64 + AArch64) |

---

 
## Related work

This toolchain complements [**TOSCA Designer**](https://github.com/Modelio-R-D/ToscaDesigner)
(latest: [v0.5.1](https://github.com/Modelio-R-D/ToscaDesigner/releases/tag/v0.5.1), Sep 2025),
an open-source module for [Modelio 5.4.1](https://github.com/ModelioOpenSource/Modelio)
that provides graphical UML-integrated modeling of cloud-fog-edge TOSCA topologies,
with a custom `eu.myrtus.*` node type hierarchy, policy and constraint editors,
and enriched CSAR export. Developed by Softeam R&D as part of the
[MYRTUS Horizon Europe project](https://myrtus-project.eu/) (Grant No. 101135183). 

**Complementarity:**

| | TOSCA Designer | This toolchain |
|---|---|---|
| Interface | Graphical (Modelio UML) | Command-line / Python |
| Input | Visual diagram → TOSCA YAML | TOSCA YAML directly |
| Focus | Design-time modeling, constraint authoring, CSAR export | Static risk analysis, CI/CD integration |
| Output | `.tosca` / `.csar` files | HTML report, interactive graph, JSON audit |
| Use when | Designing a new topology from scratch | Auditing any existing TOSCA file |

---

## Talk

**[Open Source Experience](https://www.opensource-experience.com/) — Paris, December 2025**

*TOSCA-Driven Governance of Heterogeneous Computing Continuums:
Detecting Architectural Risks Before They Become Runtime Failures*
https://www.opensource-experience.com/fr/programme-2026
Slides and live demo available in [`/talk`](./talk/).

Deploying applications on a heterogeneous cloud-fog-edge infrastructure requires topological models that account for layer heterogeneity, resource diversity, and inter-layer quality constraints. TOSCA (Topology and Orchestration Specification for Cloud Applications) provides a vendor-independent formalism for describing such topologies, but its potential for automated architectural analysis remains largely unexplored. In this presentation, we introduce a TOSCA-based governance toolchain composed of two complementary components: (1) a formal TOSCA model described in the Modelio modeling tool for heterogeneous compute nodes, and (2) an automated architectural auditor implementing 21 detection rules across 8 categories.

---

## License

[GPL](./LICENSE) — use freely, contribute back.

---

## Contributing

Topology files for your own infrastructure are welcome as pull requests.
New audit rules too.
