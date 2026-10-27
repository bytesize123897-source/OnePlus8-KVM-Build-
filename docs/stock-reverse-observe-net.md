# OnePlus 8 IN2010 STOCK Reverse-Observe-Net build

Target firmware: ColorOS IN2010_13.1.0.190(CN01)
Kernel release target: 4.19.157-perf+
Official kernel commit: 1d2678a3548f4825447f17b25a5cc1754c4cd1ae
Official modules/device-tree commit: 01fb3921e0f57c338c95f9bcbe1a9938821c7e6d

Known-good STOCK baseline boot SHA256:
9b8cc7ecab4dc29ca4d032360a3d61ca67292ad1350a63e054ee2da0ce04fbc9

## Goal

Keep the verified STOCK module ABI/trust baseline and add kernel observability useful for authorized reverse-engineering, runtime tracing and network diagnostics.

## Already enabled by the ColorOS STOCK config

- CONFIG_KPROBES=y
- CONFIG_KRETPROBES=y
- CONFIG_UPROBES=y
- CONFIG_KPROBE_EVENTS=y
- CONFIG_UPROBE_EVENTS=y
- CONFIG_BPF_EVENTS=y
- CONFIG_PERF_EVENTS=y
- CONFIG_KALLSYMS=y
- CONFIG_KALLSYMS_ALL=y
- CONFIG_BPF=y
- CONFIG_BPF_SYSCALL=y
- CONFIG_TRACEPOINTS=y
- CONFIG_NETFILTER=y
- CONFIG_NETFILTER_ADVANCED=y
- CONFIG_NETFILTER_NETLINK_QUEUE=y
- CONFIG_NF_CONNTRACK=y
- CONFIG_NF_CONNTRACK_EVENTS=y
- CONFIG_NF_CT_NETLINK=y
- CONFIG_NETFILTER_XT_TARGET_TPROXY=y
- CONFIG_NETFILTER_XT_MATCH_SOCKET=y
- CONFIG_NETFILTER_XT_MATCH_OWNER=y
- CONFIG_NET_CLS_BPF=y
- CONFIG_NET_ACT_BPF=y

## Added by this build

- CONFIG_FUNCTION_TRACER=y
- CONFIG_DYNAMIC_FTRACE=y
- CONFIG_FTRACE_SYSCALLS=y
- CONFIG_DEBUG_FS=y

## Intentionally kept disabled

- CONFIG_KVM
- CONFIG_VIRTUALIZATION
- CONFIG_KVM_ARM_HOST
- CONFIG_MMU_NOTIFIER
- CONFIG_PREEMPT_NOTIFIERS
- CONFIG_FUNCTION_GRAPH_TRACER

FUNCTION_GRAPH_TRACER is deliberately not enabled in the first build because this 4.19 tree stores graph-tracer state inside task_struct, changing a core ABI-sensitive structure and risking the same broad CONFIG_MODVERSIONS CRC breakage observed in the KVM experiment.

## Module compatibility policy

- CONFIG_MODVERSIONS=y
- CONFIG_MODULE_SIG_FORCE=y
- CONFIG_SYSTEM_TRUSTED_KEYS="certs/stock-trusted.pem"
- Keep the ColorOS stock module trust certificate embedded.
- Preserve CONFIG_ARM64_SVE=y.

First-device test must use `fastboot boot`, not `fastboot flash boot`.
